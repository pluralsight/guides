Welcome! Please check the [Live Demo](https://library-app.firebaseapp.com) page and play with the app what we are going to build together.

* Live demo: [library-app.firebaseapp.com](https://library-app.firebaseapp.com/)

The tutorial is based on and inspired by [@zoltan](https://github.com/zoltan-nz) with the original repo sitting at [https://github.com/zoltan-nz/library-app](https://github.com/zoltan-nz/library-app) and powers the [yoember.com](http://yoember.com/) 

# Contents

* [Lesson 1 - Creating our first static page with Ember.js](#lesson-1)
  * [Add Bootstrap and Sass to Ember.js App](#ember-bootstrap-sass)
* [Lesson 2 - Computed property, actions, dynamic content](#lesson-2)
* [Lesson 3 - Models, saving data in database](#lesson-3)
* [Lesson 4 - Deploy your app and add more CRUD functionality](#lesson-4)
* [Lesson 5 - Data down actions up, using components](#lesson-5)
* [Lesson 6 - Advance model structures and data relationships](#lesson-6)

# Prerequisites

- node.js: Currently, Ember CLI works fine with Node (0.12) and npm (2.x) so if you already have any of this move to the next step otherwise install the latest 5.9.0.

To check for the versions run 
`node --version`
`npm --version`

To install use the following guide
  [The best way to install Node.js on Mac, Linux and on Windows](http://yoember.com/nodejs/the-best-way-to-install-node-js/)

- Ember Inspector Extension

   - Chrome extension: [Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en)

   - Firefox lovers: [firefox extension](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/)

- (Optional) Watchman from Facebook: On Mac and Linux, you can improve file watching performance by installing [watchman](https://facebook.github.io/watchman/docs/install.html)


# Lesson 1

This tutorial uses Ember CLI tool (v2.4.2).

## Install Ember CLI 

The following `npm` command installs Ember CLI latest stable version in the global namespace. The latest Ember CLI version 2.4.2 is released on 1 of March 2016, it generates app with Ember.js v2.4 and Ember Data v2.4. (If you have an earlier version of Ember CLI, the following command automatically updates it to the latest.)

    $ npm install -g ember-cli
    
or if you would like to install with fixed version

    $ npm install -g ember-cli@2.4.2

You have now a new `ember` command in your console. Check with

    $ ember -v
    
You should see something similar:

```
version: 2.4.2 
node: 5.9.0 
os: linux x64 
```

(Node version, npm version and OS version may be different in your configuration.)

Please read more about Ember CLI here: [www.ember-cli.com](http://www.ember-cli.com)

## Create the app

In your terminal, navigate in the folder where you usually create your web applications.
For example, if you have a `projects` folder use 

```
$ cd ~/projects
```

Inside this folder run the following command.

    $ ember new library-app

This command will create the new app for you.

Please change to your new app directory with: 

```
$ cd library-app
```

Open this folder in your favourite code editor and look around. You will see a few files and folders. Ember CLI is scaffolded for you everything what need to run and create an amazing web application. 

## Launch the app

Your skeleton app is ready and you can run it with the following command. Type in your terminal:

    $ ember server

Open your new empty app in your browser: <a href="http://localhost:4200" target="_blank">http://localhost:4200</a>

You should see a "Welcome to Ember" message on your website. Well Done! You have your first Ember.js application. :)

You can open Ember Inspector in your Browser. Hope you've already installed it. Ember Inspector exists in Chrome and in Firefox as an extension. After installation you should have a new tab in your developer console in your browser. Check it out, look around. [More details about Ember Inspector here](https://guides.emberjs.com/v2.4.0/ember-inspector/installation/). 

## Add Bootstrap and Sass to Ember.js App

Let's add some basic style to our application. We use Bootstrap with Sass. Ember CLI can install for us add-ons and useful packages. These add-ons simplify our development process, because we don't have to reinvent the wheel, we get more out of the box. You can find various packages, add-ons on these websites: http://www.emberaddons.com or http://www.emberobserver.com

We install an add-on for Sass and another for Bootstrap.

Exit your `ember server` with `Ctrl+C` in your terminal.

Run the following two commands:

```
$ ember install ember-cli-sass
$ ember install ember-cli-bootstrap-sassy
```

This adds some code to your `package.json` and `bower.json` files.

I love using sass because of the flexibility it gives. Lets use it in our app. We start by renaming our `app.css` to `app.scss` with the following terminal command or you can use your editor to rename the `./app/styles/app.css` file:

```
$ mv app/styles/app.css app/styles/app.scss
```

Open `./app/styles/app.scss` file in your editor and add the following line:

```
@import "bootstrap";
```

Adding bootstrap will not take effect immediately until you relaunch your app with `ember server`. You should see in the browser, that 'Welcome to Ember' uses Bootstrap default font.
