_There has to be a better way to incorporate the JavaScript ecosystem into Rails._

![pic1](assets/pic1.jpg) Have you:

1. Wondered if there’s a better way to utilize modern JavaScript client frameworks the context of an existing Ruby on Rails project?
2. Gotten confused about how to integrate JavaScript libraries and examples that are packaged up into proper “modules”?
3. Discovered the drawbacks of having all applications JavaScript littering the global name-space?
4. Heard about [ES6 (aka Harmony)](http://www.slideshare.net/domenicdenicola/es6-the-awesome-parts), the next version of JavaScript and how the cool kids in Silicon Valley (Facebook, Instagram, Square, etc.) are using ES6 syntax?

How would you like to achieve the following within a Rails project?

1. The ability to prototype a rich UI, seeing changes in JavaScript and CSS/Sass code almost instantly after hitting save, _without the page reloading_.
2. First class citizenship for utilizing the **Node ecosystem**, by specifying dependencies in `package.json`, running `npm install`, and then simply requiring modules in JavaScript files.
3. Seamless integration of Node based JavaScript assets for the Rails Asset Pipeline, thus **not circumventing** the asset pipeline, but co-existing with it and leveraging it.
4. The ability to plug the node client side ecosystem into an existing Rails project seamlessly.
5. Utilization of many JavaScript tools, such as the **React JSX tranpiler** and **ES6 transpiler**.

This article will show you how you can utilize Webpack in your Rails development process to achieve these goals!

## Background

First, I’m going to tell you a brief story of how I came to the realization that _there had to be a better way to incorporate JavaScript ecosystem into Rails_.


## What’s Wrong with Copying and Pasting Tons of JavaScript into /vendor/assets/javascripts?


Ever imagine doing Ruby on Rails projects without [Bundler](http://bundler.io/)? Oh, the horror! Well that’s what copying tidbits of JavaScript into `/vendor/assets/javascripts` is like! It’s actually a bit worse than that, as many of these JavaScript libraries depend on either [AMD (aka require.js)](http://requirejs.org/) or [CommonJs module syntax](http://wiki.commonjs.org/wiki/Modules/1.1)    being available. (For a great explanation of how these module systems work, see [Writing Modular JavaScript With AMD, CommonJS & ES Harmony](http://addyosmani.com/writing-modular-js/).) This would be much more of a problem in the Rails community were it not for the fact that many popular JavaScript libraries are packaged into gems, such as the [jquery-rails gem](https://github.com/rails/jquery-rails). I used to think that worked fine, until I really wanted to start leveraging the JavaScript ecosystem, and started to encounter JavaScript modules that lacked Gems. Specifically, I wanted to start leveraging the many [npm packaged](https://www.npmjs.org/) react components, such as [react-bootstrap](https://github.com/react-bootstrap/react-bootstrap).

I also _needed_ (or wanted) to leverage the JavaScript toolchain, such as the [JSX](http://facebook.github.io/react/jsx-compiler.html) and ES6 transpilers ([es6-transpiler](https://github.com/termi/es6-transpiler) and [es6-module-transpiler](https://github.com/esnext/es6-module-transpiler)).

_Thankfully, this experience has broken me away from the JavaScript baby bottle of gemified JavaScript! You can now become a 1st class JavaScript citizen!_

![pic3](assets/pic3.jpg)


### Motivation: React and ES6


My foray down the Node rabbit hole began with a desire to use the React framework, including its JSX transpiler. In a nutshell, the [React library](http://facebook.github.io/react/) stood out as **unique, innovative, and impressive**.

I could simply think about the client-side UI as a set of components that are recursively composed and which render based on set of data that flows in one direction, from the top level component to each of its children. For further details into why I like React, see my [React on Rails Tutorial](https://hackhands.com/react-rails-tutorial/ "React on Rails Tutorial"). For purposes of this article, you can imagine substituting my example of using React with your favorite rich client JavaScript framework. _I’d be thrilled if somebody would fork my project and create a version using [EmberJs](http://emberjs.com/)_. At first this mission of integrating React seemed easy, as there is a Ruby Gem, the [react-rails gem](https://github.com/reactjs/react-rails), that provided a relatively painless mechanism of integrating react into a Rails project. This is definitely the simplest method. I’ve created a tutorial with a companion github repository, [justin808/react-rails-tutorial](https://github.com/justin808/react-rails-tutorial/commits/react), that walks you through using the `react-rails` gem with the [Rails 4.2 scaffold generator](http://guides.rubyonrails.org/command_line.html#rails-generate). Then I wanted to plug in the [react-bootstrap](https://github.com/react-bootstrap/react-bootstrap) library. With no gem available, I considered manually copy-pasting the source to my `/vendor/assets/javascripts` directory, but that just seemed to _smell_ for the following reasons:

1. JavaScript has a mature system for managing dependencies (packages & modules): [npm](https://www.npmjs.org/) (and [bower](http://bower.io/)).
2. Dependencies often depend on other dependencies, in both the Ruby and JavaScript worlds. Imagine managing Ruby dependencies by hand?
3. JavaScript modules often depend on either CommonJs or RequireJs being available.

(Side note: in terms of Node, a module is a special case of package that JavaScript code can `require()`. For more info, see the [npm faq](https://www.npmjs.org/doc/misc/npm-faq.html) and [Stack Overflow](http://stackoverflow.com/questions/20008442/difference-between-a-module-and-a-package-in-node)) Here’s a good summary of other ways to handle the assets in a Rails app: [Five Ways to Manage Front-End Assets in Rails](http://www.codefellows.org/blog/five-ways-to-manage-front-end-assets-in-rails). I briefly tried those techniques, plus the [browserify-rails](https://github.com/hsume2/browserify-rails) gem. However, they seemed to conflict with the `react-rails` gem, and if I didn’t use that gem, I’d need a way to convert the jsx into js files. This led me to try the webpack module bundler.

## Webpack

What’s [Webpack](http://webpack.github.io/docs/what-is-webpack.html)?

> webpack takes modules with dependencies and generates static assets representing those modules.

Why did I try Webpack? It was recommended to me by [Pete Hunt of the React team](http://2013.jsconf.eu/speakers/pete-hunt-react-rethinking-best-practices.html). Here’s some solid reasons for “why Webpack”:

1. Leverages npm (and optionally bower) for package management.
2. Supports whatever module syntax you prefer.
3. Has loaders (think pipeline), including ES6 and JSX.
4. Its Webpack Dev Server rocks for quick prototypes (Hot Module Replacement) of JS and CSS/Sass code.

I initially tried the [webpack module bundler](http://webpack.github.io/) separate from Rails, as I wanted to see the “hot reloading” of react code in action. You can try this sample code: [react-tutorial-hot](https://github.com/gaearon/react-tutorial-hot). [Hot module Replacement](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) changes the JS code (and possibly the CSS) of the running code without any page refresh. Thus any data in the JS objects sticks around! This is way cooler than [Live Reload](http://livereload.com/), which refreshes the whole browser page. Then I started using these features of Webpack:

1. [es6-loader](https://github.com/shama/es6-loader), which incorporates both of the [es6-transpiler](https://github.com/termi/es6-transpiler) and the [es6-module-transpiler](https://github.com/esnext/es6-module-transpiler). For fun, try out the ES6 syntax with the [ES6 Fiddle](http://www.es6fiddle.net/).
2. [jsx-loader](https://github.com/petehunt/jsx-loader), which handles jsx files using es6.
3. Trivial integration of any additional packages available via **[npm](https://www.npmjs.org/)** and the ability to use whichever module syntax is most convenient.

As Webpack generates a “bundle” that is not necessarily minified, it would seem that this could be incorporated into the Rails asset pipeline, and sure enough, it can be! This is well described in this article: [Setting Up Webpack with Rails](https://medium.com/brigade-engineering/setting-up-webpack-with-rails-c62aea149679) along with this example code to precompile with Webpack: [Webpack In The Rails Asset Pipeline](http://www.tomdooner.com/2014/05/26/webpack.html). With the basic parts in place, I wanted achieve the following:

1. Be able to prototype client side JS using Webpack Dev Server (with hot module replacement), while having this same code readily available in my Rails app. This involves having JavaScript, Sass, and Image files commonly available to both Rails and the Webpack Dev Server.
2. Be able to easily deploy to Heroku. My solution to the problem is shown in this github repo: [justin808/react-webpack-rails-tutorial](https://github.com/justin808/react-webpack-rails-tutorial). This is based on my tutorial using the `react-rails` gem: [Rails 4.2, React, completed tutorial](https://github.com/justin808/react-rails-tutorial). I will now describe this solution in detail.

### Setup

You’ll need to install Node.js following. I’m assuming you already have Ruby and Rails installed.

- Node.js: You can find the [Node.js download file here](http://nodejs.org/download/). Note, some friends of mine recommended the Node.js installer rather than using Brew. I did not try Brew.
- Many articles recommend running the following command, so that you don’t need to run node commands as sudo, thus changing the ownership of your /usr/local directory to yourself.

``` bash
sudo chown -R $USER /usr/local
```
- Your `/package.json` file describes all other other dependencies, and running `npm install` will install everything required.

Once I got this working, it felt like Santa Clause came to my app with the whole Node ecosystem!
![pic2](assets/pic2.jpg)

### Bundler and Node Package Manager

All Rails developers are familiar with gems and [Bundler (bundle)](http://bundler.io/). The equivalent for Javascript are package.json files with [Node Package Manager (npm)](https://www.npmjs.org/) (see discussion in next point on why not [Bower](http://bower.io/)). Both of these package manager systems take care of retreiving dependencies from reputable online sources. Using a `package.json` file is far superior to manually downloading dependencies and copying the `/vendor/assets/` directory!

![bundle-npm](assets/bundle-npm.png)

### Why NPM and not Bower for JS Assets?

The most popular equivalants for JavaScript are [Node Package Manager (npm)](https://www.npmjs.org/) and [Bower](http://bower.io/). For use with webpack, you’ll want to prefer npm, per the reasons in the [documentation](http://webpack.github.io/docs/usage-with-bower.html):

> In many cases modules from npm are better than the same module from bower. Bower mostly contains only concatenated/bundled files which are:
>
>
> * More difficult to handle for webpack
>
> * More difficult to optimize for webpack
>
> * Sometimes only useable without a module system
>
> So prefer to use the CommonJs-style module and let webpack build it.



### Webpack Plus Rails Solution Description


To integrate webpack with Rails, webpack is used in 2 ways:

1. Webpack is used soley within the `/webpack` directory in conjunction with the Webpack Dev Server to provide a rapid tool for prototyping the client side Javascript. The file `webpack.hot.config.js` sets up the JS and CSS assets for the Webpack Dev Server.
2. Webpack watches for changes and generates the `rails-bundle.js` file that bundles all the JavaScript referenced in the `/webpack/assets/javascripts` directory. The file `webpack.rails.config.js` converts the JSX files into JS files throught the JSX and ES6 transpilers.

The following image describes the organization of integrating Webpack with Rails.
![webpack-rails-organization](assets/webpack-rails-organization1.jpg)



| File | Notes and Description |
| ------------------------------
| `/app/assets/javascripts/rails-bundle.js` | Output of `webpack --config webpack.rails.config.js` |
| `/app/assets/javacripts/application.js` | Add `rails-bundle` so webpack output included in sprockets |
| `/app/assets/javascripts` | Do not include any files used by Webpack. Place those files in `/webpack/assets/javascripts` |
| `/app/assets/stylesheets/application.css.scss` | Reference sass files in `/webpack/assets/stylesheets` |
| `/node_modules` | Where npm puts the loaded packages |
| `/webpack` | All webpack files under this directory except for node_modules and package.json |
| `/webpack/assets/images` | `Symlink to /app/assets/images`. Needed so that Webpack Dev Server can see same images referenced by Rails sprockets |
| `/webpack/assets/javascripts` | javascripts are packaged into rails-bundle.js as well as used by the Webpack Dev Server |
| `/webpack/assets/stylesheets` | stylesheets are used by the asset pipeline (referenced directly by `/app/assets/stylesheets/application.css.scss`) as well as used by the Webpack Dev Server |
| `/webpack/index.html` | the default page loaded when testing the Webpack Dev Server |
| `/webpack/scripts` | files used by only the Rails or Webpack Dev Server environments |
| `/webpack/server.js` | server.js is the code to configure the Webpack Dev Server |
| `/webpack/webpack.hot.config.js` | configures the webpack build for the Webpack Dev Server |
| `/webpack/webpack.rails.config.js` | configures web pack to generate the rails-bundle.js file |
| `/.buildpacks` | used to configure multiple node + ruby buildpacks for Heroku |
| `/npm-shrinkwrap.json` and `/package.json` | define the packages loaded by running ‘npm install’ |


### webpack.config

To reiterate, we needed Webpack for the following reasons:

1. To enable the use of JS “modules”, using either the either the [AMD (aka require.js)](http://requirejs.org/) or [CommonJs module syntax](http://wiki.commonjs.org/wiki/Modules/1.1).
2. To convert JSX files (ES6 and JSX syntax) into JS files. Note, you probably don’t want to blindly convert all JS files into ES6, as that may conflict with some imported modules.

This setup with the `webpack.config` file. We need 2 versions of this file for the two different needs, the Webpack Dev Sever and the Asset Pipeline.

![webpack-files](assets/webpack-files.png)


#### Changing the webpack.config


You maybe wondering if you’ll need to edit these webpack config files. Here’s some things you’ll need to pay attention to.

- **module.exports.entry**: The entry points will determine what webpack places in the bundle. While this may seem similar to the manifest file of `/app/assets/javascripts/application.js`, it’s very different in that you _only_ need to specify the **_entry_** points. So if you specify `./assets/javascripts/example` (you don’t need the file suffix) is the entry point, then you do not and should not specify `./assets/javascripts/CommentBox` as an entry point. Once again, dependencies are calculated for Webpack, unlike Rails.

``` js
module.exports = {
 context: __dirname,
 entry: [
   "./assets/javascripts/example"
 ],
```

- **module.exports.externals**: If you want to load jQuery from a CDN or from the Rails gem, you might specify:
``` js
module.exports.externals: {
  jquery: "var jQuery"
},
```
- **module.exports.module.loaders**: This is the place where you can expose jQuery from your Webpack rails-bundle.js so that the rest of the non-module using parts of Rails can use jQuery.
``` js
module.exports.module: {
  loaders: [
    // Next 2 lines expose jQuery and $ to any JavaScript files loaded after rails-bundle.js
    // in the Rails Asset Pipeline. Thus, load this one prior.
    { test: require.resolve("jquery"), loader: "expose?jQuery" },
    { test: require.resolve("jquery"), loader: "expose?$" }
  ]
}
```


That being said, it’s well worth familiarizing yourself with the [documentation for webpack](http://webpack.github.io/docs/). The [gitter room for webpack](https://gitter.im/webpack/webpack) is also helpful.


### Webpack Dev Server and Hot Module Replacement


While waiting for webpack to create the rails-bundle.js file and then reloading the Rails page is not terribly time consuming, there’s **no comparison** to using the [Webpack Dev Server](http://webpack.github.io/docs/webpack-dev-server.html) with [Hot Module Replacement](https://github.com/webpack/docs/wiki/hot-module-replacement-with-webpack) which loads new JavaScript and Sass code without modifying the existing client side data if possible. If you thought Live Reload was cool, you’ll love this feature. To quote the documentation:

> The webpack-dev-server is a little node.js express server, which uses the webpack-dev-middleware to serve a webpack bundle. It also has a little runtime which is connected to the server via socket.io. The server emit information about the compilation
> state to the client, which reacts on that events. It serves static assets from the current directory. If the file isn’t found a empty HTML page is generated whichs references the corresponding javascript file.

In a nutshell, the file `/webpack/server.js` is the http server utilizing the [Webpack Dev Server API](http://webpack.github.io/docs/webpack-dev-server.html):

1. `/webpack/webpack.hot.config.js` configures the webpack assets.
2. Has a couple of json responses.
3. Configures “hot” to be true to enable hot module replacement.


### JavaScripts


Webpack handles the following aspects of the `/webpack/assets/javascripts` directory:

1. Preparing a “bundle” of the JavaScript files needed by either Rails or the Webpack Dev Server. This includes running the files through the jsx and es6 loaders which transpile the jsx and es6 syntax into standard javascripts. Heres’ the configuration that does the loading:
``` ruby
 module.loaders = [{ test: /\.jsx$/, loaders: ["react-hot", "es6", "jsx?harmony"] }]
```
2. Webpack also normalizes whichever module loading syntax you choose (RequireJs, CommonJs, or ES6).

### Sass and images


For the Webpack Dev Server build (not the Rails build that creates `rails-bundle.js`), Sass is loaded via webpack for 2 reasons:

1. Webpack takes care of running the sass compiler.
2. Any changes made to sass or css files are loaded by the hot module loader into the browser.

The file `/webpack/scripts/webpack_only.jsx` contains this:
``` ruby
require("test-stylesheet.css");
require("test-sass-stylesheet.scss");
```

This “requires” stylesheet information just like a “require” of JavaScript. Thus, `/webpack/index.html` does not reference any output from the Sass generation. This file, `webpack_only.jsx` is referenced only in the `webpack.hot.config.js` file as an “entry point”, which means that it gets loaded explicitly in the created bundle file.

**Images** were a bit tricky, as during deployment, you want your images fingerprinted for caching purposes. This is nearly invisible to users of newer versions of the Rails, thanks to the [fingerprinting feature of the Rails asset pipeline](http://guides.rubyonrails.org/asset_pipeline.html#what-is-fingerprinting-and-why-should-i-care-questionmark).

While webpack can also fingerprint images, that’s not needed as we’re not depending on this feature of webpack for our Rails deployments. So we just need the Webpack Dev Server to access the same image files. I.e., we need to be able to use a syntax in the `scss` files to reference images that works for both the Webpack Dev Server as well as the Rails asset pipeline. For example, here’s a snippet of sass code to load the `twitter_64.png` image from the top level of the `/app/assets/images` directory. This needs to work for both the Asset Pipeline as well as the Webpack Dev Server.

``` ruby
.twitter-image {
  background-image: image-url('twitter_64.png');
}
```

The problem of how to get the same images into the stylesheets of both Rails and Express server versions was solved by using a **symlink**, which git will conveniently store.

- `/webpack/assets/images` is a symlink for the `/app/assets/images` directory.
- The `image-url` sass helper takes care of mapping the correct directories for images. The image directory for the webpack server is configured by this line:

``` ruby
module.loaders = [{ test: /.scss$/, loader: “style!css!sass?outputStyle=expanded&imagePath=/assets/images”}]
```

The sass gem for rails handles the mapping for the Asset Pipeline.

- The symlink was necessary, as the Webpack Dev Server could not reference files above the root directory.

This way the images are signed correctly for production builds via the Rails asset pipeline, and the images work fine for the Webpack Dev Server.


### Sourcemaps

When debugging JavaScript using the Rails app, I did not want to have to scroll through a giant
`rails-bundle.js` of all js assets. Sourcemap support in Webpack addressed that issue. At first I tried to use plain sourcemaps (separate file rather than integrated), but that resulted in an off by one error. Furthermore, I had to do [some fancy work to move the created file to the correct spot](https://github.com/justin808/react-webpack-rails-tutorial/blob/3aa3cd112453ce436b942c45bb3b906458532b89/webpack/webpack.rails.config.js) of `/public/assets`. Also note that building the sourcemap file when deploying to Heroku breaks the Heroku build. Both of these cases are handled at the bottom of the file `webpack.rails.config.js`. This is what sourcemaps looks like in Chrome ![React-Sourcemaps](assets/React-Sourcemaps1.jpg)


### Heroku Deployment


There were 2 main things needed to get builds working on Heroku.

1. I needed to configure the `compile_environment` task to create the `rails-bundle.js` via Webpack using the file `/lib/tasks/assets.rake`.
2. Heroku needs both the node and ruby environments. In order to deploy to heroku, you’ll need run this command once to set a custom buildpack:

``` bash
heroku config:add BUILDPACK_URL=https://github.com/ddollar/heroku-buildpack-multi.git
```
This runs the two buildpacks in the `/.buildpacks` file courtesy of the [ddollar/heroku-buildpack-multi](https://github.com/ddollar/heroku-buildpack-multi) buildpack.

### Why node_modules and package.json are not in the webpack directory?

While it would be tidier to put `node_modules` and `package.json` into the `/webpack` directory, the problem is that this would require a custom buildpack for installing the node_modules on Heroku.

### Why Have a Second Assets Directory Under Webpack?

At first, I had Webpack reference the JSX files from the `/app/assets/javascripts directory`. However, I wanted to be able to use a [WebStorm](http://www.jetbrains.com/webstorm/) project just based on the JavaScript code. I’d either have to put the WebStorm project at the root level, thus including all the Ruby directories, or I could use a sym link to the `javascripts` directory. You **NEVER** want run two different JetBrains products simultaneously on the same directory, so that ruled out using WebStorm at the top of my Rails app. The symlink approach seemed to work, but that got confusing especially given I’d sometimes open the JSX files in Emacs. The approach of putting the webpack bundled assets under the `/webpack/assets` directory worked out well for me. It seems natural that Webpack bundles those assets and puts them into the `rails-bundle.js` file in the `/app/assets/javascripts` directory. For the same reasons, I’m keeping style sheets referenced by Webpack under the `/webpack` directory. Note, I’m using Webpack to load stylesheets, as that allows the style sheet changes to be hot loaded into the browser! If you edit any of the files in the `/webpack/assets/stylesheets` directory, you’ll see the browser update with the style changes almost immediately after you hit save. The standard Rails file `/app/assets/stylesheets/application.css.scss` references the file style sheets in `/webpack/assets/stylesheets`.


### How to Add a NPM (JavaScript) module dependency?

This is a bit like modifying your Gemfile with a new gem dependency.

- Modify your `/package.json` file with the appropriate line for the desired package inside the “dependencies” section. You’ll want to specify an exact version, as that’s the recommendation in the Node community. Just google “npm <whatever module>” and you’ll get a link to the npm page for that module where you can see the version. For example, to add `marked` as a dependency, I added this line to `package.json`.

```
"marked": "^0.3.2",
```
- Include the appropriate line to require the module. For example, to include the `marked` library:
``` js
  var marked = require("marked");
```


### How to update Node Dependencies

When you’re ready to take the time to ensure that upgrading your packages will not break your code, you’ll want to take the following steps. Refer to [npm-check-updates](https://www.npmjs.org/package/npm-check-updates) and [npm-shrinkwrap](https://www.npmjs.org/doc/cli/npm-shrinkwrap.html).

``` bash
 cd <top level of your app>
 rm -rf node_modules
 npm install -g npm-check-updates npm-check-updates -u
 npm install npm-shrinkwrap
```


## Rapid Client Development


Congratulations! You’ve gotten through what I believe is the secret sauce for rapid client side JavaScript development. Once you get the setup, per the above steps, the flow goes like this:

1. Run the Webpack Dev Server on port 3000 ` cd webpack && node server.js `
2. Point your browser at [http://0.0.0.0:3000](http://0.0.0.0:3000/).
3. Start another shell and run `foreman start -f Procfile.dev`
4. Point your browser at [http://0.0.0.0:4000](http://0.0.0.0:4000/) and verify you can see the usage of the rails-bundle.js file.
5. Update the `jsx` and `scss` files under `/webpack/assets` and see the browser at port 3000 update when files are saved.
6. Start with static data in the JSX creation, and then move to having the `server.js` file vend JSON to the client.
7. Once that works, have the rails server create the JSON.
8. Deploy to Heroku!
9. Prosper!

## Helpful Links


1. Github repo for this code: [justin808/react-webpack-rails-tutorial](https://github.com/justin808/react-webpack-rails-tutorial)
2. Live version of this code on Heroku: [http://react-webpack-rails-tutorial.herokuapp.com/](http://react-webpack-rails-tutorial.herokuapp.com/)

## Acknowledgments

This work was inspired by a project for my client, [Madrone Inc.](http://madroneco.com/). The founder clearly desired a UI that did not fit into the standard request/response HTML of Rails. If you want to work with me on this project, or other related projects, please [email me](mailto:justin@railsonmaui.com). I’d like to thank the following reviewers: Ed Roman, [@ed_roman](https://twitter.com/ed_roman), Greg Lazarev, [@gylaz](https://twitter.com/gylaz), Geoff Evason, [@gevason](https://twitter.com/gevason), Jose Luis Torres, [@joseluis_torres](https://twitter.com/joseluis_torres), Mike Kazmier, [@Kaztopia](https://twitter.com/Kaztopia), John Lynch, [@johnrlynch](https://twitter.com/johnrlynch), [Jonathan Soeder](https://twitter.com/soederpop), and Ben Ward, [@mauilabs](https://twitter.com/mauilabs).

