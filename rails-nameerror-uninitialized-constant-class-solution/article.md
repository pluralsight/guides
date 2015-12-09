Often, **rails nameerror uninitialized constant class** will occur if your rails console is not loaded with configuration of the class file containing method being called.

### Problem

Basically this can happen if you happened to **call the function from the class which is not loaded with configuration** when your rails console/server was started.

## Example

Suppose you create a file named Util.rb in the lib folder of your Rails project.

``` ruby
Class Util
   def self.get_date
 Time.now.strftime(‘%F’)
   end
end
```

And then if you want to call the function through command line i.e. rails console probably you will start rails console and and call the method get_date using class name Util as follows,

``` ruby
rails console
> Util.get_date
```


This will give you following error: `NameError: uninitialized constant Util`
It means that your class was not loaded when the rails console was started

### Solution

To resolve this problem your class has to be loaded in environment, this can be done in following way,

1\. Open **application.rb** file of your Rails project
2\. **Add** following line in application.rb

``` ruby
config.autoload_paths += %W(#{config.root}/lib)
```

Your application.rb will look something like,

``` ruby
require File.expand_path('../boot', __FILE__)
require 'rails/all'

# Require the gems listed in Gemfile, including any gems
# you've limited to :test, :development, or :production.
Bundler.require(:default, Rails.env)

module RubyInRailsApp
  class Application &lt; Rails::Application
    # the new line added for autoload of lib
    config.autoload_paths += %W(#{config.root}/lib)

    # Settings in config/environments/* take precedence over those specified here.
    # Application configuration should go into files in config/initializers
    # -- all .rb files in that directory are automatically loaded.

    # Set Time.zone default to the specified zone and make Active Record auto-convert to this zone.
    # Run "rake -D time" for a list of tasks for finding time zone names. Default is UTC.
    # config.time_zone = 'Central Time (US &amp; Canada)'

    # The default locale is :en and all translations from config/locales/*.rb,yml are auto loaded.
    # config.i18n.load_path += Dir[Rails.root.join('my', 'locales', '*.{rb,yml}').to_s]
    # config.i18n.default_locale = :de
  end
end
```


3\. **Done**

### Conclusion

Thus, **Setting autoload_paths for config in application.rb** will result for - ruby files present in lib directory of your Rails project to **get automatically loaded** when your console is started. Calling method will not give rails nameerror uninitialized constant class error anymore.
Now, calling the method -

``` ruby
Util.get_date
```

will return result as expected. As the class was loaded with configuration when your console was started thus Name resolution was successful.

Ultimately these kind of errors can be reduced if you configure your Rails application properly. Read [Configuring Rails application](http://edgeguides.rubyonrails.org/configuring.html) to know more about configurations.

