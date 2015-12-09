Sometimes, having the user sign in through ajax is one of the best things you can do to improve the user experience on your website. I found myself in this situation, so I hacked around devise to allow the user to sign in via ajax. Here is how you can do it:

## Generate Devise Views

First, in case you’re not familiar with it, [devise](https://github.com/plataformatec/devise) is an incredible rails gem that handles all user authentication stuff. Once you got the basic devise functionality setup by following the README, make sure to [generate the devise views](https://github.com/plataformatec/devise/wiki/How-To:-Create-Haml-and-Slim-Views) inside your own app as well. The wiki page explains it all, so I won’t go into detail on this part of the process.

## Configure Devise To Accept JSON

We need to configure devise to accept a JSON request, which is what our AJAX will send it. To do that, simply change the following variables in your config/initializers/devise.rb file:


``` ruby
# config/initializers/devise.rb

config.http_authenticatable_on_xhr = false
config.navigational_formats = ["*/*", :html, :json]
```


## Create a Sessions Controller

The sign in action happens from the [devise sessions controller](https://github.com/plataformatec/devise/blob/master/app/controllers/devise/sessions_controller.rb). You’ll have to overwrite the controller to respond with JSON. So just create a sessions controller, and here is an example of how you would overwrite it:

``` ruby
# app/controllers/sessions_controller.rb

class SessionsController resource_name, :recall => "#{controller_path}#failure")
    sign_in_and_redirect(resource_name, resource)
  end

  def sign_in_and_redirect(resource_or_scope, resource=nil)
    scope = Devise::Mapping.find_scope!(resource_or_scope)
    resource ||= resource_or_scope
    sign_in(scope, resource) unless warden.user(scope) == resource
    return render :json => {:success => true}
  end

  def failure
    return render :json => {:success => false, :errors => ["Login failed."]}
  end
end
```


## Add Devise Mappings To Your Application Helper

Since devise is using the generic “resource” in it’s code, you need to map your user model to the devise resource. You can do that by adding the following to your applications helper or the helper file for the controller where you’d like to use the devise ajax functionality:


``` ruby
# app/helpers/application_helper.rb

module ApplicationHelper
  def resource_name
    :user
  end

  def resource
    @resource ||= User.new
  end

  def devise_mapping
    @devise_mapping ||= Devise.mappings[:user]
  end
end
```


## Redirect Routes!

In case you’ve tried the above steps and nothing changed, that’s because you have to redirect devise to use your controller instead of it’s own. This is done very easily in your routes file:


``` ruby
# config/routes.rb

devise_for :users, :controllers => {sessions: 'sessions'}
```


## Update Your Sign In Form

This is where those devise views in your rails app become handy. The form_for helper accepts a _:remote => true_ options, which automatically posts the form remotely (via ajax) for you, and the _:format => :json_ option converts your form information into json. Here is what your form should look like (if you’re using haml):


``` ruby
 # app/views/devise/sessions/new.html.haml

%h2 Sign in
= form_for(resource, :as => resource_name,
                     :url => session_path(resource_name) ,
                     :html => {:id => "sign_in_user"},
                     :format => :json,
                     :remote => true ) do |f|
  %div
    = f.email_field :email, placeholder: 'email'
  %div
    = f.password_field :password, placeholder: "password"
  - if devise_mapping.rememberable?
    %div.remember
      = f.check_box :remember_me
      = f.label :remember_me
  %div= f.submit "Sign in", class: 'btn btn-success btn-medium'
```


## Add The JavaScripts

And, of course, you need JavaScript to catch the ajax response from your server and handle it on the client side. Here is an example (in coffeescript):


``` ruby
# app/assets/javascripts/application.js.coffee

$("form#sign_in_user").bind "ajax:success", (e, data, status, xhr) -&gt;
    if data.success
      $('#sign_in').modal('hide')
      $('#sign_in_button').hide()
      $('#submit_comment').slideToggle(1000, "easeOutBack" )
    else
      alert('failure!')
```


And now, try signing in! Isn’t this fun?

