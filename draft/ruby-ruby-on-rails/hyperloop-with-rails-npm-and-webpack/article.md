# HyperReact, NPM and Webpack

## Overview and prerequisites

In an Isomorphic Ruby world, we need a good way of including Ruby and JavaScript components so they co-exist and play nicely together.

Most Ruby libraries are distributed as Gems whereas (these days) most Javascript components are distributed through NPM. Webpack co-exists well with Rails and the combination of Rails, Yarn, Sprockets and Webpack looks like it might become a standard within the Rails community.

This tutorial will take you through [Setup](#setup) of NPM and Webpack, [Usage](#usage) instructions for adding new components and then give an [Example](#example) of adding and using a JavaScript component in a HyperReact project.

This tutorial assumes that NPM is installed. Please see the [NPM website](https://www.npmjs.com/) for installation instructions.

Additionally, this tutorial starts with an existing Rails app with **HyperReact already setup and working**. For more information on how to get to that stage, please see the [HyperReact and Rails Tutorial](http://ruby-hyperloop.io/tutorials/hyperreact_with_rails/).

The source of this tutorial is avaiable as a [Quickstart](https://github.com/ruby-hyperloop/quickstart) which you can clone.

## Setup

If you have completed the prerequisites above, let's get started - there are 5 steps to this process:

1. Adding Webpack to your Rails project
2. Setting up Webpack
3. Using Webpack to build your client and server bundles
4. Installing React and ReactDOM via NPM
5. Adding Webpack bundles to the Rails asset pipeline


### Step 1 - Adding Webpack to your Rails project

Run these three commands:

`npm init` will create an empty package.json in your root folder

`npm install webpack --save-dev` install Webpack and create a node_modules folder

`npm install webpack -g` so we can run Webpack from the command line

The commands above will have created a `package.json` (similar concept to a Gemfile) and a `node_modules` folder containing hundreds of JavaScript dependancies that we do not need in the source of our project so let's tell git to ignore them by adding a `.gitignore` file:

```
#/.gitignore
/node_modules
```

### Step 2 - Setting up Webpack

Now that we have Webpack, we need to add 3 boiler plate files to configure it. As you add more JavaScript packages you will be updating these files. Again this is similar to updating your Gemfile when you add new gems to a project.

Add `webpack.config.js` to the root of your project:

```javascript
// webpack.config.js
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

Then create a folder called webpack and add the following two files:

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
console.log('client_and_server.js loaded')
```

### Step 3 - Using Webpack to build your client and server bundles

Simply run this command:

`webpack`

You should see a result something like this:

```
Hash: 756a1dc4a11c8fccd0a4
Version: webpack 1.14.0
Time: 55ms
               Asset     Size  Chunks             Chunk Names
client_and_server.js  1.61 kB       0  [emitted]  client_and_server
      client_only.js  1.61 kB       1  [emitted]  client_only
   [0] ./webpack/client_and_server.js 214 bytes {0} [built]
   [0] ./webpack/client_only.js 206 bytes {1} [built]```
```
Our `client_and_server.js` and `client_only.js` bundles are built and ready to be included in our application. If you look in your `app/assets/javascripts/webpack` folder you should see the two files there. Note that these bundles are empty at the moment as we have not added any JavaScript components yet. We will do that in Step 6.

### Step 4 - Adding Webpack bundles to the Rails asset pipeline

Finally we need to require these two bundles into our rails asset pipeline.

Edit `app/assets/javascripts/application.js` and add

```ruby
//= require 'webpack/client_only'
```

Then edit `app/views/components.rb` and directly after `require 'hyper-react'` add the following two lines:

```ruby
require 'webpack/client_and_server.js'
require 'reactrb/auto-import'
```

### Step 5 - Installing React and ReactDOM

This step is really simple as we have NPM installed. Just run these two commands:

`npm install react --save` to install React
`npm install react-dom --save` to install ReactDOM

Note how this modifies your 'package.json' and installs React and ReactDOM in your `node_modules` folder.

Next we need to `require` them into `client_and_server.js`:

```javascript
// webpack/client_and_server.js
ReactDOM = require('react-dom')
React = require('react')
```

And finally we need to run `webpack` to add them to the bundle:

```
webpack
```

The output should look something like this:

```
Hash: db338e21dcc44f66e5b5
Version: webpack 1.14.0
Time: 737ms
               Asset     Size  Chunks             Chunk Names
client_and_server.js   740 kB       0  [emitted]  client_and_server
      client_only.js  1.61 kB       1  [emitted]  client_only
   [0] ./webpack/client_and_server.js 271 bytes {0} [built]
   [0] ./webpack/client_only.js 206 bytes {1} [built]
    + 177 hidden modules
```

Note that `client_and_server.js` has gone from 1.61kB to 740kB as it now includes the source of React and ReactDOM.

## Usage

Now that we have completed the setup, using NPM and Webpack to include new JavaScript components is simply a case of running Step 6 again for each component you would like to add. You simply follow these three steps:

1. Install the component using NPM `npm install xx --save`
2. `require` the component into either `webpack/client_and_server.js ` or `webpack/client_only.js`
3. Run the `webpack` command to rebuild your bundles

Adding a components via NPM and then making it accessible to HyperReact as a Ruby class really is that simple.


## Example

To complete this tutorial, let's install a JavaScript React component and use it in HyperReact.

We are going to use [Pete Cook’s React rplayr](https://github.com/CookPete/rplayr) to play a YouTube video.

First let’s install the component via NPM:

```
npm install react-player --save
```

Next we need to `require` it in `webpack/client_and_server.js`

```javascript
// webpack/client_and_server.js
ReactPlayer = require('react-player')
```

Then run `webpack` so it can be bundled

```
webpack
```

And then finally let’s use it while rendering a HyperReact component:

```ruby
def render
  div do
    ReactPlayer(url:  'https://www.youtube.com/embed/FzCsDVfPQqk',
      playing: true
    )
  end
end
```

Please see the [Hyperloop site](http://ruby-hyperloop.io/tutorials/) for further tutorials






