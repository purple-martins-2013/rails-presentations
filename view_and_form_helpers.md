# View and Form Helpers

## link_to

#### In Rails:
```erb
<%= link_to "My Blog", controller: "posts" %>
```
Or:
```erb
<%= link_to "My Blog", posts_path %>
```    
The hyperlink displays “My blog” and takes you to `/posts` and the specific route is shown by running ‘rake routes’ in terminal.
#### In HTML:
```html
<a href="/posts">My Blog</a>
```    
The `link_to` method is one of Rails' built-in view helpers. It creates a hyperlink based on text to display and where to go - in this case, to the path for posts.
#### In Rails:
```erb
<%= link_to 'New post', new_post_path %>
```
The hyperlink displays “New post” and takes you to `/posts/new` and the specific route is is shown by running ‘rake routes’ in terminal.

If you want to link to an action in the same controller, you don’t need to specify the `controller` option, as Rails will use the current controller by default.

You can also use `method` or `data-confirm` so that when the link is clicked, Rails will show a confirm dialog to the user, and then submit the link method.
#### In Rails:
```erb
<%= link_to 'Destroy', post_path(post), method: :delete, data: { confirm: 'Are you sure?' } %>
```
Without the `app/views/layouts/application.html.erb` file, the confirmation dialog box won’t appear. The `application.html.erb` file is automatically created when you generated the application.

## image_tag
The image_tag helper builds an HTML `<img />` tag to the specified file. By default, files are loaded from `public/images`. 
Note you must supply the file extension of the image.
#### In Rails:
```erb
<%= image_tag "icon.png" %>
```
#### In HTML:
```html
<img src="icon.png">
```
You can supply a has of additional HTML options such as `{height: 50}`, you can supply alternate text if images are turned off with `alt: “link”`, size with `size: “100x40”`, or give standard HTML options such as `class`,`id`, or `name`. 

#### In Rails:
```erb
<%= image_tag "icons/delete.gif", {height: 45} %>
```
#### In HTML:
```html
<img src="icons/delete.gif" height="42">
```
#### Other examples in Rails:
```erb
<%= image_tag "home.gif" %>

<%= image_tag "home.gif", alt: "Home" %>

<%= image_tag "home.gif", size: "50x20" %>

<%= image_tag "home.gif", alt: "Go Home",
                          id: "HomeImage",
                          class: "nav_bar" %>
```
See also: http://apidock.com/rails/v3.2.8/ActionView/Helpers/AssetTagHelper/image_tag 

## form_tag

`form_tag` is the most basic of the form helpers.
```erb
<%= form_tag do %>
  Form contents
<% end %>  
```
When called without arguments (like above), it will post to the current page by default. You can also specify a url and method using the following syntax: 
```erb
<%= form_tag("/users", method: "get") do %>
```
There are also a bunch of helpers for generating form elements:
```erb
<%= label_tag(:email, "Email:") %>
```
Generates a label for the form field with the name “email”.
```erb
<%= text_field_tag(:email) %>
```
Generates an input field of the type “text” with the name “email”. The input field is also automatically assigned an ID of the same name.

#### Other examples in Rails:
```erb
<%= check_box_tag(:has_car) %>
<%= radio_button_tag(:car, "truck") %>
<%= password_field_tag(:password) %>
<%= date_field(:user, :birthdate) %>
<%= email_field(:user, :email_address) %>
<%= text_area_tag(:message, "Enter comments", size: "24x6") %>
```

## form_for

Binds a form to a model object (for example, `@user`). If you are using RESTful routes and resources, Rails can infer the method and url. If `@user` doesn’t yet exist, `form_for` will automatically make a post to `/users`. If it already exists, the form url will be `/users/:id` (so you can update the model object).

#### In Rails:
```erb
<%= form_for @user do |f|  %>
    <%= f.text_field :username, placeholder: "Username" %>
    <%= f.email_field :email, placeholder: "Email" %>
    <%= f.password_field :password, placeholder: "Password" %>
    <%= f.submit "Sign up!" %>
<% end %>
```
In the example above, `params[:user]` will return all the form inputs.

#### In HTML:
```html
<form action="/users" method="post">
    <input type="text" name="user[username]" placeholder="Username">
    <input type="email" name="user[email]" placeholder="email">
    <input type="password" name="user[password]" placeholder="Password">
    <input type="submit" value="Sign up!">
</form>
```

## fields_for
Creates a scope around a specific model object like `form_for`, but doesn’t create the form tags themselves.  This makes `fields_for` suitable for editing or creating multiple model objects in the same form. If you had both a `Person` model and a `ContactDetail` model which is associated with `Person`, you could create a form like this:
```erb
<%= form_for @person, url: {action: "create"} do |person_form| %>
  <%= person_form.text_field :name %>
  <%= fields_for @person.contact_detail do |contact_details_form| %>
    <%= contact_details_form.text_field :phone_number %>
  <% end %>
<% end %>
```
The form produced by the code above will create both a Person model object and a ContactDetail model object. The object yielded by `fields_for` is a form builder like the one yielded by `form_for` (`form_for` calls `fields_for` internally).
