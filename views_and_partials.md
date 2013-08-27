<h1>Views & Partials</h1>

<h6>In Sinatra:</h6>
```ruby
get '/users/:id' do
  @user = User.find(params[:id])
  erb :show_user
end
```

<h6>In Rails:</h6>

```ruby
class UsersControllers < ApplicationController
  def show
  @user = User.find(params[:id])
  render :show
  end
end
``` 

However, the ```render :show``` is not necessary.  [By default, controllers in Rails automatically render views with names that correspond to valid routes](http://guides.rubyonrails.org/layouts_and_rendering.html#rendering-by-default-convention-over-configuration-in-action); in this case, Rails will look for a file named show.html.erb ( *or show.builder if you’re using Builder, the XML generator* ) inside your Views folder, because of the method ( *i.e., route* ) name "show". You can also adjust it to use [haml, slim or whatever you prefer](http://railsapps.github.io/rails-haml.html).


<h3>Rails provides three ways of creating an HTTP response:</h3>

1) ```render ``` creates a response, a la ```erb :"<blah blah>”``` or ```haml :”<blah blah>”```

2) ```redirect_to ``` sends an HTTP redirect to the browser

3) ```head``` sends back only HTTP headers

<h2>Render</h2>

Writing the word "render" becomes necessary when you’re trying to render a view that lives elsewhere -- i.e., your route/method/action name doesn’t match the name of the view ( *note that “route”, “controller method” and “action” are essentially synonymous for the purposes of this discussion* ).  In that case, simply specify the location/name of the view:

```
render "products/show"
```
-- will render a template show.html.erb within app/views/products/


```
render "/u/apps/warehouse_app/current/app/views/products/show"
```
-- if you’d like to render a view from a completely separate Rails app; the leading slash character (/) indicates to Rails that you’re rendering a separate file


```
render action: "index"
```
-- sort of a "goto" command, where you’re rendering the index method/route, but without executing the code inside the route itself.  If you’re looking to run the code, use ```redirect_to```


```
render inline: "<% products.each do |p| %><p><%= p.name %> </p><% end %>"
```
-- render inline allows you to supply ERB instead; this should be avoided unless absolutely necessary, since it’s not MVC.  Put it into an erb file


```
render text: "Hello World!"
```
-- useful when you’re responding to AJAX or web service requests that want text; if you want text rendered with the current layout, add a comma and ```layout: true```


```
render json: @product
```
-- Will automatically convert ```@product``` to JSON and render it to the browser


```
render xml: @product
```
-- Will automatically convert ```@product``` to XML and render it to the browser


```
render js: "alert(‘Hello World!’);
```
-- Will render vanilla JavaScript in browser


You can add four different types of options to render ... ```:content_type```, ```:layout```, ```:location```, ```:status```

```
render file: filename, content_type: "application/rss"
```
-- specify content-type if you’re rendering anything besides HTML, JSON, XML or JS


```
render layout: "special_layout"
render layout: false
```
-- Use a specific file as the layout, or use no layout at all (<em>same as Sinatra</em>)


```
render xml: photo, location: photo_url(photo)
```
-- Use location: to set the HTTP Location header


