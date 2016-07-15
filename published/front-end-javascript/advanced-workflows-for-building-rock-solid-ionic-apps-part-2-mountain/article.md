![Advanced workflows for building rock-solid Ionic apps. Part 2: Mountain](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/blog-02.png)

Welcome to Serious-App-Development Mountain! 

In the second part of our series on developing Ionic apps with [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), you'll be learning about wonderful ingredients like testing, sub-generators, plugins, and ecosystem integration into platforms like the [Ionic Platform](http://ionic.io/). **We are building on the project we created in Part 1 of this series.**

### Cross-platform HTML5 app development is hard
Even with awesome tools like Generator-M-Ionic, Angular JS, Cordova, Ionic (which supports iOS, Android, the mobile web, and Windows (since [Ionic 1.2](http://blog.ionic.io/announcing-ionic-1-2/)), cross-platform HTML5 app development can be a pain.

Here's why: maybe your app should run on phones and tablets with a different, specifically designed layout and workflow for either case. Ah, and what the customer forgot to tell you: obviously it should also run in the browser! *"It's built using web technologies anyways, isn't it?"* your customer asks. Maybe even on desktops using [Electron](http://electron.atom.io/)? Uff ... You'll be optimizing and testing a lot!

And when that's taken care of, there's a lot of complex topics like handling translations, offline data, persistence, and data syncing between devices or the backend of your app which you have to handle to keep customers satisfied. 

These things need to be thoroughly coded while you juggle certificates and licenses when building for the different app stores, trying to integrate complicated Push services or coding custom Cordova plugins and hooks. Worst of all, if customer requirements change, you're back to the drawing board.

I know. We've been through all of this. That's why we built [Generator-M-Ionic](https://github.com/mwaylabs/generator-m-ionic), a trust-worthy and powerful coding companion so that you can focus on the real challenges. And development shouldn't be one of them!

Are you ready to climb Serious-App-Development Mountain? Let's go.

### Quality assurance
You've set up your project, created your first commit, and discovered that you're ready to start coding. Whether you are coding alone or in a team, you'll want to take some measures to ensure that your code is of high quality. Generator-M-Ionic has got you covered here:

#### Linting
Your Generator-M-Ionic project comes with established coding guidelines and workflows already baked in using [ESLint](http://eslint.org/). On every iteration of `gulp watch` Gulp will check all your application JavaScript files for guideline violations.

![eslint in terminal](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/eslint.png)

To get linting notifications as you develop in your editor or to learn how to configure the default set of rules, check out our [ESLint Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/eslint.md). If you are working with JSON files in your `app/` folder (for instance, to handle translations), the generator's linting will validate those too! This keeps your development trouble free.

#### Testing
Another area where you don't have to deal with the hassle of setting up and configuring everything yourself is unit testing (with Karma) and end-to-end testing (with Protractor). Your sample app even comes with a ready-to-use test-suite that you can try out right now by running:
```sh
gulp karma
# and
gulp protractor
```
The relevant files for the test setup are these:
```
test/
  └── karma/
  └── protractor/
karma.conf.js
protractor.conf.js
```
Our [Testing Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/testing.md) can help you get started with writing your own unit and end-to-end tests for your app. Once you have that mastered, the [Husky Hooks Guide](https://github.com/mwaylabs/generator-m-ionic/tree/master/docs/guides/testing_workflow.md) explains how you can run linting and tests automatically before you `git commit` or `git push`.

### Coding

Now for the actual development. You'll probably want to:
- add your own Angular components using our [subgenerators](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/sub_generators.md)
  - controllers, templates, directives, services, filters, constants, ... or even whole modules
- add some [Sass](http://sass-lang.com/) to spice up your app's styling
- add [Cordova Plugins](http://ngcordova.com/docs/plugins/) to use with [ngCordova](http://ngcordova.com/) for that real app-feeling
- add some additional [bower packages](http://bower.io/search/) for special tasks

We will go through each of these tasks briefly to give you a general idea of how things work. For more detailed explanations, visit our [Documentation](https://github.com/mwaylabs/generator-m-ionic/).

#### Adding Angular components
Our array of [subgenerators](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/sub_generators.md) allows you to create the most important Angular components very easily. If applicable they also generate sample test files so you can start testing right away.

For instance, we use the pair-subgenerator a lot. It creates a controller, with a test file and a template with the same name.
```sh
yo m-ionic:pair phone
```
![pair sub-generator](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/sub_generator.png)

Now we only need to add a state to the `main.js`:
```js
.state('main.phone', {
  url: '/phone',
  views: {
    'tab-phone': {
      templateUrl: 'main/templates/phone.html',
      controller: 'PhoneCtrl as ctrl'
    }
  }
})
```
Then add a navigation item in our `tabs.html` file, which you'll find in `app/main/templates/`, to bring it all together:
```html
<!-- List Tab -->
<ion-tab title="Phone" icon-off="ion-ios-telephone-outline"
  icon-on="ion-ios-telephone"
  ui-sref="main.phone">
  <ion-nav-view name="tab-phone"></ion-nav-view>
</ion-tab>
```
That's it. A new navigation item, a new route, controller, test file and template in about two minutes. Here's the result:

<p align="center">
  <img alt="See new navigation item in app" src="https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/nav_phone.png" width="330px">
</p>

#### Adding Sass
This is an even easier task. Every module you generate comes with a default Sass file. For your main module this would be `main.scss` and it's located in `app/main/styles/`. Open it and add some Sass:
```scss
ion-list {
  ion-item {
    color:red;
  }
}
```
Upon saving, your `gulp watch` task will automatically compile and inject the resulting CSS, even reload your browser. Not sassy enough? As your project grows larger, you may want to split your Sass in multiple files. Find out how in our [Sass integration Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/sass_integration.md).

#### Adding plugins
If we don't add some nice [Cordova Plugins](http://ngcordova.com/docs/plugins/) to our app, it won't be a real hybrid app. So let's do it!

Your project comes with a local installation of the latest version of the [Cordova CLI](https://cordova.apache.org/docs/en/latest/guide/cli/index.html), which you can invoke through Gulp. We install it locally, so you don't have to worry about hunting for file, project, and version. The syntax is almost exactly the same as using a global CLI installation. For instance, to install the Cordova camera plugin, run:
```sh
gulp --cordova "plugin add cordova-plugin-camera --save"
```
You want to develop your app for Windows as well? Install the appropriate [Cordova Platform requirements](https://cordova.apache.org/docs/en/latest/guide/platforms/win8/index.html) and type:
```sh
gulp --cordova "platform add windows --save"
```
Don't forget to call `--save` in order to persist new plugins and platforms in the `config.xml`! 

Our Development Introduction has a dedicated part on [using the Cordova CLI wrapper](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/development_intro.md#using-the-cordova-cli).

Now that we installed a new plugin, we want to use it! [ngCordova](http://ngcordova.com/) is declared as a dependency for every module you create using the generator. Thus you can just start using the plugins right away. 

Refer to the main module declaration in your `main.js` to see how it's done. So in my `PhoneCtrl` I'll only have to inject the `$cordovaCamera` [service](http://ngcordova.com/docs/plugins/camera/) in order to access the plugin and that's all:
```js
angular.module('main')
.controller('PhoneCtrl', function ($cordovaCamera) {

  var options = {
    destinationType: Camera.DestinationType.FILE_URI,
    sourceType: Camera.PictureSourceType.CAMERA,
  };

  $cordovaCamera.getPicture(options)
  .then(function(imageURI) {
    console.log(imageURI);
  });
});
```
Adding new plugins has never been so simple!

In some cases you need to extend your ESLint configuration because some plugins expose JavaScript globals that you might want to use (like `Camera` in the example above). Or you might prefer to access your plugins through the `cordova` global because ngCordova is not always up to date with the latest plugin versions or may not support the plugin you want to use. In order to use those globals without ESLint complaining, augment the `globals` section of your `app/.eslintrc`:
```js
//..
"globals": {
  "angular": true,
  "ionic": true,
  "localforage": true,
  // add those
  "cordova": true,
  "Camera": true
},
//..
```

#### Bower packages
And last, but not least, you'll probably want some more [bower packages](http://bower.io/search/) that go with your app. Maybe your app needs to support different languages. For that, we at M-Way Solutions usually use [angular-translate](https://github.com/angular-translate/angular-translate). Install it by running:
```sh
bower install angular-translate --save
```
The `--save` flag will persist that package in the `bower.json`. Sometimes to ensure that your changes take effect, it is necessary to restart `gulp watch`. Then the only thing left to do is to mark the module as a dependency in your `main.js` module declaration:
```js
'use strict';
angular.module('main', [
  'ionic',
  'ngCordova',
  'ui.router',
  // add this one
  'pascalprecht.translate'
])
```
Now you got your translations! Refer to their [documentation](https://angular-translate.github.io/) to learn more.

### Browser or device?
Up until now we have only seen our app in the browser using `gulp watch`. But at least when you are working with plugins you may want to test your app on a device or emulator. If you have your system correctly set up according to the [Cordova Platform Guides](http://cordova.apache.org/docs/en/latest/#develop-for-platforms), this should be easy:

1. Connect your device to your machine
2. Make sure that device and machine are on the same network
3. Start the `livereload` command and keep it running:

```sh
# run on a connected device with livereload
gulp --livereload "run ios"
# run on an emulator with livereload
gulp --livereload "run --emulate android"
```

The best part about using `livereload` is that you can make changes to your code and see the changes immediately on the device (hence the command's being called "live reload"). Just as with `gulp watch`, but on the device! You'll only have to run the command again if you make changes to the Cordova files (`config.xml`, Platforms, Plugins).

Your glorious app is ready to be tested on your device!

![See your app on your device](https://raw.githubusercontent.com/gruppjo/blogs/master/16-06-advanced-workflows-for-building-rock-solid-ionic-apps/img/device.jpg)


If you don't want to rely on your development machine to keep the `livereload` command running, you can run a full build of your app which is then pushed onto your device.
```sh
gulp --cordova "run ios"
# or
gulp --cordova "run android"
```
Two things happen when you run this command:

1. Gulp will build your app using `gulp build`
  - this takes care of all the JavaScript, HTML and CSS and puts it into the `www/` folder
2. Gulp will call Cordova with the supplied attributes and
  - takes the contents of the `www/` folder and builds a Cordova app from it
  - the app is then pushed onto your device

The implicit run of `gulp build`, for which Cordova commands it will run as well as build options like minification, are explained in our [Development Introduction](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/guides/development_intro.md#cordova-build-run-emulate--under-the-hood) in more detail.


#### gulp watch-build
Usually `gulp build` will build your app without any problems and everything should work just as if you run `gulp watch`. However sometimes it will be necessary to debug the web app part of your build. Since this is a little cumbersome to do when the app's already on your device, there's a convenient intermediate step:

```sh
gulp watch-build
```
This is just like `gulp watch` but only for the `www/` folder. It builds your app into that folder and then opens a browser windows showing the built version of your app. This allows you do quickly find out why your app doesn't build properly. Adding the `--no-build` flag enables you to watch the current version in the `www/` folder. Of course this command also works with `--no-open` and all the other flags that work with `gulp watch` as well.


### Ecosystems
I bet you start feeling like a real pro already. And you should! We're getting close to orbit. Although you're more and more becoming a seasoned astronaut, there are some things you just don't want to build yourself every time: push services, user management, or other backend services, for instance. 

Luckily there's a series of choices of platforms to handle those tasks for you: the [Ionic Platform](http://ionic.io/platform) is popular and feature-rich. It can be integrated into your project using the [Ionic CLI](http://ionicframework.com/docs/cli/) as described in the [Ionic Platform Integration Guide](https://github.com/mwaylabs/generator-m-ionic/blob/master/docs/ecosystems/ionic_platform.md). For enterprise customers, we offer our [Relution Enterprise Mobility Management](https://www.relution.io/en/) platform and there are many other options.

The [Ecosystems section of our Guides](https://github.com/mwaylabs/generator-m-ionic#ecosystems) lists some of these platforms with nice set up guides, generated sample code and templates. Most of the work we've done for you already, so focusing on building your app is easier than ever.

### Congratulations!

You have conquered app-development-mountain and learned how to season your app with a lot of interesting spices: sub-generators, Sass, plugins, ecosystems and bower packages. Be proud! Ascend your development skills to out-of-this-world levels in [part 3](http://tutorials.pluralsight.com/front-end-javascript/advanced-workflows-for-building-rock-solid-ionic-apps-part-3-orbit) of this series by owning environments, proxies and build tools, handling app icons, splash screens, continuous integration and build variables. And at the very end, see what the future of Generator-M-Ionic has to offer for you!

### Get in touch
Feedback, ideas, comments regarding this blog post or any of the features discussed here are very welcome in either the comments section below, at our [Generator-M-Ionic's Github repository](https://github.com/mwaylabs/generator-m-ionic) or the [Generator-M-Ionic Gitter Chat](https://gitter.im/mwaylabs/generator-m-ionic).

### Credits
Author: [Jonathan Grupp](https://github.com/gruppjo)  
Headline illustrations: [Christian Kahl](http://www.art-noir.net/)  
Special thanks to Volker Hahn, Mathias Maier & Tim Lancina

> Originally published July 6, 2016 on the [Ionic Blog](http://blog.ionic.io/advanced-workflows-for-building-rock-solid-ionic-apps-part-2/) in a slightly modified version.
