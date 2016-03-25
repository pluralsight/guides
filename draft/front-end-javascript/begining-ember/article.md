## Building a complex web application with Ember.js 2.4

Welcome! Please check the [Live Demo](https://library-app.firebaseapp.com) page and play with the app what we are going to build together.

* Live demo: [library-app.firebaseapp.com](https://library-app.firebaseapp.com/)

The tutorial is based on and inspired by [@zoltan](https://github.com/zoltan-nz) with the original repo sitting at [https://github.com/zoltan-nz/library-app](https://github.com/zoltan-nz/library-app) and powers the [yoember.com](http://yoember.com/) 

## Contents

* [Lesson 1 - Creating our first static page with Ember.js](#lesson-1)
  * [Add Bootstrap and Sass to Ember.js App](#ember-bootstrap-sass)
* [Lesson 2 - Computed property, actions, dynamic content](#lesson-2)
* [Lesson 3 - Models, saving data in database](#lesson-3)
* [Lesson 4 - Deploy your app and add more CRUD functionality](#lesson-4)
* [Lesson 5 - Data down actions up, using components](#lesson-5)
* [Lesson 6 - Advance model structures and data relationships](#lesson-6)

## Prerequisites

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


## Lesson 1

This tutorial uses Ember CLI tool (v2.4.2).

### Install Ember CLI 

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
