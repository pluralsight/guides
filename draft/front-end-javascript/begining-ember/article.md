# Ember.js 2 Tutorial 
## Building a complex web application with Ember.js 2.4
<p class="blog-post-meta">Latest update: <time datetime="2016-03-20" itemprop="datePublished">20 Mar 2016</time> • <span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name"><a href='http://zoltan.nz'>Zoltan</a></span></span></p>

This is an [Ember.js 2 tutorial](http://yoember.com) from the absolute beginner level. End of the course we touch some advance topic as well.

Welcome! Please check the [Live Demo](https://library-app.firebaseapp.com) page and play with the app what we are going to build together.

* Live demo: [library-app.firebaseapp.com](https://library-app.firebaseapp.com/)

You can clone the original repository from GitHub and launch on your desktop any time.

* Original repo: [https://github.com/zoltan-nz/library-app](https://github.com/zoltan-nz/library-app)

[@jkeat](https://github.com/jkeat), I really appreciate the effort you have put into proofreading this [Ember.js tutorial](http://yoember.com). [@sigu](https://github.com/sigu), [@batisteo](https://github.com/batisteo), thanks guys for fixing too. 

## Other tutorials

* Bookstore API (Ruby on Rails): [https://github.com/zoltan-nz/bookstore-api](https://github.com/zoltan-nz/bookstore-api)
* Bookstore Client (Ember.js): [https://github.com/zoltan-nz/bookstore-client](https://github.com/zoltan-nz/bookstore-client)
* Contacts App Client (Ember.js): [https://github.com/zoltan-nz/contacts-app-client](https://github.com/zoltan-nz/contacts-app-client)
* Chat App (Ember.js): [https://github.com/zoltan-nz/chat-app-v2](https://github.com/zoltan-nz/chat-app-v2) 

## Contents

* [Lesson 1 - Creating our first static page with Ember.js](#lesson-1)
  * [Add Bootstrap and Sass to Ember.js App](#ember-bootstrap-sass)
* [Lesson 2 - Computed property, actions, dynamic content](#lesson-2)
* [Lesson 3 - Models, saving data in database](#lesson-3)
* [Lesson 4 - Deploy your app and add more CRUD functionality](#lesson-4)
* [Lesson 5 - Data down actions up, using components](#lesson-5)
* [Lesson 6 - Advance model structures and data relationships](#lesson-6)

## Prerequisites

* node.js: at least 0.12, but the best if you install the latest 5.9.0.

[The best way to install Node.js on Mac, Linux and on Windows]({% post_url 2016-03-20-the-best-way-to-install-node-js %})

* Ember Inspector Chrome Extension

Install Ember Inspector Chrome extension in your Chrome Browser: [Ember Inspector](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en)

* (Optional) Watchman from Facebook

Install Watchman on Mac: `brew install watchman`
More info: https://facebook.github.io/watchman/

## <a name='lesson-1'></a>Lesson 1

This tutorial uses the latest Ember CLI tool (v2.4.2).

### Install Ember CLI

The following `npm` command installs Ember CLI latest stable version in the global namespace. The latest Ember CLI version 2.4.2 is released on 1 of March 2016, it generates app with Ember.js v2.4 and Ember Data v2.4. (If you have an earlier version of Ember CLI, the following command automatically updates it to the latest.)

    $ npm install -g ember-cli
    
or if you would like to install with fixed version

    $ npm install -g ember-cli@2.4.2

You have now a new `ember` command in your console. Check with

    $ ember -v
    
You should see something similar:

``` bash {% raw %}
version: 2.4.2
node: 5.9.0
os: darwin x64{% endraw %}
```

(Node version, npm version and OS version may be different in your configuration.)

Please read more about Ember CLI here: [www.ember-cli.com](http://www.ember-cli.com)
