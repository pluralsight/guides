In this tutorial, we'll be creating a Rails 5 App that uses a custom Devise log in page with ActionCable! What makes this app so unique is that we want to define a subscribed user in our connection channel to be the `current_user` that signs in. I will walk you through the steps of completing this challenge.

Together, Devise and ActionCable will allow us to track a user's actions, specifically in a game setting. This can later let us create a game lobby, see users' past scores, and determine who is #1 on the leaderboards.

Let's get started. 

First make sure that you have Rails up to date. Use: `rails -v` and make sure the output matches this: `Rails 5.0.0.rc1`

Once you have the most up-to-date version of Rails, create a new Rails 5 app:

`rails new <Your App Name>`

Next include the Devise Gem File: 

`gem 'devise'`

Bundle:

`bundle install`

Next install the Devise basics, along with the Devise Views: 

`rails generate devise:install && rails generate devise:views`

Now it is time to create a User model with Devise

`rails generate devise <Your Model Name, Usually Just: user>`

Finally we need to migrate and rollback our Devise table:

`rake db:migrate && rake db:rollback && rake db:migrate`

Congrats! You have successfully created a Devise model.

But now we want to add in a custom sign in such as studentID instead of email. For this example we will use:
`studentid`

We are first going to add authentication to the User Model so we can allow `studentid` to be the verification for signing a user in by adding `:authentication_keys`: 
```
class User < ApplicationRecord

  # Include default devise modules. Others available are:
  # :confirmable, :lockable, :timeoutable and :omniauthable
  validates :studentid, numericality: { only_integer: true}, presence: true, length: { is: 6 }

  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :trackable, :validatable, :authentication_keys => [:studentid]
end

```

Next we are going to go into our ApplicationController to permit studentid inputs when signing in, as well as other devise attributes:

```
class ApplicationController < ActionController::Base
  # Prevent CSRF attacks by raising an exception.
  # For APIs, you may want to use :null_session instead.
  protect_from_forgery with: :exception
  before_action :configure_permitted_parameters, if: :devise_controller?

  protected

  def after_sign_in_path_for(users)
    grid_index_path
  end


  def configure_permitted_parameters
    devise_parameter_sanitizer.permit(:sign_up) { |u| u.permit(:studentid, :email, :password, :password_confirmation, :remember_me) }
    devise_parameter_sanitizer.permit(:sign_in) { |u| u.permit(:studentid, :password, :remember_me) }
    devise_parameter_sanitizer.permit(:account_update) { |u| u.permit( :studentid, :email, :password, :password_confirmation, :current_password) }
  end

end
```
`def after_sign_in_path_for(users)` will allow you to redirect users after sign-in in to your home page. In this case my home page is: `grid_index_path`

We will now need to include this in our `routes.rb` file. 
```
Rails.application.routes.draw do
  devise_for :users, controllers: { registrations: "registrations"}
  as :user do
    get '/' => 'devise/registrations#new'
  end
  mount ActionCable.server => "/cable"
  resources :grid
end

```
Here we will specify that upon visiting our site a user will be sent to creating a new account via `'devise/registrations#new'`. You can send them other places as well by changing the route. For now ignore the Action Cable Mount but we will need that for later! 

Next we'll go into our Views for Devise to add our custom sign in. 

First, change the `new.html.erb` file in `views/devise/registrations` by adding in a field for our new custom sign in:
```
<div class="field">
    <%= f.label :studentid, 'Colorado College ID' %><br />
    <%= f.text_field :studentid, autofocus: true %>
  </div>
```
The full file will now look like this: 
```
<h2>Sign up</h2>

<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

  <div class="field">
    <%= f.label :studentid, 'Colorado College ID' %><br />
    <%= f.text_field :studentid, autofocus: true %>
  </div>
  <br>
  <div class="field">
    <%= f.label :email %><br />
    <%= f.email_field :email%>
  </div>
  <br>
  <div class="field">
    <%= f.label :password %>
    <% if @minimum_password_length %>
    <em>(<%= @minimum_password_length %> characters minimum)</em>
    <% end %><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>
  <br>

  <div class="field">
    <%= f.label :password_confirmation %><br />
    <%= f.password_field :password_confirmation, autocomplete: "off" %>
  </div>
  <br>
  <div class="actions">
    <%= f.submit "Sign up" %>
  </div>
<% end %>

<%= render "devise/shared/links" %>

```
We will now also need to change the file `new.html.erb` file in `views/devise/sessions`
```
<h2>Log in</h2>

<%= form_for(resource, as: resource_name, url: session_path(resource_name)) do |f| %>
  <div class="field">
    <%= f.label :studentid, 'Colorado College ID' %><br />
    <%= f.text_field :studentid, autofocus: true %>
  </div>

  <div class="field">
    <%= f.label :password %><br />
    <%= f.password_field :password, autocomplete: "off" %>
  </div>

  <% if devise_mapping.rememberable? -%>
    <div class="field">
      <%= f.check_box :remember_me %>
      <%= f.label :remember_me %>
    </div>
  <% end -%>

  <div class="actions">
    <%= f.submit "Log in" %>
  </div>
<% end %>

<%= render "devise/shared/links" %>

```
Now it is time to add this to our table and re-migrate: 
```
class DeviseCreateUsers < ActiveRecord::Migration[5.0]
  def change
    create_table :users do |t|
      ## Database authenticatable
      t.integer :studentid,  null: false
      t.string :email,              null: false, default: ""
      t.string :encrypted_password, null: false, default: ""

      ## Recoverable
      t.string   :reset_password_token
      t.datetime :reset_password_sent_at

      ## Rememberable
      t.datetime :remember_created_at

      ## Trackable
      t.integer  :sign_in_count, default: 0, null: false
      t.datetime :current_sign_in_at
      t.datetime :last_sign_in_at
      t.inet     :current_sign_in_ip
      t.inet     :last_sign_in_ip



      t.timestamps null: false
    end

    add_index :users, :email,                unique: true
    add_index :users, :reset_password_token, unique: true
    # add_index :users, :confirmation_token,   unique: true
    # add_index :users, :unlock_token,         unique: true
  end
end
```
*You can add in validations here as you like. I would recommend adding in a unique validation so multiple users can not have the same id or in other cases, username.*

Now Migrate again: 
`rake db:migrate`

Great! We've customized sign-in options and enabled sign-in. Now, we'll set up a subscribed connection if a user has signed in via Devise. 

First make sure that you have ActionCable by seeing that you have the folder `app/channels`. After that, define a subscription by updating the file `app/channels/connection.rb`.

```
module ApplicationCable
  class Connection < ActionCable::Connection::Base
    identified_by :current_user

    def connect
      self.current_user = find_verified_user
      logger.add_tags 'ActionCable', current_user.studentid
    end

    protected

    def find_verified_user # this checks whether a user is authenticated with devise
      if verified_user = env['warden'].user
        verified_user
      else
        reject_unauthorized_connection
      end
    end
  end
end
```
This will allow you to check the current user on the page and verify that the user is signed in. If so, the user becomes a valid subscriber. Clearly Devise and ActionCable work well together when it comes to authentication and user tracking.

Thanks for reading this tutorial. Hopefully you found my methods easy to follow. Please favorite this tutorial and leave your feedback and questions in the discussions section below. 
