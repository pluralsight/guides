I found a [great tutorial here on pluralsigt](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api) and having a spinoff problem to tackle, thought I might add some tweaks to make it work with a dual system, expected to work with both sessions and tokens. 

This guide focuses on RoR applications that use the [Devise](https://github.com/plataformatec/devise) for registration and sign in for web access and want to have token based authentication for api calls, whether from front-end platforms or mobile apps.

While this approach targets a monolith application structure, which is frowned upon for good reasons, it might be usefull for POCs and quick mocks.

# What is token-based authentication?
[Read about it in this greate guide](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api#what-is-token-based-authentication-)

# Setting up Dual authentication with Rails 4
Credits: parts of this tutorial where copied from [this guide](https://www.pluralsight.com/guides/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api) by [Haristo Georgiev](https://www.pluralsight.com/guides/author/Kaizeras)

I assume you already have a Rails 4 application set up to work with [Devise](https://github.com/plataformatec/devise). I also assume a vanilla [Devise](https://github.com/plataformatec/devise) setup i.e. You did not overide any controllers yet and you use at least :database_authenticatable, :registerable stratagies.


he implementation of the JWT token generation can start. First, the `jwt` gem will make encoding and decoding of HMACSHA256 tokens available in the Rails application. Let's install it:

 
 ```ruby
  gem 'jwt'
```
And install it:

```bash
bundle install

```
Once the  gem is installed, it can be accessed through the `JWT` global variable. Because the methods that are going to be used require encapsulation, a singleton class is  a great way of wrapping the logic and using it in other constructs. 

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
The first method,`encode`, takes three parameters -- the user id, the expiration time (1 day), and the unique base key of your Rails application -- to create a unique token. 

The second method,`decode` takes the token and uses the application's secret key to decode it. 

These are the two cases in which these methods will be used:
 * For authenticating the user and generate a token for him/her using `encode`
 * To check if the user's token appended in each request is correct by using `decode`. 

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
 Instead of using private controller methods, `simple_command` can be used: [simple_command](https://github.com/nebulab/simple_command).

The simple command gem is an easy way of creating services. Its role is similar to the role of a helper, but instead of facilitating the connection between the controller and the view, it does the same for the controller and the model. In this way, we can shorten the code in the models and controllers.

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
The command has to take the user's e-mail and password and return the user if the credentials match. Here is how this can be done:
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
    return user if user && user.authenticate(password)

    errors.add :user_authentication, 'invalid credentials'
    nil
  end
end
```