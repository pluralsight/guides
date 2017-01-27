# HyperReact and Rails

This quick tutorial will take you through all the steps necessary to get simple HyperReact components rendering in a new Rails application.

The source of this tutorial is avaiable as a [Quickstart](https://github.com/ruby-hyperloop/quickstart) which you can clone.

## Using HyperRails

Firstly, let's create a new Rails app. If you would like to add HyperReact to an existing app simply skip this step and pick up from where we add the HyperRails gem.

`rails new hyperloop-app`

Next add the [HypreRails gem](https://github.com/ruby-hyperloop/hyper-rails), which is a set of generators which will easily configure the other Hyperloop gems:

```ruby
# In your Gemfile
gem 'hyper-rails'
```
then

`bundle install`

Now let's get the HyperRails generator to install Hyperloop:

```ruby
rails g hyperloop:install
bundle update
```

HyperRails will add all the necessary Gem's and configuration to our new Rails app. If you are interested in the steps the generator has completed, please see the [Manual Rails Install](http://ruby-hyperloop.io/installation/#manual-rails-install) section on the website.

## Adding a component

Next we will ask the generator to create a simple HyperReact component for us:

```ruby
rails g hyperloop:component Home::Welcome
```

which will generate this component template for us:

```ruby
# app/views/components/home/welcome.rb
module Components
  module Home
    class Welcome < React::Component::Base

      # param :my_param
      # param param_with_default: "default value"
      # param :param_with_default2, default: "default value" # alternative syntax
      # param :param_with_type, type: Hash
      # param :array_of_hashes, type: [Hash]
      # collect_all_other_params_as :attributes  #collects all other params into a hash

      # The following are the most common lifecycle call backs,
      # the following are the most common lifecycle call backs# delete any that you are not using.
      # call backs may also reference an instance method i.e. before_mount :my_method

      before_mount do
        # any initialization particularly of state variables goes here.
        # this will execute on server (prerendering) and client.
      end

      after_mount do
        # any client only post rendering initialization goes here.
        # i.e. start timers, HTTP requests, and low level jquery operations etc.
      end

      before_update do
        # called whenever a component will be re-rerendered
      end

      before_unmount do
        # cleanup any thing (i.e. timers) before component is destroyed
      end

      def render
        div do
          "Home::Welcome"
        end
      end
    end
  end
end
```

Notice that a HyperReact component is just a Ruby class which inherits from `React::Component::Base`. This component, `Welcome`, is in the `Home` namespace. This is a great way to organize components.

The params at the top of the class are the parameters that will be passed into the component when it is created. For example, we could declare a parameter like this:

```ruby
param :name, type: String
```

Next come the lifecycle call-backs. You can read more about these in the [HyperReact documentation](http://ruby-hyperloop.io/docs/lifecycle_callbacks/).

Finally we have the `render` method. Every HyperReact component will have a `render` method. There are some exceptions, which you can learn more about in the [Flux Store Tutorial](http://ruby-hyperloop.io/tutorials/flux_store/) later.

Back to our example, the `render` method must render just one HTML element, so as you can see, this method render just one `div` element. Note that `div` and `DIV` are functionally the same - all the HTML elements can be upper or lower case depending on which style you prefer.  

Let's cut this component down to just what we need and give it some functionality:

```ruby
# app/views/components/home/welcome.rb
module Components
  module Home
    class Welcome < React::Component::Base

      param :name, type: String

      def render
        h1 { "Hello and welcome #{params.name}!" }
      end
    end
  end
end
```

## Controller Rendering

And then finally we need to create a controller to render the component. We could do this through a view, but in this case we will use the `render_component` shortcut to create the view for us dynamically.

**Note** how we pass a value to our `name` paramater.

```ruby
# app/controllers/home_controller.rb
class HomeController < ApplicationController
  def welcome
    render_component name: "Sally"
  end
end
```

Very last step is to create a route:

```ruby
#config/routes.rb
root 'home#welcome'
```

In a few simple steps, we have created a Rails application that uses Opal to compile Ruby and

pre-renders (on the server) then renders React components in your browser - all in just a few lines of Ruby code. **Welcome to Hyperloop!**

Make sure you see the [Docs](http://ruby-hyperloop.io/docs/dsl_overview/) for more information on the DSL.

Please be sure to see the other tutorials on the [Hyperloop site](http://localhost:4567/tutorials)






