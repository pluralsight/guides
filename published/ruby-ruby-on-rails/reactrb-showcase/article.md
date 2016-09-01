# Reactrb Showcase

This is a simple Rails application showcasing Reactrb, Opal, NPM, Webpack, React Bootstrap, Reactive Record, Synchromesh, Rearcrb Router and other associated technologies.  This showcase is intended to be a companion project to the excellent Reactrb tutorials already written which you can find in [Further Reading](#further-reading) below.

This Showcase application will mix native React and Reactrb components, be styled by Bootstrap CSS (using ReactBootstrap), display a video (using a native React component) and use Reactrb Reactive Record and Synchromesh to handle data for a Post and Comments feed.

The Showcase application will look like this:

![Screen](public/screen.png)

### Technologies highlighted in this Showcase application

+ For the backend we are using Rails 4.2.6 with Ruby 2.3.1
+ NPM and Webpack to manage front end assets
+ [React Rails](https://github.com/reactjs/react-rails) to use React with Rails
+ [Reactrb](https://github.com/reactrb/reactrb) to write reactive UI components with Ruby's elegance
+ [React Bootstrap](https://react-bootstrap.github.io/) to show how to use native React components in Reactrb
+ [Reactive Record](https://github.com/reactrb/reactive-record) to move data between Rails models and the front end
+ [Synchromesh](https://github.com/reactrb/synchromesh) to magically push changed data between all connected clients
+ [Reactrb Hot-Reloader and Opal IRB](https://github.com/fkchang/opal-hot-reloader) for programmer joy and hot-loading with developing


## Introduction

### Introductions to Reactrb
+ [An overview of Reactrb by Mitch VanDuyn](http://slides.com/mitchvanduyn/deck-1-3#/)
+ [Power of React-js with the joy of Ruby by Forrest Chang](http://www.slideshare.net/fkchang/reactrb-all-the-power-of-reactjs-with-all-the-joy-of-ruby)

### Reactrb Help and Questions

+ [Gitter.im](https://gitter.im/reactrb/chat) for general questions, discussion, and interactive help.
+ [Stack Overflow](http://stackoverflow.com/questions/tagged/reactrb) tag `reactrb` for specific problems.
+ [Github Issues](https://github.com/reactrb/reactrb/issues) for bugs, feature enhancements, etc.
+ [Further reading](#further-reading) at the end of this tutorial

### Using NPM and Webpack alongside Rails

Ruby libraries are distributed as gems, and are managed in your Rails app using the Gemfile and bundler.

In the Javascript world things are still evolving but I have found that the easiest way to manage Javascript libraries is using NPM (Node Package Manager) and Webpack.  Pretty much every front end library is packaged with NPM these days so it is easy to get help and most things just work.

Happily NPM, Webpack, Rails, and Reactrb can all play together very nicely.

+ [NPM](https://www.npmjs.com/)
+ [Webpack](https://www.npmjs.com/package/webpack)

This tutorial requires that Ruby, Rails, NPM and Webpack are installed. Please see their websites for installation instructions.

## Setup

### Step 1: Creating a new Rails application

	rails new reactrb-showcase
	cd reactrb-showcase
	bundle install

You should have a empty Rails application

	bundle exec rails s

And in your browser

	http://localhost:3000/

You should be seeing the Rails Welcome aboard page. Great, Rails is now installed. Lets get started with the interesting stuff.

### Step 2: Adding Reactrb

[We will use the Reactrb Rails Generator Gem](https://github.com/loicboutet/reactive-rails-generator)

In your `Gemfile` under the development group add

	gem "reactrb-rails-generator"

then

	bundle install
	bundle exec rails g reactrb:install --all
	bundle update

At this stage Reactrb is installed but we don't have any components yet. Lets create one via the generator:

	rails g reactrb:component Home::Show

This will add a new Component at app/views/components/home/show.rb

```ruby
module Components
  module Home
    class Show < React::Component::Base

      # param :my_param
      # param param_with_default: "default value"
      # param :param_with_default2, default: "default value" # alternative syntax
      # param :param_with_type, type: Hash
      # param :array_of_hashes, type: [Hash]
      # collect_all_other_params_as :attributes  #collects all other params into a hash

      # The following are the most common lifecycle call backs,
      # the following are the most common lifecycle call backs
			# delete any that you are not using.
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
          "Home::Show"
        end
      end
    end
  end
end
```

Have a look at this component as the generator creates a boilerplate component and instructions for using the most common Reactrb macros. Note that Reactrb components normally inherit from class `React::Component::Base` but you are free to `include React::Component` instead if you need your component to inherit from some other class.

Also note how `params` are declared and how `before_mount` macros (and friends) macros are used. Finally note that every component must have one `render` method which must return just one DOM `element` which in this example case is a `div`.

Next let's get this simple component rendering on a page. For that we will need a rails controller and a route.

	rails g controller home

And add a route to your `routes.rb`

	root 'home#show'

And a `show` method in the HomeController which will render the component using the `render_component` helper.

```ruby
class HomeController < ApplicationController
	def show
		render_component
	end
end
```

Fire up the server with `bundle exec rails s`, refresh your browser and if all has gone well, you should be rewarded with `Home::Show` in your browser.

If you open your JavaScript console you can also check which version of React has been loaded.

	React.version

Remember this value, as we will need to use it later.

### Step 3: Webpack for managing front-end assets

There are three parts to this step:  

* Setting up NPM (node package manager) for the project
* Setting up Webpack
* Updating the rails asset pipeline to use the bundles generated by Webpack

This is just a matter of adding 4 boiler plate files, and updating two of your rails files.

First add a package.json file to your root directory (same place as your Gemfile) like this:

	// package.json
	{
		"name": "reactrb-showcase",
		"version": "0.0.1",
		"dependencies": {
			"bootstrap": "^3.3.6",
			"react": "^0.14.2",      
			"react-dom": "^0.14.2",
			"react-bootstrap": "^0.29.5",
			"webpack": "^1.13.1"
		},
			"devDependencies": {
		}
	}

Notice how similar this is to your Gemfile.

Now run `npm install` which will make sure you have all these packages.

So that we can run Webpack from the command line do a `npm install webpack -g`

Now that we have Webpack, we need to add 3 boiler plate files to configure it.  As you add more javascript packages you will be updating these files.  Again this is similar to updating your Gemfile when you add new gems to a project.

Add webpack.config.js to the root of your project:

```javascript
var path = require("path");

module.exports = {
    context: __dirname,
    entry: {
      client_only:  "./webpack/client_only.js",
      client_and_server: "./webpack/client_and_server.js"
    },
    output: {
      path: path.join(__dirname, 'app', 'assets',   'javascripts', 'webpack'),
      filename: "[name].js",
      publicPath: "/webpack/"
    },
    module: {
      loaders: [
        // add any loaders here
      ]
    },
    resolve: {
      root: path.join(__dirname, '..', 'webpack')
    },
};
```
and create a folder called `webpack` and add the following two files:

```javascript
// webpack/client_only.js
// any packages that depend specifically on the DOM go here
// for example the webpack css loader generates code that will break prerendering
console.log('client_only.js loaded');
```

```javascript
// webpack/client_and_server.js
// all other packages that you can run on both server (prerendering) and client go here
// most well behaved packages can be required here
ReactDOM = require('react-dom')
React = require('react')
console.log('client_and_server.js loaded')
```

Now run `webpack` from the command line.  This will grab all necessary dependencies and package them up into the `client_and_server.js` and `client_only.js` bundles.  If you look in the `app/assets/javascripts/webpack` directory you should see the two files there.

Finally we need to require these two bundles into our rails asset pipeline.

Edit `app/assets/javascript/application.js` and add
```javascript
//= require 'webpack/client_only'
```
just *above* the line that reads `Opal.load('components');`.  This will pull in any webpack assets that can only run on the client.

Then edit `app/views/components.rb` and replace the `require 'react'` line with
```ruby
require 'webpack/client_and_server.js'
```

In otherwords instead of pulling in react from the react-rails gem, we are going to pull in react *and* any other javascript packages we want from our webpack bundle.

Reactrb can automatically access our components loaded by Webpack, but we have to opt in to this behavior.  Edit `app/views/components.rb` and add
```ruby
require 'reactrb/auto-import'
```

immediately after `require 'reactrb'` (which is right near the top of the file.)  Auto-import will now search the javascript name space, and import into ruby any components that are referenced by your Reactrb components.

Now run `bundle exec rails s` and refresh the browser.  Look at the console and you should see something like this:

```text
client_and_server.js loaded
client_only.js loaded
client_and_server.js loaded
************************ React Prerendering Context Initialized Show ***********************
************************ React Browser Context Initialized ****************************
Reactive record prerendered data being loaded: [Object]
```

Congratulations you are setup and ready to begin adding javascript packages to your application.

## Working with native React components

It is time to reap some of the rewards from all the hard work above. We have everything setup so we can easily add front end components and work with them in Reactrb. Lets jump in and add a native React component that plays a video.

[We are going to use Pete Cook's React rplayr](https://github.com/CookPete/rplayr)

First let's install the component via NPM:
```text
npm install react-player --save
```

Next we need to `require` it in `webpack/client_and_server.js`
```javascript
ReactPlayer = require('react-player')
```

Next run webpack so it can be bundled
```text
webpack
```

And then finally let's add it to our Show component:
```ruby
def render
  div do
    ReactPlayer(url:  'https://www.youtube.com/embed/FzCsDVfPQqk',
      playing: true
    )
  end
end
```

Refresh your browser and you should have a video. How simple was that!

## Working with React Bootstrap

[We will be using React Bootstrap which is a native React library](https://react-bootstrap.github.io/)

The main purpose for React Bootstrap is that it abstracts away verbose HTML & CSS code into React components which makes it a lot cleaner for React JSX developers. One of the very lovely things about Reactrb is that we already work in beautiful Ruby. To emphasise this point, consider the following:

Sample 1 - In HTML (without React Bootstrap):

	<button id="something-btn" type="button" class="btn btn-success btn-sm">
	  Something
	</button>
	$('#something-btn').click(someCallback);

Sample 2 - In JSX (with React Bootstrap components):

	<Button bsStyle="success" bsSize="small" onClick={someCallback}>
	  Something
	</Button>

Sample 3 - In Reactrb (without React Bootstrap):

	button.btn_success.btn_sm {'Something'}.on(:click) do
		someMethod
	end

Sample 4 - In Reactrb (with React Bootstrap):

	Bs.Button(bsStyle: 'success' bsSize: "small") {'Something'}.on(:click) do
		someMethod
	end

As you can see, sample 3 & 4 are not that different and as a Reactrb developer, I actually prefer sample 3. If I were a JavaScript or JSX developer I would completely understand the advantage of abstracting Bootstrap CSS into React Components so I don't have to work directly with CSS and JavaScript but this is not the case with Reactrb as CSS classes are added to HTML elements with simple dot notation:

	span.pull_right {}

compiles to (note the conversion from _ to -)

	<span class='pull-right'></span>

So I hear you ask: why if I prefer the non-React Bootstrap syntax why am worrying about React Bootstrap? For one very simple reason: components like Navbar and Modal that requires `bootstrap.js` will not work with React on it's own so without the React Bootstrap project you would need to implement all that functionality yourself. The React Bootstrap project has re-implemented all this functionality as React components.

Lets implement a Navbar in this project using React Bootstrap in Reactrb. First, we need to install Bootstrap and React Bootstrap:

	npm install bootstrap react-bootstrap --save

Note: The `--save` option will update the package.json file.

And then we need to `require` it in `webpack/client_and_server.js` by adding this line:
```javascript
ReactBootstrap = require('react-bootstrap')
```
Run the `webpack` command again, and restart your rails server.

If you refresh your browser now and open the JavaScript console we will be able to interact with React Bootstrap by typing:

In the JavaScript console type: ```ReactBootstrap```

and you will see the ReactBootstrap object with all its components like Accordion, Alert, Badge, Breadcrumb, etc. This is great news, React Bootstrap is installed and ready to use. Accessing the JavaScript object in this way is a really great way to see what you have to work with. Sometimes the documentation of a component is not as accurate as actually seeing what you have in the component itself.

To make sure everything is working lets add a *Button* to our our Show component like this:

```ruby
module Components
  module Home
    class Show < React::Component::Base
      def render
        ReactBootstrap::Button(bsStyle: 'success', bsSize: "small") do
          'Success'
        end.on(:click) do
          alert('you clicked me!')
        end
      end
    end
  end
end
```
Notice that we reference `ReactBoostrap` in ruby using the same identifer that was in the require statement in our `client_and_server.js` webpack bundle.  The first time Reactrb hits the `ReactBootstrap` constant it will not be defined. This triggers a search of the javascript name space for something that looks either like a component or library of components.  It then defines the appropriate module or component class wrapper in ruby.

Visit your page and if all is well you will see a clickable button.  However it will not have any styles.  This is because ReactBootstrap does not automatically depend on any particular style sheet, so we will have to supply one.  An easy way to do this is to just copy the css file from the bootstrap repo, and stuff it our rails assets directory, however with a little upfront work we can setup webpack to do it all for us.

First lets add four webpack *loaders* using npm:
```text
npm install css-loader file-loader style-loader url-loader --save-dev
```
Notice we use `--save-dev` instead of just `--save` as these packages are only used in the development process.

Now edit your `webpack.config.js` file, and update the loaders section so it looks like this:

```javascript
var path = require("path");

module.exports = {
...
    module: {
      loaders: [
        { test: /\.css$/,
          loader: "style-loader!css-loader"
        },
        { test: /\.(woff|woff2)(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'url?limit=10000&mimetype=application/font-woff'
        },
        { test: /\.ttf(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'url?limit=10000&mimetype=application/octet-stream'
        },
        { test: /\.eot(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'file'
        },
        { test: /\.svg(\?v=\d+\.\d+\.\d+)?$/,
          loader: 'url?limit=10000&mimetype=image/svg+xml'
        }
      ]
    },
...
};
```

We have set webpack up so that when a css file is required it uses the style loader to process the file.  Because the bootstrap css file will require font face files, we also have 4 font loaders.  All this will package up everything when we require any css file.

Now we are ready to require CSS files, and have webpack build a complete bundle including the css and any fonts referenced.

To bundle in the bootstrap css file add this line to `webpack/client_only.js`
```javascript
require('bootstrap/dist/css/bootstrap.css');
```

And install the bootstrap package
```text
npm install bootstrap --save
```

Now run `webpack` to update our bundles, and restart your server.  Now our button is properly styled you should be rewarded with a nice Bootstrap styled green Success Button.

Now that everything is loaded, lets update our component to use a few more of the Bootstrap components.  Update your Show component so that it looks like this:

```ruby
module Components
  module Home
    class Show < React::Component::Base

      def say_hello(i)
        alert "Hello from number #{i}"
      end

      def render
        div do
          ReactBootstrap::Navbar(bsStyle: :inverse) do
            ReactBootstrap::Nav() do
              ReactBootstrap::NavbarBrand() do
                a(href: '#') { 'Reactrb Showcase' }
              end
              ReactBootstrap::NavDropdown(
                eventKey: 1,
                title: 'Things',
                id: :drop_down
              ) do
                (1..5).each do |n|
                  ReactBootstrap::MenuItem(href: '#',
                    key: n,
                    eventKey: "1.#{n}"
                  ) do
                    "Number #{n}"
                  end.on(:click) { say_hello(n) }
                end
              end
            end
          end
          div.container do
            ReactPlayer(url: 'https://www.youtube.com/embed/FzCsDVfPQqk',
              playing: true
            )
          end
        end
      end
    end
  end
end
```

A few things to notice in the code above:

We add React Bootstrap components simply by `ReactBootstrap::Name` where `Name` is the JavaScriot component you want to render. All the components are documented in the React Bootstrap [documentation](https://react-bootstrap.github.io/components.html)

See with `div.container` we are mixing in CSS style which will compile into `<div class='container'>`

Also notice how I have added an `.on(:click)` event handler to the `MenuItem` component while setting `href: '#'` as this will allow us to handle the event instead of navigating to a new page.

So far we have a very basic application which is looking OK and showing a video. Time to do something a little more interesting. How about if we add Post and Comment functionality which will let us explore Reactive Record!

## Using Reactrb Reactive Record

[We will be using the Reactive Record gem](https://github.com/reactrb/reactive-record)

Reactive Record compiles your Active Record models so they are accessible to the front-end and implements an API based on your models and their associations. Lazy loads just the data that is needed to render a component and is fully integrated with Reactrb and paired with Synchromesh to push database changes to all connected clients. ReactiveRecord and Synchromesh give you Relay + GraphQL like functionality with a fraction of the effort and complexity (the original idea for Reactive Record is credited to [Volt](https://github.com/voltrb/volt) and not Relay).

### Installing Reactive Record

Installing Reactive Record is straight forward.

First add this line to your application's Gemfile:

```ruby
gem 'reactive-record'
```

And then execute:

```
$ bundle install
```

Finally you need to add a line to your `routes.rb`:

```ruby
mount ReactiveRecord::Engine => '/rr'
```

### Creating the models

We are going to need a few models to work with so let's go ahead and create those now.

```text
rails g model Post
rails g model Comment post:references
```

And then before you run the migrations, lets flesh them out a little so they look like this:

```ruby
# db/migrate/..create_posts.rb
class CreatePosts < ActiveRecord::Migration
  def change
    create_table :posts do |t|
      t.string :body
      t.timestamps null: false
    end
  end
end

# db/migrate/..create_comments.rb
class CreateComments < ActiveRecord::Migration
  def change
    create_table :comments do |t|
      t.references :post, index: true, foreign_key: true
      t.string :body
      t.timestamps null: false
    end
  end
end
```

Now would be a good time to run the migrations:

```text
rake db:migrate
```

### Making your models accessible to Reactive Record

Reactive Record needs to 'see' your models as a representation of them get compiled into JavaScript along with your Reactrb components so they are accessible in your client side code.

The convention (though this is choice and you can change this if you prefer) is to create a `public` folder under `models` and then provide a linkage file which will `require_tree` your models when compiling `components.rb`.

Create a new folder:

```text
models/public
```

Then move `post.rb` and `comment.rb` to `models/public`

```text
$ mv app/models/post.rb app/models/public
$ mv app/models/comment.rb app/models/public
```

Next create `_react_public_models.rb` in your models folder:

```ruby
# models/_react_public_models.rb
require_tree './public'
```

Finally add a line to your `views/components.rb` file:

```ruby
# views/components.rb
...
require '_react_public_models'
```

### Model Associations

Reactive Record is particular about both sides of an association being specified. If you forget to do this you will see warnings to this effect.

```ruby
# models/public/post.rb
class Post < ActiveRecord::Base
  has_many :comments
end

# models/public/comment.rb
class Comment < ActiveRecord::Base
  belongs_to :post
end
```

### Accessing your models in Reactrb components

To get started, lets create a new component which will display a list of Posts and Comments under the video:

```ruby
# views/components/show.rb
...
div.container do
  ReactPlayer(url: 'https://www.youtube.com/embed/FzCsDVfPQqk', playing: true)
  br # line break
  PostsList()
end
...
```

Note that to place a Reactrb component you either need to include ( ) or { }, so `PostsList()` or `PostsList { }` would be valid but just `PostsList` would not.

Next lets create the `PostsList` component:

```ruby
module Components
  module Home
    class PostsList < React::Component::Base
      define_state :new_post, ""

      before_mount do
        # note that this will lazy load posts
        # and only the fields that are needed will be requested
        @posts = Post.all
      end

      def render
        div do
          new_post
          ul.list_unstyled do
            @posts.reverse.each do |post|
              PostListItem(post: post)
              CommentsList(comments: post.comments)
            end
          end
        end
      end

      def new_post
        ReactBootstrap::FormGroup() do
          ReactBootstrap::FormControl(
            value: state.new_post,
            type: :text,
          ).on(:change) { |e|
            state.new_post! e.target.value
          }
        end
        ReactBootstrap::Button(bsStyle: :primary) do
          "Post"
        end.on(:click) { save_new_post }
      end

      def save_new_post
        post = Post.new(body: state.new_post)
        post.save do |result|
          # note that save is a promise so this code will only run after the save
          # yet react will move onto the code after this (before the save happens)
          alert "unable to save" unless result == true
        end
        state.new_post! ""
      end
    end

    class PostListItem < React::Component::Base
      param :post

      def render
        li do
          # note how you access post.body just like with Active Record
          h4 { params.post.body }
        end
      end

    end
  end
end
```

Things to note in the code above:

See how we fetch the Reactive Record Post collection in `before_mount`. Setting this here instead of in `after_mount` means that we do not need to worry about `@posts` being `nil` as the collection will always contain at least one entry with the actual records being lazy loaded when needed.

Note how we are binding the state variable `new_post` to the `FormControl` and then setting its value based on the value being passed to the `.on(:change)` block. This is a standard React pattern.

Also see how we are saving the new post where Reactive Record's save returns a promise which means that the block after save is only evaluated when it returns yet React would have moved on to the rest of the code.

Finally note that there is no code which checks to see if there are new posts yet when you run this, the list of posts remains magically up-to-date.

Welcome to the wonderful of Reactive Record and React!

## Synchromesh

[We will be using the Synchromesh gem](https://github.com/reactrb/synchromesh)

Reactive Record is the data layer between one client and its server and Synchromesh uses push notifications to push changed records to all connected Reactive Record clients.

Synchromesh is incredibly simple to setup. Add this line to your Gemfile:

```ruby
gem 'synchromesh', git: "https://github.com/reactrb/synchromesh.git"
```

And then execute:

``` text
$ bundle install
```

Next add this line to your `components.rb`:

```ruby
require 'synchromesh'
```

Finally, you need to add an initialiser `config/initializers/synchromesh.rb`

```ruby
# config/initializers/synchromesh.rb
Synchromesh.configuration do |config|
  # this is the initialiser for polling, see the synchromesh
  # documentation for using pusher.com
  config.transport = :simple_poller
  config.channel_prefix = "synchromesh"
  config.opts = {
    seconds_between_poll: 1.second,
    seconds_polled_data_will_be_retained: 1.hour
  }
end
```

Restart your server, open two browser windows and be amazed to see any new posts added to one session magically appearing in the other!

Todo:
+ Reactrb Router

## Reactrb Hot-reloader and Opal IRB

Before we go any further, let's install too fantastic tools written by Forrest Chang:

+ [Opal Hot Reloader](https://github.com/fkchang/opal-hot-reloader)
+ [Opal Console](https://github.com/fkchang/opal-console)

Opal Hot Loader is for pure programmer joy (not having to reload the page to compile your source) and the Opal console is incredibly useful to test how Ruby code compiles to JavaScript.

We are also going to add the Foreman gem to run our Rails server and the Hot Loader service for us.

Add the following lines to your `gemfile` and run `bundle`:

```ruby
gem 'opal_hot_reloader', git: 'https://github.com/fkchang/opal-hot-reloader.git'
gem 'foreman'
```

`bundle install`

Modify your `components.rb`, adding the following lines inside the if statement so they only run on the client and not as part of the server pre-rendering process:

```ruby
require 'opal_hot_reloader'
OpalHotReloader.listen(25222, true)
```

Then modify your `procfile` so that the Hot Loader service will start whenever you start your server:

```text
rails: bundle exec rails server
hotloader: opal-hot-reloader -p 25222 -d app/views/components
```

To start both servers:

`foreman start`

Refresh your browser for the last time and try modifying your `show.rb` component and you should see your changes appearing magically in your browser as you save. Pure joy.  

## Further reading

### Other Reactrb tutorials and examples
+ [Getting started with Reactrb and Rails](https://github.com/loicboutet/reactrb_tutorial)
+ [ChatRB Demo App](https://github.com/reactrb/reactrb.github.io/blob/master/docs/tutorial.md)
+ [Reactive Record sample ToDo app](https://github.com/loicboutet/reactivetodo)
+ [Flux pattern in Reactrb](https://github.com/reactrb/reactrb.github.io/wiki/Sending-data-from-deeply-nested-components)
+ [Getting with Reactrb, React Bootstrap and Webpack](https://github.com/fkchang/getting-started-reactrb-webpack)

### Other Reactrb resources
+ [Reactrb website](http://reactrb.org/)
+ [Reactrb GitHub site](https://github.com/reactrb/reactrb)

### Reactrb is powered by React

Reactrb and friends are in most cases simple DSL Ruby wrappers to the underlying native JavaScript libraries and React Components. It is really important to have a solid grip on how these technologies work to complement your understanding of Reactrb. Most searches for help on Google will take you to examples written in JSX or ES6 JavaScript but you will learn over time to transalte this to Reactrb equivalents. To make headway with Reactrb you do need a solid understanding of the underlying philosophy of React and its component based architecture. The 'Thinking in React' tutorial below is an excellent place to start. (Make sure you see the Flux pattern in Reactrb above for an example of how to communicate between grandparent and child components).   

+ [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html)
+ [React](https://facebook.github.io/react/docs/getting-started.html)
+ [React Router](https://github.com/reactjs/react-router)
+ [React Bootstrap](https://react-bootstrap.github.io/)

### Opal under the covers

Reactrb is a DSL wrapper of React which uses Opal to compile Ruby code to ES5 native JavaScript. If you have not used Opal before then you should at a minimum read the excellent guides as they will teach you enough to get you started with Reactrb.

+ [Opal](http://opalrb.org/)
+ [Opal Guides](http://opalrb.org/docs/guides/v0.9.2/index.html)
+ [To see the full power of Opal in action watch this video](https://www.youtube.com/watch?v=vhIrrlcWphU)
