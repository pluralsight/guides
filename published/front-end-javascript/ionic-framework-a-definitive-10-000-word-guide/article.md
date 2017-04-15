Welcome!  This definitive guide to the ionic framework -- an incredibly easy way to build beautiful and interactive mobile apps using HTML5 and AngularJS.  We will help you understand the major aspects of the framework.  We'll also be creating a running example of a calculator application throughout the tutorial.

In this first section, you'll learn:

+ An overview of the choices for making applications
+ Introduction to the Ionic framework
+ How to install Ionic on both Mac and Windows
+ How to use Ionic CLI to start an Ionic project
+ How to run an Ionic application

## Introduction
> 85% of mobile time is spent in apps

You're bombarded with reports [all](http://www.theguardian.com/technology/appsblog/2014/apr/02/apps-more-popular-than-the-mobile-web-data-shows) [over](http://www.smartinsights.com/mobile-marketing/mobile-marketing-analytics/mobile-marketing-statistics/) [the](http://venturebeat.com/2013/04/03/the-mobile-war-is-over-and-the-app-has-won-80-of-mobile-time-spent-in-apps/) [web](http://techcrunch.com/2015/06/22/consumers-spend-85-of-time-on-smartphones-in-apps-but-only-5-apps-see-heavy-use/) that users tend to spend way more time on their phones and especially in apps (rather than surfing the web using their phones) and you decided that **it's time to learn how to make an app**.

Up until fairly recently, if you wanted to make an app for (currently) two most popular mobile operating systems (iOS and Android) your only bet was to make the, so-called, **native application**. This meant you needed to make two versions of your application. Therefore, developers were opting either for iOS or Android, whereas big firms had two developing departments, one for each platform.

Nowadays, with the Ionic Framework you can create one application by using the skills you, as a web developer, already have and then deploy this **one codebase** as an app to both iOS and Android stores.

## Ways you can make an app these days
There are actually 3 ways that you can make an application for mobile devices these days.

### Native app
You can make an app specifically for iOS and Android by using their specific SDKs. If you want to build a native application for iOS you have to:

