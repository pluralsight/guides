I found a [great Pluralsight tutorial](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api) regarding Ruby on Rails and token-based authentication. Having a similar problem to tackle, I thought I might add some tweaks to make it work with a dual system. In other words, I wanted authentication to work with both sessions and tokens. 


This guide focuses on RoR applications that use Java Web Tokens [(JWT)](https://jwt.io/) and [Devise](https://github.com/plataformatec/devise) for online registration and sign-in functionality and want to have token based authentication for API calls, whether from front-end platforms or mobile apps.

While this approach targets a monolithic application structure, which is frowned upon for good reasons, it might be usefull for Proof of Concept demos, quick mocks, and small-scale work.


### What is token-based authentication?
In short, token-based authentication is a security strategy in which users log in using their credentials and are given a signed and usually time-limited token that allows them to access various resources. As a result, the user no longer needs to send login information to the server whenever sending requests, making the system more secure. This [great guide](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api#what-is-token-based-authentication-) covers everything else in more depth. 


# Setting up Dual authentication with Rails 4
Parts of this tutorial were directly copied from [the guide](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api) by [Hristo Georgiev](https://www.pluralsight.com/guides/author/hggeorgiev).

I assume you already have a Rails 4 application set up to work with [Devise](https://github.com/plataformatec/devise). I also assume that you are using a vanilla [Devise](https://github.com/plataformatec/devise) setup. In other words, you did not override any controllers yet and you use at least `:database_authenticatable`, `:registerable strategies`.

Let's start by implementing the JWT controller. First, the `jwt` gem will make encoding and decoding of HMACSHA256 tokens available in the Rails application. Let's install it:
 
 ```ruby
  gem 'jwt'
```
And in bash:


```bash
bundle install
```

Once the  gem is installed, it can be accessed through the `JWT` global variable. Because the methods that are going to be used require encapsulation, a singleton class is a great way of wrapping the logic and using it in other constructs. 

For those who are unfamiliar, **a singleton class** restricts the instantiation of a class to a single object, which comes in handy when only one object is needed to complete the tasks at hand.

 ```ruby
 
 # lib/json_web_token.rb

class JsonWebToken
  class << self
    def encode(payload, exp = 24.hours.from_now)
      payload[:exp] = exp.to_i
      JWT.encode(payload, Rails.application.secrets.secret_key_base)
    end

    def decode(token)
      body = JWT.decode(token, Rails.application.secrets.secret_key_base)[0]
      HashWithIndifferentAccess.new body
    rescue
      nil
    end
  end
end
```
The first method,`encode`, takes three parameters -- the user ID, the expiration time (1 day), and the unique base key of your Rails application -- to create a unique token. 

The second method,`decode` takes the token and uses the application's secret key to decode it. 

These are the two cases in which these methods will be used:
 * For authenticating and providing a signed token for the user by using `encode`
 * To check if the user's token appended to each request is correct by using `decode`. 

To make sure everything will work, the contents of the `lib` directory have to be included when the Rails applciation loads:
```ruby
    #config/application.rb
module ApiApp
  class Application < Rails::Application
    #.....
    config.autoload_paths << Rails.root.join('lib')
    #.....
    end
   end
```
### Authenticating users

Instead of using private controller methods, we can use [`simple_command`](https://github.com/nebulab/simple_command) instead.

The `simple_command` gem is an easy way of creating services. Its role is similar to the role of a helper, but instead of facilitating the connection between the controller and the view, it does the same for the controller and the model. In this way, we can shorten the code in the models and controllers.

Add the gem to your `Gemfile`:
```ruby
gem 'simple_command'
```
And bundle it:
```bash
bundle install
```

Then, the alias methods of the `simple_command` can be easiy used in a class by writing `prepend SimpleCommand`. Here is how a command is structured:
```ruby

class AuthenticateUser
   prepend SimpleCommand

  def initialize()
   #this is where parameters are taken when the command is called
  end

  def call
   #this is where the result gets returned
  end

end
```
The command takes the user's e-mail and password and return the user if the credentials match. Here is how this can be done:
```ruby
# app/commands/authenticate_user.rb

class AuthenticateUser
  prepend SimpleCommand

  def initialize(email, password)
    @email = email
    @password = password
  end

  def call
    JsonWebToken.encode(user_id: user.id) if user
  end

  private

  attr_accessor :email, :password

  def user
    user = User.find_by_email(email)
    return user if user && user.valid_password?(password)

    errors.add :user_authentication, 'invalid credentials'
    nil
  end
end
```


The command takes the parameters and initializes a class instance with `email` and `password` attributes that are accessible within the class. The private method `user` uses the credentials to check if the user exists in the database using `User.find_by_email` . 

If the user is found, the method uses the [Devise](https://github.com/plataformatec/devise) built-in method `valid_password?` in the User model to check if the user's password is correct. If everything is true, the user will be returned. If not, the method will return `nil`.


### Checking user authorization
The token creation is done, but there is no way to check if a token that's been appended to a request is valid. The command for authorization has to take the `headers` of the request and decode the token using the `decode` method in the `JsonWebToken` singleton. 

> **A refresher on headers**
  Http requests have fields known as [headers](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html). Headers can contain a wide variety of information about the request that can be helpful for the server to interpret it. For example, a header can contain the format of the request body, authorization information and other meta information (you can find all the types [here](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)). Tokens are usually attached to the 'Authorization' header.
  
  Here is how the code is structured:
```ruby
# app/commands/authorize_api_request.rb

class AuthorizeApiRequest
  prepend SimpleCommand

  def initialize(headers = {})
    @headers = headers
  end

  def call
    user
  end

  private

  attr_reader :headers

  def user
    @user ||= User.find(decoded_auth_token[:user_id]) if decoded_auth_token
    @user || errors.add(:token, 'Invalid token') && nil
  end

  def decoded_auth_token
    @decoded_auth_token ||= JsonWebToken.decode(http_auth_header)
  end

  def http_auth_header
    if headers['Authorization'].present?
      return headers['Authorization'].split(' ').last
    else
      errors.add(:token, 'Missing token')
    end
    nil
  end
end
```
This code executes a chain of methods. **Let's go from bottom to top.** 

*Operations are explained easiest in reverse order.*

The last method in the chain, `http_auth_header`, extracts the token from the authorization header received upon the initialization of the class. The second method in the chain is `decoded_auth_token`, which decodes the token received from `http_auth_header`and also retrieves the user's ID.

   The logic in the `user` method might seem abstract, so let's go through it line by line.

In the first line, the `||=` operator is used to assign `@user` by assigning "if not `nil`". If the `User.find()` returns an empty set or `decoded_auth_token` returns false, `@user` will be `nil`, so a nil check is necessary.

Moving to the second line, the  `user` method will either return the user or throw an error. In Ruby, the last line of the function is implicitly returned, so the command ends up returning the user object.

We have the token mechanism in place. We can now set up devise.

### Setting up Devise

There are a couple of issues with using devise session-based authentication alongside JWT token-based authentication:

-  Override the Devise `registrations_controller.rb` and `sessions_controller.rb` to prevent the `json` requests from trying to render a view.
-  Setup routes for the *Devise* override
-  Create logic to distinguish between browser `html` requests and API `json`

#### *Devise* controllers and routes setup

First, we need to override Devise controllers to restrict API rendering. 

Override `registrations_controller.rb`:
```ruby
# app/controllers/registrations_controller.rb

class RegistrationsController < Devise::RegistrationsController  
    respond_to :html, :json
    clear_respond_to if request.format == 'json'
end 
```

Let's do the same for `sessions_controller.rb`:
```ruby
# app/controllers/sessions_controller.rb

class SessionsController < Devise::SessionsController  
    respond_to :html, :json
    clear_respond_to if request.format == 'json'
end 
```

Setup `routes.rb`

```ruby
devise_for :users, :controllers => {sessions: 'sessions', registrations: 'registrations'}  
```

### Controller Logic
In our controllers we need to write the logic for destinguishing between web browser requests and API requests.

#### Logging in users
 
 First, let's start with the user(s) logging-in:
 
 ```ruby
 # app/controllers/authentication_controller.rb

class AuthenticationController < ApplicationController
  skip_before_action :authenticate_request

  def authenticate
    command = AuthenticateUser.call(params[:email], params[:password])

    if command.success?
      render json: { auth_token: command.result }
    else
      render json: { error: command.errors }, status: :unauthorized
    end
  end
end
 ```
The `authenticate` action will take the JSON parameters for e-mail and password through the `params` hash and pass them to the `AuthenticateUser` command. If the command succeeds, it will send the JWT  token back to the user.
 
Let's put a endpoint for the action:
 
```ruby
  #config/routes.rb
  post 'authenticate', to: 'authentication#authenticate'
```

There are different strategies for handling Cross-Site Request Forgery (CSRF). One for `html` requests and another for `json` or API request.
- One option is to put logic in the `application_controller.rb` to check if the request is `html` or `json` and protect accordingly. 


```ruby
#app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
    protect_from_forgery if: :json_request # return null session when API call
    protect_from_forgery with: :exception, unless: :json_request
    
    before_action :authenticate_request, if: :json_request
    before_action :authenticate_user!, unless: :json_request
    
    attr_reader :current_user
    
    private
    
    def json_request
        request.format.json?
    end
    
    def authenticate_request
        @current_user = AuthorizeApiRequest.call(request.headers).result
        render json: { error: 'Not Authorized' }, status: 401 unless @current_user
    end
end

```

- Another option would be to make an API specific controller, as follows:

```ruby
#app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
    protect_from_forgery with: :exception, unless: :json_request
    before_action :authenticate_user!, unless: :json_request
    
    protected
    
    def json_request
        request.format.json?
    end
end
```

```ruby
#app/controllers/api_base_controller.rb
class ApiBaseController < ApplicationController
    protect_from_forgery if: :json_request # return null session when API call
    before_action :authenticate_request, if: :json_request
    
    attr_reader :current_user
    
    private
    
    def authenticate_request
        @current_user = AuthorizeApiRequest.call(request.headers).result
        render json: { error: 'Not Authorized' }, status: 401 unless @current_user
    end
end
```

Now you can subclass `ApiBaseController` for your API controllers e.g

```ruby
#app/controllers/some_api_controller.rb
class SomeApiController < ApiBaseController
    def action
        # your action code here
    end
end
```