```
render status: 500
render status: :forbidden
```
-- If you want to change HTTP response status codes -- [see here](http://www.codyfauser.com/2008/7/4/rails-http-status-code-to-symbol-mapping) for an index of codes. 


<h3>Most Common Error</h3>

Unlike Sinatra, where multiple ```erb :blah``` lines in the same route (<em>not specific to each conditional path</em>) will trigger in turn, in Rails it’ll just give you an error: ```“Can only render or redirect once per action”```.

In each route/action/controller method’s path, you can only have one render or redirect -- if you’d like to create flow control on the spot, you can use ```and return```:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show" and return
  end
  render action: "regular_show"
end
```

Or an easier way of writing this particular example, since ActionController will avoid doing the **implicit render** if a render action’s already been taken:

```ruby
def show
  @book = Book.find(params[:id])
  if @book.special?
    render action: "special_show"
  end
end
```

If the book’s not special, the ActionController will simply render the default "show" template when it reaches the end of the method/action/route.

<h2> Redirect_to</h2>
Tells the browser to send a new request for a different URL; essentially works the same as Sinatra’s redirect, except for three things:

```
redirect_to photos_url
```
-- usually your path is already contained in a variable, courtesy of some of the other underlying magic of Rails


```
redirect_to :back
```
-- sends the user back to the page they just came from


```
redirect_to photos_path, status: 301
```
-- you can change the redirect status code ( *from 302, a temporary redirect, to 301, a permanent redirect* ) if you’d prefer


```
redirect_to action: :index
```
-- Browser will make a fresh request for the index method/route, execute the code inside that route (i.e., this is like ```redirect ‘/’``` in Sinatra if we’re talking of the primary index view, since ```def index``` in this case is like ```get "/" do``` in Sinatra)


<h2>Head</h2>

This will produce whatever HTTP header you prefer.

```
head :bad_request
```
-- produces HTTP/1.1 400 Bad Request …


```
head :created, location: photo_path(@photo)
```
-- produces HTTP/1.1 201 Created …


<h2>More About Layouts</h2>

Rails will automatically look for a layout file inside *app/views/layouts* with the same base name as the controller.  So, if you’re in the PhotosController class, it’ll look for *app/view/layouts/photos.html.erb* ( *or .builder* ) … if that doesn’t exist, it’ll use *app/view/layouts/application.html.erb* ( *or .builder* ).

You can override this behavior by using a layout declaration:

```ruby
class ProductsController < ApplicationController
  layout "inventory"
  # ...
end
```
-- all views rendered by the products controller will use *app/views/layouts/inventory.html.erb*


```ruby
class ApplicationController < ActionController::Base
  layout "main"
  # ...
end
```
-- all views in the entire application will default to *app/views/layouts/main.html.erb*


You can also defer/select the choice of layout in a few different ways:

```ruby
class ProductsController < ApplicationController
  layout :products_layout
  def show
    @product = Product.find(params[:id])
  end

  private
    def products_layout
      @current_user.special? ? "special" : "products"
    end
end
```
-- special users will get a special layout


```ruby
class ProductsController < ApplicationController
  layout Proc.new { |controller| controller.request.xhr? ? "popup" : "application" }
end
```
-- using a Proc (or any inline method) to determine the layout -- in the example above, the layout is based on the current request


```ruby
class ProductsController < ApplicationController
  layout "product", except: [:index, :rss]
end
```
-- product layout is used for everything except the index and rss methods; you can also use the ``` :only ``` option (i.e., ```only: [:index, :rss]```) as well


Layout inheritance works exactly as you’d expect, with regard to overwriting default layouts.  Note that since ``` render :<view_name> ``` isn’t required when you’re in the correctly named method/route, your last line of code when rendering a different layout would simply be ``` render layout: "<different layout name>" ```

<h2>Simplifying Layouts and Views</h2>

<h4>with Tags, Yield/Content_For, and Partials</h4>

Now that we’re building in Rails, we’ll need to make much greater use of partials, as part of good coding practice.  Partials, asset tag helpers and ``` yield ``` / ``` content_for ``` can be used in **all types of views**, both layouts and any of your other view files, though the ```auto_discovery_link_tag```, ```javascript_include_tag```, and ```stylesheet_link_tag`` are most commonly found in the header section of a layout.

<h3>Asset Tag Helpers</h3>

There are six asset tag helpers in Rails:
``` auto_discovery_link_tag ```
``` javascript_include_tag ```
``` stylesheet_link_tag ```
``` image_tag ```
``` video_tag ```
``` audio_tag ```


<h4>auto_discovery_link_tag</h4>
Builds HTML that most browsers & newsreaders can use to detect the presence of RSS or Atom feeds.  It takes either ```:rss``` or ```:atom``` type, a hash of options passed through to url_for, and a hash of options for the tag.

``` <%= auto_discovery_link_tag(:rss, {action: "feed"}, {title: “RSS Feed”}) %>```

Within the second hash of options, you can include ```:rel``` ( *rel value in the link, default "alternate"* ), ```:type``` ( *explicit MIME type, otherwise automatically generated* ), and ``` :title ``` ( *as above; defaults to uppercase :type value -- either ATOM or RSS* )

<h4>javascript_include_tag</h4>
Returns HTML script tag for each source provided; with the [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) enabled, this will generate a link to /assets/javascripts rather than public/javascripts.  You can specify full paths relative to the document root, or a URL, if you prefer.  JavaScript files within Rails applications or engine go into one of three locations: app/assets, lib/assets or vendor/assets.

``` <%= javascript_include_tag "main" %> ``` ==>
```
<script src=’/assets/javascripts/main.js’></script>
```
-- the request to this asset is then served by the Sprockets gem


``` <%= javascript_include_tag "main", “/photos/columns” %> ``` ==>

``` <script src=’/assets/javascripts/main.js’></script> ```
``` <script src=’/assets/javascripts/photos/columns.js’></script> ```
``` stylesheet_link_tag ```

Same as ``` javascript_include ``` above, except for CSS stylesheets.  Stylesheet files can be stored in one of three locations: app/assets, lib/assets or vendor/assets.  With [Asset Pipeline](http://guides.rubyonrails.org/asset_pipeline.html) enabled, the helper generates a link to /assets/stylesheets/.

By default, links are created with ```media="screen” rel=”stylesheet”;``` you can overrie these by specifying an appropriate option (```:media```, ```:rel```) as follows:
``` <%= stylesheet_link_tag "main_print", media: “print” %> ```

<h4>image_tag, video_tag and audio_tag </h4>

```image_tag:```  Builds an HTML img tag to the specified file; by default, loaded from public/images.  You must specify the extension of the image.  Additional tags or options can be added as noted below.
``` <%= image_tag "header.png" %>```
``` <%= image_tag "icons/delete.gif", {height: 45} %>```
``` <%= image_tag "home.gif", alt: “Home” %>```
``` <%= image_tag "home.gif", size: “50x20” %>```
``` <%= image_tag "home.gif", alt: “Go Home”, id: “HomeImage”, class: “nav_bar” %>```


```video_tag:``` Builds an HTML5 video tag to the specified file; by default, loaded from public/videos.  Again, can add additional tags or options ( *same as the ones mentioned above* ).  Also supports all <video> HTML options through the HTML options hash, including ```poster``` ( *placeholder image before starting play* ), ```autoplay``` ( *plays on page load* ), ```loop``` ( *loops video* ), ```controls``` ( *browser-supplied user controls enabled* ) and ```autobuffer``` ( *preload file on page load* ).

``` <%= video_tag "movie.ogg", size: “250x250” %>```
``` <%= video_tag ["trailer.ogg", “movie.ogg”] %>```
-- specify multiple videos by passing in an array of videos 
produces ``` <video><source src="trailer.ogg” /><source src=”movie.ogg” /></video>```


```audio_tag:``` Builds an HTML5 audio tag to the specified file; by default, loaded from public/audios.  Again, has additional tags/options, and special audio options (**autoplay, controls, autobuffer**).
``` <%= audio_tag "music.mp3" %>```


<h2>Yield / Content_For</h2>

As with Sinatra, can use ```<%= yield %>``` to specify the location where content from the view should be inserted.  Multiple yielding regions can also be used, say for head and footer regions, or for sidebars, or to insert page-specific JavaScript or CSS into headers:

```erb
<html>
  <head>
    <%= yield :head %>
  </head>
  <body>
  <%= yield %>
  </body>
</html>
```

The main body of the view will always render into the unnamed yield.  To render content into a named yield block, use the ```content_for``` method.

```erb
<% content_for :head do %>
  <title>A simple page </title>
<% end %>
```

<h2>Partials</h2>

Preferred: ```<%= render 'menu', :item => Item.last, :user => User.last %>```

Try to avoid: ```<%= render :partial => 'menu', :locals => {:item => Item.last, :user => User.last} %>```

A partial works as in Sinatra -- to render a partial as part of a view, you use the render method within the view:

```<%= render "menu" %>```

This renders a file named ``` _menu.html.erb``` -- note that all partials are named with a leading underscore to distinguish them from regular views, even though they’re referred to without the underscore.  The same is true when you’re pulling in a partial from another folder  ```<%= render "shared/menu" %>``` pulls code from app/views/shared/_menu.html.erb.

You can use partials to simplify views, the equivalent of subroutines -- it’s particularly useful for content shared among many pages in your application.  If it’s shared among all pages in the app, you can put the partials in your layouts.

A partial can also use its own layout file, just as a view might use a layout.  You can call a partial like this:

```<%= render partial: "link_area", “layout: “graybar” %>```

Note that we’re explicitly specifying ```:partial``` -- this is only required when we’re passing additional options such as ```:layout```.  This line looks for a partial named ```_link_area.html.erb``` and renders it using the layout ```_graybar.html.erb```.  **The layout should be placed in the same folder as the partial that it belongs to, not in the master layouts folder**.

You can also pass local variables into partials:

```erb
<%= render partial: "form", locals: {zone: @zone} %>
<%= form_for (zone) do |f| %>
  <%= f.text_field :name %>
  ...
  <%= f.submit %>
<% end %>
```

Through Rails magic, the Action View’s submit helper will return different responses for different pages from which the form arises, since the zone is unique to each rendering of the partial, even though the same partial is rendered.

Every partial also has a local variable with the same name as the partial (minus the underscore) … you can pass an object to this local variable via the ```:object``` option:

<h4><%= render partial: "customer", object: @new_customer %></h4>

Within the customer partial, the customer variable will refer to ```@new_customer``` from the parent view.

Finally, if you want to **render a model into a partial**, you can use shorthand syntax:
```<%= render @customer %>```

If the ```@customer``` instance variable contains an instance of the Customer model, it will use _customer.html.erb to render it, and pass the local variable ```customer``` into the partial, which will refer to the ```@customer``` instance variable in the parent view.

<h3>Rendering Collections</h3>

Partials are also very useful in rendering collections.  When you pass in a collection as below, you will insert the partial once for each member in the collection:

```<%= render partial: "product", collection: @products %>```

When you use a pluralized name for the collection, the individual instances of the partial have access to the member of the collection being rendered via a variable named after the partial.  Within the ```_product``` partial, you can refer to ```product``` to get the instance being rendered.

Again, with shorthand, if ```@products``` is a collection of product instances, you can simply write this for the same result:
```<%= render @products %>```

In addition, you could create a heterogeneous collection to render, and Rails will choose the proper partial for each member of the collection:
```<%= render [customer1, employee1, customer2, employee2] %>```

If the collection is empty, render returns nil -- you can provide alternative content easily as follows:
```<%= render(@products) || "There are no products available" %>```

You can also customize local variable names in the partial, by specifying the ```:as``` option in the call to the partial:
```<%= render partial: "product", collection: @products, as: :item %>```

or pass in arbitrary local variables through the ```locals: {}``` option:
```<%= render partial: "product", collection: @products, as: :item, locals: {title: “Products Page”} %>```


Rails also makes a counter variable available within a partial called by the collection, named after the member of the collection followed by ```_counter```.  For ```@products```, you can refer to ```product_counter``` to tell you how many times the partial has been rendered … this does not work in conjunction with the ```as: :value``` option.

The ```:spacer_template``` option allows you to render a second partial between instances of the main partial:
```<%= render partial: @products, spacer_template: "product_ruler" %>```

Between each pair of ```_product``` partials, Rails will render the ```_product_ruler``` partial (with no data passed to it).

It is also possible to use the ```:layout``` option when rendering collections:
```<%= render partial: "product", collection: @products, layout: “special_layout” %>```

The layout will be rendered with the partial, for each item in the collection.  The current object and ```object_counter``` variables will also be available in the layout.


<h3>Nested Layouts</h3>

Nested layouts, or sub-templates, allow you to modify your layout for a particular controller.  Say that you have one layout for your ApplicationController, but want to hide the top menu and add a right-sided menu for pages generated by a sub-controller.

```erb
<% content_for :stylesheets do %>
  #top_menu {display: none}
  #right_menu {float: right; background-color: yellow; color: black}
<% end %>

<% content_for :content do %>
  <div id="right_menu”>Right menu items here </div>
<% end %>

<%= render template: "layouts/application"%>
```

You can do this in a variety of different ways, with different sub-templating schemes, to get a similar result; there is no limit in the depth of nesting levels.  You can use the ```ActionView::render``` method via render template to base a new layout on the prior layout.  If you wish to subtemplate the sub-controller’s layout further, you can ensure that the existing template is used by default only on the subcontroller page with ```content_for?(:<name_of_page>) ? yield(:<name_of_page>) : yield```