+ have a Mac computer
+ download [Xcode](https://developer.apple.com/xcode/) from the App Store
+ buy the [Apple Developer license](https://developer.apple.com/programs/) that costs 99$ per year

If you want to build a native application for Android you have to:

+ download the appropriate SDKs
+ buy the [Google Developer license](https://play.google.com/apps/publish/signup/) which is 25$ [one time registration fee](https://support.google.com/googleplay/android-developer/answer/6112435?hl=en)

One of the pros of a native applications would be it's **speed** and direct access to a **native API**. A definite con of a native applications is that you need to build two (or more) applications, one for each desired platform.

### Mobile website
Mobile website is actually a _"normal"_ website that you visit with your browser on your phone, designed specifically to adapt to your phone's screen. 

Developers used to make specific sites just for mobile browsers (on it's own domain; usually m.domain.com) but this proved to be hard to maintain. A practice called **[responsive website design](https://developers.google.com/web/fundamentals/layouts/rwd-fundamentals/)** is used these days, where you basically have one HTML codebase, and you determine the look for specific devices (based on resolutions) by using the so-called **[media queries](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Media_queries)** in CSS.

A definite advantage of the mobile websites is that you can update them without waiting for the approval from Apple or Google. One of the disadvantages are that the mobile websites these days have way lower engagement than they used to, and that you can't use any of the additional phone features.

### Hybrid app
A hybrid app is a mobile application, written with the same languages that you use when building websites, with the addition that it contains an isolated browser instance, called **WebView**, which runs this web application inside of a native app. Hybrid apps can access the mobile device and use the additional phone features like for example camera or GPS.

Definite advantage of the hybrid apps is that you can access the additional phone features via plugins and that you can do all the development with the same set of skills you use when developing _"normal"_ web applications. One of the disadvantages is that, even though it's improving, the so-called Web View has it's limitations in terms of speed. Basically, you wouldn't use a hybrid approach if you want to build a 3D game.

## 1. What is Ionic and why it's so good
As I gave an [answer on StackOverflow](http://stackoverflow.com/questions/31179211/use-ionic-or-cordova/31180666):

> Disclaimer: This will sound like advertisement, so I have to say I'm in no way affiliated with Ionic, I just happen to like it so much that I'm sharing the love for it.
> 
> Ionic is so much more than “just” an UI framework. Ionic allows you to:
> 
> + generate icons and splash screens for all devices and device sizes with a single command: `ionic resources`. This alone saves you at least a day of image preparing for various sizes.
> + instantly update your apps with code changes, even when running directly on your device with `ionic run --livereload`
> + build and test iOS and Android versions side-by-side and see changes instantly with `ionic serve --lab`
> + share your Ionic apps with clients, customers, and testers all around the world without ever going through the App Store with `ionic share`
> + easily access the full native functionality of the device using ngCordova (here you get to use any Cordova plugin)
> Also, [Drifty](http://drifty.com/) (the team behind the Ionic framework) is building a full-stack backend services and tools for your Ionic app like [Deploy](Deploy) (for deploying a new version without going through Apple review process! - **this is huge!**), [Analytics](http://www.nikola-breznjak.com/blog/ionic/ionic-analytics-alpha-lets-you/), [Push notifications](http://blog.ionic.io/announcing-ionic-push-alpha/).
> Ionic CLI (command line interface) uses Cordova in the backend and allows you to build (directly using Ionic CLI) apps for iOS and Android (just by doing `ionic build ios` or `ionic build android`).
> Ionic uses Angular as a frontend framework so if you’re familiar with it, it will come as a bonus. They’re [working closely with the Angular 2.0 team](http://blog.ionic.io/angular-2-ionic/) too.
> All in all, I personally think Ionic framework has a bright future, so if nothing else – give it a try I bet you’ll like the ease of making an app with it.

## Installing prerequisites for both Mac and Windows
We need to have [Node.js](https://nodejs.org/en/) and [Git](https://git-scm.com/) installed in order to install both Ionic and Cordova. If you're a web developer chances are that you already have these tools installed. If not, just visit the aforementioned websites and install them.

## Installing Ionic
To install Ionic (both on Mac and Windows) you have to run the following command:

`npm install ionic cordova -g`

We had to install Cordova because Ionic uses it _under the hood_. We're using the **-g** flag to install the packages [globally](https://docs.npmjs.com/getting-started/installing-npm-packages-globally).

## Using Ionic CLI
If you run `ionic` in your Terminal you will get a nice summary of all the commands that you can run using the ionic CLI, along with their short descriptions.

## Starting a project with Ionic by using the existing templates
Ionic CLI allows us to start and initialize your project by using the `ionic start` command. If you just run `ionic start appname` the Ionic CLI will create a bootstrap application with all the needed parts in the **appname** folder, with the so-called **blank** template.

In our example, we will use the **sidemenu** template, so execute the following command:

`ionic start Ionic_1stTutorial sidemenu`

## Running the Ionic application
First, change the directory to the name of the application you gave in the `ionic start` command (**Ionic_1stTutorial** in our case).
There are few ways in which you can get your Ionic application running:

+ `ionic serve` - starts the app in a local web browser
+ `ionic emulate android` - starts the app in an emulator (in this example android is used; you can also use **ios** if you're on a Mac and have all the prerequisites installed)
+ `ionic run android` - starts the app on the actual device that's plugged into your computer
+ `ionic build android` - creates an **.apk** file which you can copy to your Android device and run it (this scenario doesn't work for iOS devices; you have to go through Xcode)

Run the following command in your Terminal:

`ionic serve`

You should get your local browser started up automatically, pointing to the address **http://localhost:8100/#/app/playlists**, with an output similar to the one on the image below:

![](http://i.imgur.com/Yvs4kzKl.jpg)

Great thing about this is that you automatically have a **live reload** feature, which means that as soon as you change the code in your **www** folder, the application will reload automatically so that you don't have to do it manually.

> If you like, you can get this project on [Github](https://github.com/Hitman666/Ionic_1stTutorial).

## 2. How to create a simple calculator application with Ionic framework by using Ionic Creator for UI

In this section you'll learn:

+ How to create a calculator interface mockup
+ How to create a calculator interface prototype without coding by using Ionic Creator
+ How to start the Ionic application based on the created interface
+ Which are the most important folders and files and what is the starting point of the application
+ How to create the calculator logic by using controllers

Finished project:

+ clone the code from [Github](https://github.com/Hitman666/Ionic_2ndTutorial)
+ in the project directory execute `ionic serve` to run the project locally in your browser
+ you can check out the [live example](http://nikola-dev.com/IonicCalculator/mobile.html) of this application

## Calculator interface mockup
Before starting any application we should have an idea of:

+ what we want
+ what problem are we trying to solve with our application 
+ how are we going to solve that problem
+ how our application will look like

The problem is that we can't do calculations fast enough in our mind (except if you're [Arthur Benjamin](https://www.youtube.com/watch?v=e4PTvXtz4GM)) and we need a tool to help us with that. Sure, there are specific calculator devices, but it would be too cumbersome to carry one with us all the time.

Besides, since these days *almost everyone* has a smartphone, it turns out it would be an awesome idea to make an app for it. Because, as they say:

> there's an app for that

where **that** is basically everything these days.

At this point we don't need any fancy options, we'll just stick with the basic mathematical operations like adding, substracting, dividing and multiplying.

These basic operations will be our MVP (Minimal Viable Product), as the author Eric Ries explains in his awesome (and highly recommended) book [Lean Startup](http://www.nikola-breznjak.com/blog/books/the-lean-startup-eric-ries/). We can always add features later, if it turns out that our idea was good. Or, we can pivot away from it, if it turns out it was not a _next best thing_, by not spending too much time and money building the app with dozen of features which in end would not be used at all.

You can use various tools that help with mocking up your application's user interface ([Balsamiq Mockups](https://balsamiq.com/products/mockups/), [Mockingbird](https://gomockingbird.com/home), [Mockup Builder](http://mockupbuilder.com/), etc.), but I tend to be old school about it and I made a little hand drawing of how I imagined the app should look like, and this is what I came up with after few attempts:

![](http://imgur.com/znFbgb1.jpg)

## Calculator interface prototype
We're going to use [Ionic Creator](https://creator.ionic.io/), which is a great tool that helps in quick user interface prototyping. The best thing is that you can just download the created HTML and use it directly in your Ionic application.

### Creating calculator interface with Ionic Creator
To use [Ionic Creator](https://creator.ionic.io/) you have to signup/login on their website. Once you do that, you'll be asked to name your application (I chose *IonicCalculator*):

![](http://i.imgur.com/7yx5g5F.jpg)

The main screen of the Ionic Creator application looks like this:

![](http://i.imgur.com/SB3E1QY.jpg)

In the upper left hand side you'll see the **PAGES** panel and inside it you'll see **Page** item. Click on **Page**, and on the right hand side change the **TITLE** to *Calculator*. Next, remove the **PADDING** by switching off the slider. The way this should look like now is shown on the image below:

![](http://i.imgur.com/T2uGCrr.jpg)

First, we need some kind of a "display" which will show the numbers that we're clicking. Ideally, we would use a `<label>` component, however it's not available in the Ionic Creator as of yet. So, for this we will use the `<input>` element, and we can easily adjust this later when we'll download the generated HTML code.

First you need to drag the **Form** component on the page, and then on it you should drag&drop the **Input** component as well.

Select the Input component and change its PLACEHOLDER to **0** (zero digit) and NAME to **display**. The way this should look like now is shown below:

![](http://i.imgur.com/4r10hfI.jpg)

Next, according to our mockup, we should add buttons that would represent the digits from **0** to **9**, and the buttons that would represent mathematical operations (+, -, *, /). Also, we will need the equals button (**=**) and a clear button (**C**).

In the first row we need to have 4 buttons, the ones representing digits 7, 8, 9 and one representing a dividing operation. To accomplish this, we will use a **Button Bar** component from the Components panel so that we'll drag&drop it just below the Input field on the screen. You should see something like this now in your Ionic Creator window:

![](http://i.imgur.com/CVcreWF.jpg)

As you can see from the previous image, three buttons were added inside the Button Bar component. Click on a button with text **1** and change it to **7**. Repeat the process for other two buttons by changing the text to 8 and 9, respectively.

Ionic Creator adds 3 buttons within the Button Bar, but we need the 4th one. So, to add it, drag&drop a **Button** component next to the button which you labeled as 9. Click on this newly added button and change its text to **/**, and its style to **Energized**. Your interface should look like the one on the image below:

![](http://i.imgur.com/OPr1wpc.jpg)

Repeat the process by adding next three rows:

+ buttons representing 4, 5, 6 and a button representing multiplication (**x**)
+ buttons representing 1, 2, 3 and a button representing subtraction (**-**)

Your interface should look like the one on the image below:

![](http://i.imgur.com/xdzhTdg.jpg)

In the last row you should also add four buttons:

+ one button representing the Clear operation (**C**), with the `Assertive` style
+ one button representing 0
+ one button representing the equals operation (**=**), with the `Positive` style
+ one button representing the addition operation (**+**), with the `Energized` style

Your interface should look like this:

![](http://i.imgur.com/Ci5bPkV.png.jpg)

With this our interface is ready and now we can export it by clicking on the Export icon in the header, as shown below:

![](http://i.imgur.com/BLBjCpu.jpg)

After this appear s a popup:

![](http://i.imgur.com/BsQTesK.jpg)

## Starting a project with Ionic CLI by using the template made in Ionic Creator
From the previous image we see that we have three options (Ionic CLI, ZIP file, and Raw HTML) to start our application based on the mockup we created in Ionic Creator. We're going to use the Ionic CLI option, by running the following command from our Terminal:

`ionic start Ionic_2ndTutorial creator:167ff268a01d`

*Note: if you followed this tutorial by creating the interface in Ionic Creator yourself, then you will have a different creator number. Of course, you're free to use mine if you wish to do so. Also, you're free to name your project any way you want, I chose __Ionic_2ndTutorial__ in this example.*

Once the command is project is created, `cd` into it:

`cd Ionic_2ndTutorial`

Execute `ionic serve` to run the application, and you should see something like:

![](http://i.imgur.com/Kp49zVS.png)

## Ionic application folder structure
Ionic's folder structure looks like this:

![](http://i.imgur.com/WAqDPs3.jpg)

### hooks folder
**hooks** folder contains code for *Cordova hooks*, which are used to execute some code during the Cordova build process. For example, Ionic uses Cordova's **after_prepare** hook to inject platform specific (iOS, Android) CSS and HTML code.

### plugins folder
**plugins** folder contains Cordova plugins which are added to the project. You can add [any available Cordova plugin](http://plugins.cordova.io/#/) to your project, and you can even create your own if you're technical enough. As you can see from the image above we currently have 5 installed plugins.

### scss folder
**scss** folder contains the **ionic.app.scss** file which is based on [SASS](http://sass-lang.com/). This file contains default variables like for example theme colors. By default, you don't have to use SASS, but I encourage you to do so and learn more about it [here](http://sass-lang.com/guide).

### www folder
**www** folder is the most important since it contains all the files our application is made of, and we will be spending most of our development time in this folder.

### bower.json and .bowerrc files
**bower.json** is a configuration file for [Bower](http://bower.io/).

Bower is a package manager for front-end modules that are usually comprised of JavaScript and/or CSS. It lets us easily search for, install, update, or remove these front-end dependencies. If you want to learn more about Bower, you can check out my post [How to Manage Front-End JavaScript and CSS Dependencies with Bower](http://www.nikola-breznjak.com/blog/codeproject/how-to-manage-front-end-javascript-and-css-dependencies-with-bower-on-ubuntu-14-04/).

Bower stores the downloaded front-end modules in a folder which is defined in a **.bowerrc** file, and is set to **www/lib** by default.

### config.xml file
**config.xml** is a configuration file for the [Cordova](https://cordova.apache.org/) project. It contains some meta information about the app like permissions and a list of Cordova plugins which are used in the app. To learn more about available settings in the `config.xml` file, please refer to the [official documentation](https://cordova.apache.org/docs/en/4.0.0/config_ref_index.md.html).

### gulpfile.js file
**gulpfile.js** is a configuration file for [Gulp](http://gulpjs.com/).

Gulp is a JavaScript task runner that helps with rapid development by making use of its multiple plugins for different tasks like concatenation, minifying, automatic unit test running, LESS/SASS to CSS compilation, copying modified files to different directories, etc...

*Note: you don't need to learn Gulp in order to use Ionic. But, if you want to add some additional features to your build process (like minifying your scripts), you can check out the official [getting started documentation](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md).*

### ionic.project
**ionic.project** is a configuration file for Ionic, used to store meta information about Ionic project and the associated Ionic.io cloud account.

### package.json
**package.json** is a file used by `npm` to store versions of the `npm` packages installed in the current project.

`npm` (Node.js Package Manager) is a CLI tool which comes with Node.js installation and it's used for installing other tools like aforementioned Bower, Gulp, etc...

### .gitignore and README.md file
Both .gitignore and README.md are files related to [GitHub](https://github.com/). I'm sure most of the readers know and use GitHub (and consequently [Git](https://git-scm.com/)).

**.gitignore** is a configuration file for GitHub. Contents of the `.gitignore` file, shown below, basically instructs Git that it should not track folders **node_modules**, **platforms** and **plugins** when uploading the code to GitHub.
```
node_modules/
platforms/
plugins/
```

**README.md** is a [Markdown](http://daringfireball.net/projects/markdown/) file used on GitHub to explain the purpose and usage of your project.

## Refactoring our application
We'll take a look at two most important files (**index.html** and **app.js**) and explain their contents step by step.

Also, we'll refactor our code into separate files, since Ionic Creator put everything in one. Finally, we'll add the calculator logic so that our calculator will work as expected.

> Generally, putting all the code in one file and mixing logic and presentation (JavaScript and HTML code) is simply a big NO-NO, and often referred to as [spaghetti code](https://en.wikipedia.org/wiki/Spaghetti_code). You can learn more about refactoring your code from the cult book [Refactoring: Improving the Design of Existing Code](http://amzn.to/1LWo4Ow) by Martin Fowler, Kent Beck, and others. If you're searching for a bit lighter introduction to refactoring and good programming practices in general I can't recommend the classic [Code Complete 2](http://amzn.to/1LWomoq) by Steve McConnell enough.

### index.html file
The starting point of our Ionic application is `index.html`. Now we'll go step by step through this file by explaining the certain lines of code. Also, we'll remove the unnecessary pieces, and move some other parts of the code to different files.

The `<meta>` element is used to properly resize our application on mobile devices.

Next, you can safely remove the `<style>` element completely out of the file.

Since we're not going to use SASS in this example, we can safely remove the following `<link>` tag:

`<link href="css/ionic.app.css" rel="stylesheet">`

Next, as you can see, there is nowhere a reference to the `js/app.js` file inside the `index.html` file, thus meaning it's not used anywhere. If by any chance you've looked around the file structure of the application we created in the first section you will most likely find this strange. Indeed, Ionic Creator needs some additional tweaking, but I'm sure this will all be dealt with in the future.

Since the code inside the `<script>` element is JavaScript (AngularJS to be more exact) and it's mixed with HTML, we don't want to have that so we're going to copy the contents of the `<script>` element and paste it (by overwriting its whole contents) to the file `app.js` (resides in the `js` folder).

So, our `js/app.js` file should now look like this:

```javascript
angular.module('app', ['ionic'])
.run(function($ionicPlatform) {
  $ionicPlatform.ready(function() {
    // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
    // for form inputs)
    if(window.cordova && window.cordova.plugins.Keyboard) {
      cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
    }
    if(window.StatusBar) {
      // org.apache.cordova.statusbar required
      StatusBar.styleDefault();
    }
  });
})

.config(function($stateProvider, $urlRouterProvider) {
    // Ionic uses AngularUI Router which uses the concept of states
    // Learn more here: https://github.com/angular-ui/ui-router
    // Set up the various states which the app can be in.
    // Each state's controller can be found in controllers.js
    $stateProvider
    .state('page1', {
        url: '',
        templateUrl: 'page1.html'
    });

    // if none of the above states are matched, use this as the fallback
    $urlRouterProvider.otherwise('');
});
```

Since we removed the contents of the `<script>` element and pasted it in the `js/app.js` file, we need to add a reference to it from the `index.html` file, and we can do this by placing the following `<script>` element just before the ending `</head>` element:

`<script src="js/app.js"></script>`

The next important thing is the **ng-app** directive which is added as an attribute on the `<body>` tag. This is just a way to let AngularJS know how to [bootstrap the application](https://docs.angularjs.org/guide/bootstrap). We have to use the same name in `app.js` file when we define the root angular module. As you can see from the `app.js` file:

`angular.module('app', ['ionic'])`

and from the `index.html` file:

`<body ng-app="app" animation="slide-left-right-ios7">`

in our example this name is simply **app**.

There's just one more thing we're going to do in the `index.html` file. See the another `script` tag inside our body tag:

`<script id="page1.html" type="text/ng-template">`

Well, this also looks like a good piece to take out and put into its own file. Again, if you saw the file structure from the application in the first section you might remember that the default place to put the template files is inside the `templates` folder.

Since we don't have that folder in our example, create it inside the `www` folder. Then, create the `calculator.html` file and paste the contents of the previously mentioned `<script>` tag. This is how our `templates/calculator.html` file should look like now:

```html
<ion-view title="Calculator">
    <ion-content padding="false" class="has-header">
        <form>
            <label class="item item-input" name="display">
                <span class="input-label">Input</span>
                <input type="text" placeholder="0">
            </label>
        </form>
        <div class="button-bar">
            <button class="button button-stable button-block ">7</button>
            <button class="button button-stable button-block ">8</button>
            <button class="button button-stable button-block ">9</button>
            <button class="button button-energized button-block ">/</button>
        </div>
        <div class="button-bar">
            <button class="button button-stable button-block ">4</button>
            <button class="button button-stable button-block ">5</button>
            <button class="button button-stable button-block ">6</button>
            <button class="button button-energized button-block ">x</button>
        </div>
        <div class="button-bar">
            <button class="button button-stable button-block ">1</button>
            <button class="button button-stable button-block ">2</button>
            <button class="button button-stable button-block ">3</button>
            <button class="button button-energized button-block ">-</button>
        </div>
        <div class="button-bar">
            <button class="button button-assertive button-block ">C</button>
            <button class="button button-stable button-block ">0</button>
            <button class="button button-positive button-block ">=</button>
            <button class="button button-energized button-block ">+</button>
        </div>
    </ion-content>
</ion-view>
```

Just for reference, here's the contents of the `index.html` file, as it should look like now:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title></title>

    <link href="lib/ionic/css/ionic.css" rel="stylesheet">
    <script src="lib/ionic/js/ionic.bundle.js"></script>
    <link href="http://code.ionicframework.com/ionicons/1.5.2/css/ionicons.min.css" rel="stylesheet">

    <script src="js/app.js"></script>
</head>

<body ng-app="app" animation="slide-left-right-ios7">
    <div>
        <div>
            <ion-nav-bar class="bar-stable">
                <ion-nav-back-button class="button-icon icon ion-ios7-arrow-back">Back</ion-nav-back-button>
            </ion-nav-bar>
            <ion-nav-view></ion-nav-view>
        </div>
    </div>
</body>
</html>
```

### app.js file
Contents of the **app.js** file bootstraps our Ionic (well, more accurately AngularJS) application, configures necessary plugins, and defines the states in our application. As we said before, the name of the root module matches the value given to the `ng-app` directive in the `index.html` file (**app** in our case). Also, we need to *inject* '`ionic`' module dependency to our root module so that we can use the Ionic library directives and other Ionic components.

`$ionicPlatform.ready` event is triggered after the device (your mobile phone on which the application is started) is all set up and ready to use. This includes plugins which are used with this project. If you would try to check if some plugin is available outside the `ready` function callback, you could get wrong results due to the fact that it is possible that some plugin would not have been set up just yet. Using the aforementioned `$ionicPlatform.ready` event and placing your code to check for plugin instantiations solves these issues. You can learn more about ionicPlatform utility methods here: [http://ionicframework.com/docs/api/utility/ionic.Platform/](http://ionicframework.com/docs/api/utility/ionic.Platform/).

Inside the `$ionicPlatform.ready` function's callback, we're detecting if a Keyboard plugin is available in our Ionic application. If so, we're hiding the keyboard accessory bar (buttons like `next`, `previous` and `done`). You can change these settings, and quite a bit of additional ones (see a [full list here](https://github.com/driftyco/ionic-plugin-keyboard)), as per your project's requirements.

Inside the `config` function callback we're setting the routes for our application. Ionic uses [AngularUI Router](https://github.com/angular-ui/ui-router) which uses the concept of states. Since we moved our calculator UI into the `calculator.html` file inside the `templates` folder, we have to adjust the path to it accordingly. Also, we'll put our calculator logic inside the so called `CalculatorCtrl` controller. The state definition, with the accompanying controller definition should now look like this:

```javascript
$stateProvider
    .state('calculator', {
        url: '',
        templateUrl: 'templates/calculator.html',
        controller: 'CalculatorCtrl'
    });
```

## Calculator logic with controllers
We will put the code of our `CalculatorCtrl` inside a new file. So, create a file called `CalculatorCtrl.js` inside the `js` folder. In the `index.html` put the reference to this new controller file by adding:

`<script src="js/CalculatorCtrl.js"></script>`

just after the `app.js` inclusion script tag.

There are really multiple ways that we could have realized the calculator logic, but we'll keep it simple here. The contents of our `CalculatorCtrl.js` file will be as follows:

```javascript
angular.module('calculator', [])
.controller('CalculatorCtrl', function($scope){
    $scope.result = '';

    $scope.btnClicked = function(btn){      
        if (btn == 'C'){
            $scope.result = '';
        }
        else if (btn == '='){
            $scope.result = eval($scope.result);
        }
        else{
            $scope.result += btn;   
        }       
    };
});
```

We're defining a new AngularJS module called `calculator`, and we're defining a controller called `CalculatorCtrl`. In the `CalculatorCtrl` we're injecting AngularJS's $scope variable so that we can access the DOM in the `calculator.html` file.

Before explaining this rather simple code, we must not forget to inject the `calculator` module to our root app module in the `app.js` file, so that it will now look like this:

`angular.module('app', ['ionic', 'calculator'])`

Ok, so, back to our controller code; first, we're defining a variable `result` on the $scope, so that we can show it in our template file (`calculator.html`).

Next, we defined a new function called `btnClicked` which accepts one parameter named `btn`. Inside the function we're checking for the passed parameter and we have three cases based on the value of the parameter that is passed:

+ if the parameter equals `C` in that case we're clearing the result
+ if the parameter equals `=` in that case we calculate the equation and show the result
+ in any other case we're just appending the buttons value to the current equation

## Finishing the calculator template
The only thing which is left for us to do is to add the appropriate function calls on the buttons in the `calculator.html` template. Basically, all that we have to do is add the `ng-click="btnClicked()` function call to each button, and pass in the appropriate parameter. The way this should look like is as follows:

```html
<ion-view title="Calculator">
    <ion-content padding="false" class="has-header">
        <form>
            <label class="item item-input" name="display">
                <span class="input-label">Result</span>
                <input type="text" placeholder="0" ng-model="result">
            </label>
        </form>
        <div class="button-bar">
            <button class="button button-stable button-block" ng-click="btnClicked('7')">7</button>
            <button class="button button-stable button-block" ng-click="btnClicked('8')">8</button>
            <button class="button button-stable button-block" ng-click="btnClicked('9')">9</button>
            <button class="button button-energized button-block" ng-click="btnClicked('/')">/</button>
        </div>
        <div class="button-bar">
            <button class="button button-stable button-block" ng-click="btnClicked('4')">4</button>
            <button class="button button-stable button-block" ng-click="btnClicked('5')">5</button>
            <button class="button button-stable button-block" ng-click="btnClicked('6')">6</button>
            <button class="button button-energized button-block" ng-click="btnClicked('*')">x</button>
        </div>
        <div class="button-bar">
            <button class="button button-stable button-block" ng-click="btnClicked('1')">1</button>
            <button class="button button-stable button-block" ng-click="btnClicked('2')">2</button>
            <button class="button button-stable button-block" ng-click="btnClicked('3')">3</button>
            <button class="button button-energized button-block" ng-click="btnClicked('-')">-</button>
        </div>
        <div class="button-bar">
            <button class="button button-assertive button-block" ng-click="btnClicked('C')">C</button>
            <button class="button button-stable button-block" ng-click="btnClicked('0')">0</button>
            <button class="button button-positive button-block" ng-click="btnClicked('=')">=</button>
            <button class="button button-energized button-block" ng-click="btnClicked('+')">+</button>
        </div>
    </ion-content>
</ion-view>
```

As you can see from the code, we're passing to the function an argument which basically represents the button which was clicked.

Also, worth noting is that we used the

`ng-model="result"`

on the `input` element, so that we can access it from the controller through the AngularJS `$scope` object.

## Run the application
Now you can run the application by executing the command

`ionic serve`

and you should see your awesome calculator open up in your browser.

As mentioned in the beginning of this section, you can take a look at the source code on [GitHub](https://github.com/Hitman666/Ionic_2ndTutorial), or you can take a look at the [live example](http://nikola-dev.com/IonicCalculator/mobile.html) of this application.

## Conclusion
In this section we showed you how to create a calculator application step by step. We showed how to create a mockup of our idea, then we showed how to create an interface by using Ionic Creator, and finally how to refactor our application and create the logic with controllers.

# 3. How to polish, and test our calculator application


This third section explains:

+ How to polish our existing calculator application
+ How to create icons and splash screen images automatically in Ionic framework
+ How to implement Google AdMob ads
+ How to use Ionic.io cloud service to share our application with other users without going through the app stores
+ How to test our application on the real physical devices and emulators

## How to polish our existing calculator application
At this point we have a working simple calculator application that lacks some sanitization checks and design improvements which we'll fix in this section.

### Sanitization check
In our current application, we don't have a security measure against the possible malformed input. For example, one could enter two plus (+) signs one after another, which would consequently produce an error.

To handle this, we're going to wrap our evaluation line of code in the try/catch block and we're going to show an alert to the user using the [$ionicPopup](http://ionicframework.com/docs/api/service/$ionicPopup/) service (injected as a dependency in the `CalculatorCtrl` controller). The whole contents of the `CalculatorCtrl.js` file should look like this now:

```javascript
angular.module('calculator', [])
.controller('CalculatorCtrl', function($scope, $ionicPopup){
  $scope.result = '';

  $scope.btnClicked = function(btn){    
    if (btn == 'C'){
      $scope.result = '';
    }
    else if (btn == '='){
      if ($scope.result == ''){
        return;
      }

      try {
        $scope.result = eval($scope.result).toFixed(0);
      }
      catch (err) {
        $ionicPopup.alert({
            title: 'Malformed input',
            template: "Ooops, please try again..."
          });

          $scope.result = '';
      }
    }
    else{
      $scope.result += btn; 
    }   
  };
});
```

You may have noticed that we added the [toFixed](http://www.w3schools.com/jsref/jsref_tofixed.asp) after our `eval` function call, to show no decimal places in case of a division operation. At this point you certainly realize the "problem" of our SuperSimple Calculator - it has no way of entering the point (.), but I hope that by now you learned enough to add this option yourself, if you would need it.

If we intentionally make an error while inputting the formula we should get the following popup:

![](http://i.imgur.com/RNQ2yOS.jpg)

Clearly, there are multiple ways you could approach this problem further in order to create a better user experience. One idea is to check on every button click if the current formula would execute correctly with `eval` and if not then immediately inform the user, or completely ignore the last clicked button. I encourage you to create your own way of the "improved [UX](https://en.wikipedia.org/wiki/User_experience)" and share it in the comments.

### Design changes
If we're being honest, our app currently doesn't look quite *representative*. So, we're going to change that in order to make it a bit nicer. Most of the changes that we're going to make will be in the CSS files. However, if you talk with any web designer these days they will tell you that using purse CSS is, well, outdated.

Ionic has support for [SASS](http://sass-lang.com/). So, to take advantage of the variables, nesting, mixins and all other great stuff that SASS provides, we're going to prepare our project to use SASS by entering the following command:

`ionic setup sass`

Along with some other npm install related output, you should see something like this:

```
Successful sass build
Ionic setup complete
```

Our main starting point in changing how our application looks like is the `scss/ionic.app.scss` file. It's contents is short, and even though you may not have used SASS before, I'm sure it will look logical to you.

When, for example, you want to change the color you only need to change it in one place in one variable, without having to trace it through all the files and change the certain color in all the places that it is used. 

On the image below you can see what I came up with after changing few of the CSS rules (and a slight addition to our HTML template that I'll address additionally):

![](http://i.imgur.com/QijEnmp.png)

Now we're going to go through the parts that were changed so that you can follow along in your example and see for yourself. But, before you start doing the changes you'll need to add the following link code to your `index.html` file to load the CSS file generated through SASS:

`<link href="css/ionic.app.css" rel="stylesheet">`

As for the changes, here is the final content of the `templates/calculator.html` file:

```html
<ion-view title="SuperSimple Calculator">
    <ion-content padding="false" class="has-header" scroll="false">
        <section class = "home-container">
            <div class="myInputRow">
                <form>
                    <input type="text" placeholder="0" ng-model="result" class="myResultInput" />
                </form>
            </div>
        
            <div class="button-bar row">
                <button class="button button-stable button-block" ng-click="btnClicked('7')">7</button>
                <button class="button button-stable button-block" ng-click="btnClicked('8')">8</button>
                <button class="button button-stable button-block" ng-click="btnClicked('9')">9</button>
                <button class="button button-energized button-block" ng-click="btnClicked('/')">/</button>
            </div>
            <div class="button-bar row">
                <button class="button button-stable button-block" ng-click="btnClicked('4')">4</button>
                <button class="button button-stable button-block" ng-click="btnClicked('5')">5</button>
                <button class="button button-stable button-block" ng-click="btnClicked('6')">6</button>
                <button class="button button-energized button-block" ng-click="btnClicked('*')">x</button>
            </div>
            <div class="button-bar row">
                <button class="button button-stable button-block" ng-click="btnClicked('1')">1</button>
                <button class="button button-stable button-block" ng-click="btnClicked('2')">2</button>
                <button class="button button-stable button-block" ng-click="btnClicked('3')">3</button>
                <button class="button button-energized button-block" ng-click="btnClicked('-')">-</button>
            </div>
            <div class="button-bar row">
                <button class="button button-assertive button-block" ng-click="btnClicked('C')">C</button>
                <button class="button button-stable button-block" ng-click="btnClicked('0')">0</button>
                <button class="button button-positive button-block" ng-click="btnClicked('=')">=</button>
                <button class="button button-energized button-block" ng-click="btnClicked('+')">+</button>
            </div>
        </section>
    </ion-content>
</ion-view>
```

The changes were as follows:

+ I first removed the `<span class="input-label">Result</span>` and edited the `form` element to just contain the following line of code: 

`<form><input type="text" placeholder="0" ng-model="result" class="myResultInput"></form>`

+ Notice the added class `myResultInput` on the `input` element
+ Also, notice the `scroll="false"` on the `ion-content` tag, which disables content scrolling
+ I changed the title to SuperSimple Calculator from just Calculator on the `ion-view` tag
+ I wrapped this `form` element in a `div` element with the class `myInputRow`
+ On the every `div` element with the class `button-bar` I added the class `row`
+ Finally, I wrapped this whole content in the `div` element with class `home-container`

In the `scss/ionic.app.scss` file I added the definitions for these new classes:

```css
.myResultInput {
    font-size: 36px !important;
    text-align: right !important;
    width: 100%;
    line-height: 36px !important;
    height: 54px !important;
    padding: 10px !important;
}

.home-container {
    display: -webkit-flex;
    -webkit-flex-direction: column;
    height: 100%;
}

.home-container > .row {
    -webkit-flex: 2;
    padding: 0px !important;
}

.home-container > .myInputRow{
  -webkit-flex: 1 !important;
}
```

Flexbox is used here as a basis in making the layout which fills the whole content horizontally. You can learn more about it from [this Ionic specific tutorial](http://www.joshmorony.com/how-to-create-complex-layouts-in-ionic/), or you can learn more about Flexbox in general from [this tutorial](https://css-tricks.com/snippets/css/a-guide-to-flexbox/).

Next, in the `lib/scss/_variables.scss` file I changed the values of the following variables:

```css
$button-block-margin: 0px !default;
$button-font-size: 32px !default;
$button-height: 70px !default;
```

Finally, in the `lib/scss/_button-bar.scss` file I changed the value of the following rule inside the `button-bar .button` rule:

`border-width: 1px;`

This way we now have an interface which fills the whole available content.

## How to create icons and splash screen images automatically in Ionic framework
The icon is an important part of your application because it represents your application's brand, and it helps to quickly identify where the app is on your phone.

Ionic helps tremendously by providing a single Ionic CLI command to generate all the needed icon and splash screen sizes for us automatically (which usually used to be a tedious work). Also, Ionic created [Photoshop Icon Template](http://code.ionicframework.com/resources/icon.psd) and [Photoshop Splash Screen Template](http://code.ionicframework.com/resources/splash.psd), which you can download for free and use as a guideline for creating an icon.

When you're creating a branded product, having a custom made icon is definitely a must. However, in this case, we'll show how to use one of the free services to search for a free icon which we can use in our application. I tend to use [IconFinder](https://www.iconfinder.com) a lot and these are the filters I used:

![](http://i.imgur.com/TDkaEbp.jpg)

Download and open up the [Photoshop Icon Template](http://code.ionicframework.com/resources/icon.psd) and drag the calculator icon in Photoshop. Don't worry if you don't have Photoshop, you can use open source tools like for example [Gimp](http://www.gimp.org/) or [Paint.NET](http://www.getpaint.net/index.html).

Remove the `icon-cut-guide` layer and save the file to Desktop as `icon.png` with `File->Save for Web`. Repeat the process to create the splash screen image and name it `splash.png`. The image I came up with looks like this in Photoshop:

![](http://i.imgur.com/3hPXcza.jpg)

Now copy **icon.png** and **splash.png** imagesto the **resources** folder in the root of the application (next to the folders `www` and `plugins`). Create the folder if you don't have it yet.

Next, run the following command in the application's root folder:

`ionic resources`

From the output you will see how much icons were created and hopefully now you will see how much time this saved. All the needed configuration regarding the icons and splash screens was generated by Ionic and placed in the **config.xml** file.

It's worth noting that you will only see the icon and splash screen once you deploy the application to the actual physical device or the emulator.

## How to implement Google AdMob ads
There are multiple ways you can earn money with your app these days and here are just a few:

+ Paid app - ask for a flat out price for your app on the App/Play Store
+ Freemium - give the app for free but charge for in-app purchases like adding some extra features (like more gold or faster production in game apps)
+ Ad-based - show ads inside your application. Potentially offer the in-app purchase to remove the adds

What we're going to cover here is the Ad-based monetization option where we'll show how to add Google AdMob ads to our calculator application. There are two parts to implementing Google AdMob ads to our Ionic project: AdMob settings and Ionic settings.

> I made a [short video tutorial](https://www.youtube.com/watch?v=uvHW7Cb29lo) about how to add AdMob to your Ionic project, so you can check that out too.

### AdMob settings

1) Sign in/Sign up for AdMob at [https://www.google.com/admob/](https://www.google.com/admob/)

2) Click the **Monetize new app **button:

[![Screen Shot 2015-05-05 at 23.21.06](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.21.06.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.21.06.png)

3) If your app is not yet published you can add it manually:

[![Screen Shot 2015-05-05 at 23.23.07](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.23.07.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.23.07.png)

4) Create new tracking ID:

[![Screen Shot 2015-05-05 at 23.25.20](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.25.20.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.25.20.png)

5) Configure the adds type, size, placement, style, name:

[![Screen Shot 2015-05-05 at 23.26.29](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.26.29.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.26.29.png)

6) You can read additional info on how to implement GA and AdMob, but for now let's just click Done:

[![Screen Shot 2015-05-05 at 23.28.10](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.28.10.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.28.10.png)

7) You will now see the following similar screen:

[![Screen Shot 2015-05-05 at 23.30.11](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.30.11.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.30.11.png)
The most important thing to note here is this **Ad unit ID**, which in my test case is **ca-app-pub-7957971173858308/3599533362**. Please make a note of this string as it's the most important part of this setting.

8) Create as much of the Ad units as you may need (for each platform[iOS, Android] and ad format [Banner, Interstitial]). In my case, I just created the additional Interstitial ad and will use them in both iOS and Android devices.

### Ionic settings
In the root of the application execute the following command to add the [cordova-plugin-admobpro](https://github.com/floatinghotpot/cordova-admob-pro) plugin:

`ionic plugin add cordova-plugin-admobpro`

> If you get an error like: `Error: 404 Not Found: cordova-plugin-admobpro` then please check that you indeed have the latest version of cordova installed. You can update to the newest version by executing `npm install -g cordova`

Now, add the following code to your app.js file, inside the `.run` function so that the run function would now look like:

```javascript
.run(function($ionicPlatform) {
    $ionicPlatform.ready(function() {
        // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
        // for form inputs)
        if (window.cordova && window.cordova.plugins.Keyboard) {
            cordova.plugins.Keyboard.hideKeyboardAccessoryBar(true);
        }

        var admobid = {
            banner: 'ca-app-pub-7957971173858308/9721191760',
            interstitial: 'ca-app-pub-7957971173858308/3674658165'
        };

        if (AdMob) {
            AdMob.createBanner({
                adId: admobid.banner,
                position: AdMob.AD_POSITION.BOTTOM_CENTER,
                autoShow: true
            });
        }
    })
})
```

The mentioned documentation states that it is strongly recommend to use the Interstitial ad type, because it brings more than [10 times more profit](https://github.com/floatinghotpot/cordova-admob-pro#tips) than the banner Ad.

Of course, you can choose to use both also. There's probably no exact formula here, but there are some best practices, and this is what Google has to say about it:

>Interstitial ads work best in apps with natural transition points. The conclusion of a task within an app, like sharing an image or completing a game level, creates such a point. Because the user is expecting a break in the action, it's easy to present an interstitial without disrupting their experience. Make sure you consider at which points in your app's workflow you'll display interstitials, and how the user is likely to respond.

**The solution for which I personally opted for in the end** was to show the banner ads all the time and to show the Interstitial ad every 5th time when the equals (=) button has been pressed. Additionally I wait for 1 second after the 5th button click has been reached before showing the Interstitial ad. Here's how my CalculatorCtrl.js file looks like now:

```javascript
angular.module('calculator', [])
.controller('CalculatorCtrl', function($scope, $ionicPopup){
    $scope.result = '';

    $scope.counter = 1;
    $scope.btnClicked = function(btn){      
        if (btn == 'C'){
            $scope.result = '';
        }
        else if (btn == '='){
            if ($scope.result == ''){
                return;
            }

            try {
                $scope.result = eval($scope.result).toFixed(0);
            }
            catch (err) {
                $ionicPopup.alert({
                    title: 'Malformed input',
                    template: "Ooops, please try again..."
                });

                $scope.result = '';     
            }

            //on every 5th result show the Interstitial ad one second after the result appears
            if ($scope.counter++ == 5){
                setTimeout(function(){
                    if (AdMob){
                        AdMob.showInterstitial();   
                    }
                    
                    $scope.counter = 1;
                }, 1000);
            }
        }
        else{
            $scope.result += btn;   
        }
    };
});
```

## How to use Ionic.io cloud service to share our application with other users without going through the app store
The ability to immediately get feedback on something that you're working on, without having to install the app manually on your clients devices is indispensable for constant feedback loop which is crucial in rapid application development. *This is something that Eric Ries stresses a lot in his book [The Lean Startup](http://www.nikola-breznjak.com/blog/books/the-lean-startup-eric-ries/)*.

In Ionic this is actually easier than you may have thought. All you have to do is create the account on [https://apps.ionic.io](https://apps.ionic.io) and after that in the project root directory just execute the following command:

`ionic upload`

This will prompt you to login and after that it will upload the app to the Ionic.io cloud service and you will be able to view the app through [Ionic View](http://view.ionic.io/), which you can download for free on your smartphone from the App Store/Play Store.

To share your application with a client or testers, without first having to go through the App Store/Play Store, just type:

`ionic share EMAIL`

where, of course, you change EMAIL with an actual email address. Your client/friend will get an email which will guide him on how to install the application on his mobile phone.

## How to test our application on the real physical devices and emulators
If you run the application (to which you added AdMob) in the browser, you will see an error in the Console output (if you're using [Chrome](https://www.google.com/chrome/) or [Firebug](http://getfirebug.com/)) that AdMob is not defined, as shown on the image below:

![](http://i.imgur.com/8hgXqoM.png)

Also, if you view the app in Ionic View you will notice that no ads will show up. This is because **Cordova plugins don't work in the browser, nor in the Ionic View**. Instead, they have to be tested on the real device or in an emulator (simulator in iOS terminology).

In this section, we're going to show how to install the needed prerequisites and how to run our application in an emulator and on the actual physical device for both iOS and Android.

Before we go any further, we have to add the `cordova.js` file reference just before the `js/app.js` script tag in the `index.html` file:

`<script src="cordova.js"></script>`

Just for reference, the whole contents of my `index.html` file is now:

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title>SuperSimple Calculator</title>

    <link href="css/ionic.app.css" rel="stylesheet">

    <script src="lib/ionic/js/ionic.bundle.js"></script>
    <link href="http://code.ionicframework.com/ionicons/1.5.2/css/ionicons.min.css" rel="stylesheet">

    <script src="cordova.js"></script>

    <script src="js/app.js"></script>
    <script src="js/CalculatorCtrl.js"></script>
</head>

<body ng-app="app" animation="slide-left-right-ios7">
    <div>
        <div>
            <ion-nav-bar class="bar-stable">
                <ion-nav-back-button class="button-icon icon ion-ios7-arrow-back">Back</ion-nav-back-button>
            </ion-nav-bar>
            <ion-nav-view></ion-nav-view>
        </div>
    </div>
</body>
</html>
```

### iOS settings
The main tool for developing native iOS applications is [Xcode](https://developer.apple.com/xcode/), which is praiseworthily free for download and it comes with a lot of simulators in which you can see how your app would look like on a real device.

However, if you want to build and deploy iOS applications to the App Store, you need to have a [Mac computer](http://www.apple.com/mac/) since Xcode only works on their operating system. Yes, **even if you're using Ionic, you need to use Xcode** to build the application for iOS.

#### Installing the needed tools
First you need to install Xcode, and you can do that through the App Store application on your Mac.

![](http://i.imgur.com/yS2WuBx.png)

After the installation is finished, open up Xcode, and navigate to `Xcode -> Preferences -> Downloads` tab. Here you have to download the latest version of the Simulator available. At the time of this writing, the version is iOS 8.4 Simulator:

![](http://i.imgur.com/iah66tM.png)

Also, you need to install the ios-sim package with npm:

`npm install -g ios-sim`

#### Running our application in a simulator
To test the application in a simulator, execute the following command:

`ionic emulate ios`

This will open the application in Xcode simulator with a splash screen and ads:

![](http://i.imgur.com/uiU4uOs.png)

If you exit the simulator back to the home screen (by pressing `COMMAND + SHIFT + H`) yo'll see the application's icon:

![](http://i.imgur.com/5KrFOmO.png)

If you run the simulator with the extra `–l` flag you'll . If we also want to see the console.log output, as we would see it in the browser, then we have to add the flag `–c`:

```
ionic emulate ios -lc
```

Since we need to have an Apple Developer account in order to run the application on our physical phone device, we will cover this in the next section where we'll also show how to deploy the application to the actual App Store.

### Android settings
You can develop Android applications on Mac, Linux, and Windows computers. Android provides a number of tools for developers that are available for free from their [website](http://developer.android.com/).

#### Installing the needed tools
Download and install the Android SDK Tools from [official website](http://developer.android.com/sdk/index.html).

The installation is a simple _NextNextNext_ type of installation on Windows machine (if you downloaded the .exe version), and if you downloaded the zip file (on both Mac or Windows) then you need to extract them to a certain folder (`C:\Dev` or `/Users/nikola/Dev` for example) and set up the PATH variable correctly.

On Mac the PATH variable is set like this:

`export PATH=$PATH:/Users/nikola/Dev/android-sdk/tools/`

As for Windows, the command to set the PATH variable is:

`set PATH=%PATH%;C:\Dev\android-sdk\tools`

You can check if everything is set up correctly by running the following command:

`android --help`

Finally, you need to set up the SDK packages, so run the following command:

`android sdk`

The SDK Manager allows you to download the platform files for any version
of Android. I recommend that you download just the most recent packages:

![](http://i.imgur.com/0bTd3N0.png)

The options that you need to check (and then click on `Install`) are:

+ Android SDK Tools
+ Android SDK Platform-tools
+ Android SDK Build-tools (choose the newest one)
+ Android 6.0 (API 23) - here you can actually go with 5.1

Android SDK has an emulator that can emulate the screen size and resolution of most Android devices, but to be honest it's way too slow and clunky and I recommend that you do not use it. Instead, download [Genymotion](https://www.genymotion.com/#!/) which is free for personal use.

Since Ionic sees the Genymotion device as a real device attached to your computer, you can't run `ionic emulate android`, as we did in the iOS section. Instead, you need to use the Ionic's `run` command (before you do the build phase) like this:

`ionic build android && ionic run android`

This would now build and run the application on the Genymotion emulator:

![](http://i.imgur.com/I2cHeH4.jpg)

To run the application on your own Android device, then first build the application:

`ionic build android`

The output of the command will be something like:

```
BUILD SUCCESSFUL

Total time: 5.536 secs
Built the following apk(s):
    C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\platforms\android\build\outputs\apk\android-debug.apk
```

> To test your application on your Android device, you have to [enable the developer settings](http://stackoverflow.com/questions/18103117/how-to-enable-usb-debugging-in-android) (steps may vary depending on your Android version).

Now connect your device and execute the following command:

`adb devices`

You should see a list of devices that are attached to your computer (Genymotion emulators will also show up here), among which the one that you just attached.

To install the application on your attached device first make sure that you're in the root directory of the project and then run the following command from your Terminal: 

`adb –d install platforms\android\build\outputs\apk\android-debug.apk`

Find the application in your application list and open it. You should see the ads in the bottom and after the fifth calculation you should see an Interstitial ad appear.

> If for some reason the adb command wouldn't work, you can just copy the generated .apk file manually to your connected device. After that open this folder on your phone click on the .apk. This way you'll be prompted to install the application on your phone and then you'll be able to find it in the programs list:
> ![](http://i.imgur.com/wyzbnys.jpg)

## Conclusion
In this section, we showed you how to polish our existing calculator application by improving the design and user experience. Next, we showed how to automatically create icons and splash screen images. Then we covered how to make money with our application by using Google AdMob ads. Also, we showed how to share our application with other users without going through the app stores. Finally, we showed how to test our application on the real physical devices and emulators.

# 4. How to publish our calculator application to the Apple's App Store and Google's Play Store

## Google Play Store

In this section we're going to go over the following steps:

+ How to prepare the application for production
+ How to create the keystore file
+ How to build the production ready application
+ How to get the developer license
+ How to sign the APK file
+ How to optimize the APK file
+ How to build the update of your application
+ How to create the application listing
+ How to upload the application to the Play Store

### Preparing our application for production
Basically, you should remove any unnecessary plugins or libraries which you may have installed during development that you didn't end up using. In our example, we can safely remove the `cordova-plugin-console` by executing the following command: `cordova plugins remove cordova-plugin-console`.

Next, in your `package.json` file you should change the `name`, `description` and `version` properties (leave other settings as they are). This is how I've set up mine:

```json
{
  "name": "SuperSimpleCalculator",
  "version": "1.0.0",
  "description": "SuperSimple Calculator by Nikola Brežnjak",
  ...
}
```

In the `config.xml` file make sure that you change the `widget id`, `name`, `description` and `author` elements. My file looks like this now:

```html
<widget id="com.nikola-breznjak.calculator" version="1.0.0" xmlns="http://www.w3.org/ns/widgets" xmlns:cdv="http://cordova.apache.org/ns/1.0" xmlns:gap="http://phonegap.com/ns/1.0">
  <name>SuperSimple Calculator</name>
  <description>
        An Ionic Framework SuperSimple Calculator application.
    </description>
  <author email="nikola.breznjak@gmail.com" href="http://www.nikola-breznjak.com/">
      Nikola Brežnjak
  </author>
```

### Setting up a keystore
A keystore file stores the security key that we'll use later to sign our application. By doing this we can verify that we're the author of the application. You can learn more about application signing from the [official documentation](http://developer.android.com/tools/publishing/app-signing.html).

To create a keystore, execute (change *SuperSimpleCalculator* keyword with your own):

`keytool -genkey -v -keystore SuperSimpleCalculator.keystore -alias SuperSimpleCalculator -keyalg RSA -keysize 2048 -validity 10000`

You will be asked for a password two times (don't forget it!). The process looks like this:

This command will create a `SuperSimpleCalculator.keystore` file. You should place this file in a place where you'll remember it, because you'll need this file for any app updates you'll want to push to the Play Store. However, just to make things clear; **for every new application that you make, you need to create a new keystore file**.

### Building the production ready application 
To create a production version of your application, execute:

`cordova build --release android`

This will generate a new **unsigned** APK file, inside the `platforms/android/ant-build/` folder. The build process should end with an output showing the exact full path to the unsigned, production (release-ready) APK file of your application, like for example:

```
BUILD SUCCESSFUL
Total time: 35.208 secs
Built the following apk(s):
    C:\Users\Nikola\Desktop\IonicTesting\Ionic_4thTutorial\platforms\android\build\outputs\apk\android-release-unsigned.apk
```

### Signing the APK file
Now we'll sign the unsigned version of the APK with a keystore. We can do this with the tool called `jarsigner`. First, copy the keystore to the same directory where your unsigned APK file is. Next, rename the .apk file to something more meaningful, like for example SuperSimpleCalculator-release-unsigned.apk.

In your terminal navigate to this folder and execute the following command:

`jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore SuperSimpleCalculator.keystore SuperSimpleCalculator-release-unsigned.apk SuperSimpleCalculator`

### Optimizing the APK file
The `zipalign` tool can optimize the APK file so that it reduces the amount of space and RAM required by the app on a device.

The zipalign tool just takes the name of the signed file (we still have *unsigned* in the name of our file because we didn't change it in the previous step) and the name of the file to generate. To optimize the APK file execute:

`zipalign -v 4 SuperSimpleCalculator-release-unsigned.apk SuperSimpleCalculator.apk`

The process finishes with the `Verification successful` output and you will get a **SuperSimpleCalculator.apk** file which you can submit to the Google Play Store.

You may get an error like:

> zipalign is not recognized as an internal or external command, operable program or batch file.

In this case, you need to search your computer for the zipalign file, and make sure it's in your system PATH to fix this problem (in my case, on Windows, the path to the file was `C:\Android\android-sdk\build-tools\21.1.2\zipalign.exe`).

### Building the update of your application
To build an update of an existing app you have to follow the same steps, except that you have to use the **same keystore** to sign the application for every update, because the update will be otherwise rejected and you'll have to create a new app listing.

You must update the **version** in the **config.xml** file for the next release, otherwise the app will not be properly updated. To learn more about versioning you can take a look at the [official documentation](https://cordova.apache.org/docs/en/dev/config_ref/index.html), or this [StackOverflow question](http://stackoverflow.com/questions/25858547/phonegap-how-to-set-build-and-version-property-in-config-xml-for-ios).

### Creating the app listing and uploading the app to the Play Store
The first thing you should do is sign up for the Google developer program at [https://play.google.com/apps/publish/signup/](https://play.google.com/apps/publish/signup/). You'll need to pay **$25 one-time fee** for a developer license.

You start by clicking the `Add new application` button after which the popup appears, as show on the image below, and you have to put a `Title` for your application. Here you basically have options to `Upload the APK` and to `Prepare the Store Listing`.

![](http://i.imgur.com/TDtdXX0.jpg)

We're going to upload our APK to production, by simply clicking on the `Upload your first APK to Production` button once on the PRODUCTION tab, then selecting the signed APK that we made in the previous section.

![](http://i.imgur.com/O5WYiL3.jpg)

Once the upload is finished you'll see something like it's shown on the image below, and on the left hand side you'll see the green checkmark next to the APK, meaning that you've completed the APK upload step.

![](http://i.imgur.com/f8Qfl68.jpg)

Next you have to set the graphics for your application. Please note that you have to put at least 2 screenshots!

![](http://i.imgur.com/E440B1O.jpg)

Then we have to select a category:

![](http://i.imgur.com/3gcXBmh.jpg)

Next up is the Content rating where you'll be asked quite a few questions (here you'll be able to better choose the category of your application because it will be listing a few example applications):

![](http://i.imgur.com/62AV13D.jpg)

Next you have to click on the `Save questionnaire`, and after that on the `Get rating` button. After the content rating is done, you'll be able to click the `Apply rating` button.

Finally, you will now have an option to click the `Publish app` button:

![](http://i.imgur.com/KfMoTnb.jpg)

With this your app now goes into the review process which in my case took 4 hours. Until the app is published you will see a `In review` status, and once the app is published you'll see the following:

![](http://i.imgur.com/NaQmZ4g.jpg)

If you click on the [View in play store](https://play.google.com/store/apps/details?id=com.nikola_breznjak.calculator) link you'll see the listing in Play Store:

![](http://i.imgur.com/XKuR9nh.jpg)

You can use this link to share your application on a simple landing page like [this one](http://nikola-breznjak.com/apps/SuperSimpleCalculator/):

![](http://i.imgur.com/FMdbNhX.jpg)

## Apple App Store
First, you need to enroll in [Apple Developer Program](https://developer.apple.com/programs/), and the fee is 99$ paid yearly.

### Connecting Xcode with your developer account
After you receive your developer status (took 2 days in my case), open Xcode on your Mac and go to `Preferences -> Accounts` and add your account to Xcode by clicking the `+` button on the lower left hand side, and follow the instructions:

![](http://i.imgur.com/ByEbzsJ.png)

### Signing
Now that you linked Xcode with your developer account, go to `Preferences -> Accounts`, select your Apple Id on the left hand side and then click the `View Details` button shown on the previous image. You should see the popup similar to the one on the image below:

![](http://i.imgur.com/zTEqbqg.png)

Click the `Create` button next to the `iOS Distribution` option.

You can learn more about maintaining your signing identities and certificates from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingCertificates/MaintainingCertificates.html).

### Setting up the app identifier
Next, through the [Apple Developer Member Center](https://developer.apple.com/membercenter) we'll set up the app ID identifier details. Identifiers are used to allow an app to have access to certain app services like for example Apple Pay. You can login to Apple Developer Member Center with your Apple ID and password.

Once you're logged in you should choose `Certificates, Identifiers, and Profiles` option as shown on the image below:

![](http://i.imgur.com/1aiVMqG.png)

On the next screen, shown on the image below, select the `Identifiers` option under the iOS Apps.

![](http://i.imgur.com/tmRoIlg.png)

One the next screen, shown on the image below, select the plus (+) button in order to add a new iOS App ID.

![](http://i.imgur.com/BbK4KHq.png)

On the next screen, shown partially on the image below, you'll have to set the name of your app, and use the `Explicit App ID` option and set the `Bundle ID` to the value of the `id` in your Cordova `config.xml` `<widget>` tag.

![](http://i.imgur.com/fsENfcP.png)

Additionally, you'll have to choose any of the services that need to be enabled. For example, if you use Apple Pay or Wallet in your app, you need to choose those option. By default you can't deselect `Game Center` and `In-App Purchase` options, but for details on how to do this you can take a look at [this Stackoverflow question](http://stackoverflow.com/questions/15913115/ios-how-can-i-deselect-game-center-and-in-app-purchase-when-i-try-to-register-my).

You can learn more about registering app identifiers from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/MaintainingProfiles/MaintainingProfiles.html).

### Creating the app listing
Apple uses [iTunes Connect](https://itunesconnect.apple.com) to manage app submissions. After you login, you should see a screen similar to the one on the image below:

![](http://i.imgur.com/mvnrafK.png)

Here you have to select the My Apps button, and on the next screen select the + button, just below the `iTunes Connect My Apps` header, as shown on the image below:

![](http://i.imgur.com/68rK26r.png)

This will show three options in a dropdown, and you should select the `New App`. After this the popup appears, as shown on the image below, where you have to choose the name of the application, platform, primary language, bundle ID and SKU. As you can see on the image below I had to name my application differently, because `SuperSimple Calculator` was taken by some other application already published in the store.

![](http://i.imgur.com/fs6EDva.png)

Once you're done, click on the `Create` button and you'll be presented with the following screen where you'll have to set some basic options like Privacy Policy URL, category and sub category.

![](http://i.imgur.com/K0dIeL4.png)

Now, before we fill out everything in the listing, we'll build our app and get it uploaded with Xcode. Then you'll come back to finish the listing.

You can learn more about managing your app in iTunes Connect from the [official documentation](https://developer.apple.com/library/ios/documentation/IDEs/Conceptual/AppDistributionGuide/UsingiTunesConnect/UsingiTunesConnect.html).

### Building the app for production
In the root directory of your application execute the following command:
`cordova build ios --release`

If everything went well you'll see the **BUILD SUCCEEDED** output in the console.

### Opening the project in Xcode
Open the `platforms/ios/SuperSimpleCalculator.xcodeproj` file in Xcode (of course you would change `SuperSimpleCalculator` with your own name).

![](http://i.imgur.com/3lSsexm.png)

Once the Xcode opens up the project, you should see the details about your app in the general view, as shown on the image below:

![](http://i.imgur.com/qHttt0n.png)

You should just check that the bundle identifier is set up correctly, so that it's the same as the value you specified earlier in the app ID. Also, make sure that the version and build numbers are correct. Team option should be set to your Apple developer account. Under the deployment target you can choose which devices your application will support.

Since we're on this setting screen right now, I should mention that it's **very important** that you select the **Requires full screen** option, otherwise you may (as I did) get the following error when uploading the app to the App Store (covered further in the tutorial):

```
Invalid Bundle. iPad Multitasking support requires these orientations: 'UIInterfaceOrientationPortrait,UIInterfaceOrientationPortraitUpsideDown,UIInterfaceOrientationLandscapeLeft,UIInterfaceOrientationLandscapeRight'. Found 'UIInterfaceOrientationPortrait,UIInterfaceOrientationPortraitUpsideDown' in bundle 'com.bitscoffee.PhotoMarks.iOS'.
```

### Creating an archive of the application
In Xcode, select `Product -> Scheme -> Edit Scheme` to open the scheme editor. Next, select the `Archive` from the list on the left hand side. Make sure that the `Build configuration` is set to `Release` as shown on the image below:

![](http://i.imgur.com/WGZUw7t.png)

To create an archive choose a `Generic iOS Device`, or your device if it's connected to your Mac (you can't create an archive if simulator is selected), from the Scheme toolbar menu in the project editor, as shown on the image below:

![](http://i.imgur.com/4y3hs6p.png)

Next, select `Product -> Archive`, and the Archive organizer appears and displays the new archive. At this point Xcode runs some preliminary validations on the archive and it may display validation warnings in the project editor. At this point I got a few errors shown on the image below:

![](http://i.imgur.com/To14nx0.png)

This problem was resolved by the solution posted in this [Stack Overflow question](http://stackoverflow.com/questions/12184767/phonegap-cdvviewcontroller-h-file-not-found-when-archiving-for-ios) where basically the solution was to double click the `<multiple values>` under `Header Search Paths` and change `$(OBJROOT)/UninstalledProducts/include` to `$(OBJROOT)/UninstalledProducts/$(PLATFORM_NAME)/include` as shown on the image below:

![](http://i.imgur.com/7LBPSTN.png)

Another error that I got was about bitcode support. You can learn more about the error and solution from this [StackOverflow question](http://stackoverflow.com/questions/30848208/new-warnings-in-ios9), but basically you have to set the Bitcode support to `No` in the `Build Options`, as shown on the image below:

![](http://i.imgur.com/38v8fhV.png)

After you fix any errors you can repeat the `Product -> Archive` step and when it succeeds it will bring up the window as shown on the image below:

![](http://i.imgur.com/wRbjZbr.png)

Here it's recommended that you click the `Validate` button which will check your app for any additional issues. If everything goes well, you'll see the following notification:

![](http://i.imgur.com/7NRtPF4.png)

At this point you can click the `Upload to App Store...` button, and if everything goes fine you'll have an uploaded app, and the only thing that's left to do is to complete the iTunes Connect listing and submit it for review!

If you get an email from iTunes Connect shortly after you uploaded the archive with the content similar to this:

```
 We have discovered one or more issues with your recent delivery for "Super Simplistic Calculator". Your delivery was successful, but you may wish to correct the following issues in your next delivery:

Missing Push Notification Entitlement - Your app appears to include API used to register with the Apple Push Notification service, but the app signature's entitlements do not include the "aps-environment" entitlement.
```

If you're not using Push Notifications then you don't have to worry, as stated in the [official Ionic framework forum](http://forum.ionicframework.com/t/missing-push-notification-entitlement/5436/20).

### Finishing the app list process
Now you should head back to the [iTunes Connect portal](https://itunesconnect.apple.com), click on the `Pricing and Availability` on the left hand side under `APP STORE INFORMATION` and set the price of your application:

![](http://i.imgur.com/qEDWaZG.png)

Next, click on the `1.0 Prepare for Submission` button. When we uploaded our archive, iTunes Connect automatically determined which device sizes are supported. As you can also see on the image below, you'll need to upload at least one screenshot image for each of the various app sizes that were detected by iTunes Connect.

![](http://i.imgur.com/Rat0FRr.png)

From my personal experience, the easiest way to generate these images is to simulate the app in Simulator and make a screenshot of different screen sizes. However, please note that you have to make sure that your `Window -> Scale` property in the Simulator is set to `100%`, otherwise the image size will not be the one as expected in iTunes Connect.

Next you'll have to insert Description, Keywords, Support URL and Marketing URL (optionally), as shown on the image below:

![](http://i.imgur.com/46YV8Vw.png)

In the `Build` section you have to click on the `+` button and select the build which we've uploaded through Xcode in the previous steps, as shown on the image below:

![](http://i.imgur.com/RmZrrqX.png)

Next you'll have to upload the icon, edit the rating, and set some additional info like copyright and your information. Note that the size of the icon that you'll have to upload here will have to be 1024 by 1024 pixels. Thankfully, you can use the splash.png from the second section. If you're the sole developer then the data in the `App Review Information` should be your own. Finally, as the last option, you can leave the default checked option that once your app is approved that it is automatically released to the App Store.

Now that we're finished with adding all of the details to the app listing, we can press `Save` and then `Submit for Review`. Finally, you'll be presented with the last form that you'll have to fill out: 

![](http://i.imgur.com/BXbPvhf.png)

The only "tricky" question may be the one about Content rights. In our example we can safely select `No`. Since we're using AdMob in our application we have to select `Yes` in the `Advertising Identifier` and check the `Serve advertisements within the app` checkbox. You can learn more about it from this [StackOverflow question](http://stackoverflow.com/questions/31261926/ios-does-your-app-contain-display-or-access-third-party-content-admob).

After you submit your app for review you'll see the status of it in the My Apps as `Waiting for review`, as shown on the image below. Also, shortly after you submit your app for review you'll get a confirmation email from iTunes Connect that your app is in review.

![](http://i.imgur.com/Bm3LbQA.jpg)

Apple prides itself with a manual review process, which basically means it can take several days for your app to be reviewed. You'll be notified of any issues or updates to your app status. From my personal experience it usually takes up to 5 days for the app to be reviewed. 

### Updating the app
Since you'll probably want to update your app at some point you'll first have to update the build and version numbers in the Cordova `config.xml` file and then rebuild the application and open it up from the Xcode and follow the same steps all over again.

Once you submit for the review, you'll have to wait for the review process again. It's pivotal to note that if your changes aren't too big you could use [Ionic Deploy](http://blog.ionic.io/announcing-ionic-deploy-alpha-update-your-app-without-waiting/) to update your application without going through the review process, but more about this in the future tutorials.

## Conclusion
In this final section we showed you how to prepare the application for the Apple's App Store and Google's Play Store, how to create the listings and finally how to publish the application to both stores. 

If you have any questions about this tutorial, please read the following help guides. Of course, I encourage you to ask a question in the comments so that anyone else who may have a similar question or doubt, learns something too.

With this, I leave you and hope you had a great learning experience with the series so far. We have few of the project ideas lined up already for the next posts. However, **I encourage you to share your wishes and ideas** about what kind of an app would you like to see built step by step. Remember, I'm here to help you get the best of Ionic framework so don't hesitate to ask.

# How to get help with Ionic framework
Few of the resources that proved indispensable when I was learning about Ionic:

+ For a quick framework reference, I'm suggesting the [official documentation](http://ionicframework.com/docs/), which is indeed very good.
+ If you're in search for a good book about Ionic, then you don't need to look further than [Ionic in Action: Hybrid Mobile Apps with Ionic and AngularJS](http://amzn.to/1EjwYNJ)
+ Of course, if you haven't, check out my [first]() [three]() [posts]() in this series
+ One of the best resources on the net for programming related questions, about which you've no doubt heard, is [StackOverflow](http://stackoverflow.com/). You can view the specific Ionic tagged questions via [this link](http://stackoverflow.com/questions/tagged/ionic) or even [this one](http://stackoverflow.com/questions/tagged/ionic-framework). However, I urge you to please read their [help page](http://stackoverflow.com/help) before posting questions (especially if you're a new user to StackOverflow) because otherwise you may have a bad experience. If you're a new StackOverflow user, who's in a hurry to get acquainted with it, make sure you at least go through this [2-minute tour](http://stackoverflow.com/tour).
+ To get started with AngluarJS try [these](http://angular.codeschool.com/) [few](https://hackhands.com/finishing-Angular-TODO-application-deploying-production/) [resources](https://docs.angularjs.org/tutorial), not necessarily in that particular order.
+ Let me just kind of repeat myself by saying that if you have **any questions** about Ionic framework, or you had trouble following this tutorial (you couldn't install something), or you would like to suggest what kind of tutorials you would like to see regarding Ionic in the future, then please **share it in the comments** below. Also, you can reach me personally via @[HitmanHR](https://twitter.com/hitmanhr) or [my blog](http://www.nikola-breznjak.com/blog). I'll do my best to answer all of your questions.
+ If, however, you favor "one on one" help, you can reach me [via HackHands](https://hackhands.com/dashboard/request/hitman666/preferred).

