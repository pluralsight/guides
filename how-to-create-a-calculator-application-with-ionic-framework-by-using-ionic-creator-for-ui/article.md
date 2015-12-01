![](http://i.imgur.com/0Es31Xg.png)

This is the second post in a series of posts which will teach you how to take advantage of your web development knowledge in building hybrid applications for iOS and Android. The first post in this series was all about [How to get started with Ionic framework on Windows and Mac](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows).

This second post explains:

+ How to create a calculator interface mockup
+ How to create a calculator interface prototype without coding by using Ionic Creator
+ How to start the Ionic application based on the created interface
+ Which are the most important folders and files and what is the starting point of the application
+ How to create the calculator logic by using controllers

Finished project:

+ clone the code from [Github](https://github.com/Hitman666/Ionic_2ndTutorial)
+ in the project directory execute `ionic serve` to run the project locally in your browser
+ you can check out the [live example](http://nikola-dev.com/IonicCalculator/mobile.html) of this application

## Introduction
Since there are not so many Calculator applications in the App store (around **500** according to my search shown on the image below), who knows, you just may create your own bestseller in this category, if you add a needed tweak or two.

![](http://i.imgur.com/hW4ZTKW.jpg)

All jokes and hopes aside, this seemed to be a decent "more serious", but still easy to acomplish, tutorial. Let's start this tutorial by creating the interface for our application.

## Calculator interface mockup
Before starting any application, we should know what we want (well, at least in general). We should know what problem are we trying to solve with our application and how are we going to solve it. Also, we would need to have a decent idea of how we would want our application to look like. 

The answer to the first two questions would be rather easy; the problem is that we can't do calculations fast enough in our mind (well, except if you're [Arthur Benjamin](https://www.youtube.com/watch?v=e4PTvXtz4GM)) and we need a tool to help us with that. Surely, there are specific calculator devices, but it would be too cumbersome to carry one with us all the time.

Besides, since these days *almost everyone* has a smartphone, it turns out it would be an awesome idea to make an app for it. Because, as they say:

> there's an app for that

where **that** is basically everything these days. If you find that there isn't an app for some particular problem that you may have, that just may be your lucky break. Who knows, you just may end up having your 5 minutes of fame on the AppStore, until another clone of your app pushes you away.

Anyways, back to our calculator application; we don't need any fancy options (at least not yet, in this 1.0 version), we'll just stick with the basic mathematical operations like adding, substracting, dividing and multiplying.

These basic operations will be our MVP (Minimal Viable Product), as the author Eric Ries explains in his awesome (and highly recommended) book [Lean Startup](http://www.nikola-breznjak.com/blog/books/the-lean-startup-eric-ries/). We can always add features later, if it turns out that our idea was good. Or, we can pivot away from it, if it turns out it was not a _next best thing after Flappy Bird_, by not spending too much time and money building the app with dozen of features which in end would not be used at all.

As for the user interface, you can use various tools that help with mocking up your application's user interface ([Balsamiq Mockups](https://balsamiq.com/products/mockups/), [Mockingbird](https://gomockingbird.com/home), [Mockup Builder](http://mockupbuilder.com/), etc.), but I tend to be old school about it and I made a little hand drawing of how I imagined the app should look like, and this is what I came up with after few attempts:

![](http://imgur.com/znFbgb1.jpg)

*Note: As you can see there's no comma in our mockup, basically because we're just trying to keep it simple here. If you like, and request it in the comments, I can add it in the future tutorials.*

## Calculator interface prototype

Now that we know what our application needs to do, and how it basically has to look like, lets start by creating our interface.

We could create our interface by manually entering HTML, but we're going to show a different approach here - the one which uses [Ionic Creator](https://creator.ionic.io/), which is an awesome web application that lets you drag&amp;drop components that make up the user interface. As such, this is a great tool which helps in quick user interface prototyping. 

The best thing is that you can then just download the created HTML and use it directly in your Ionic application. Of course, some additional changes will be needed later in the HTML and CSS code, but for creating a quick prototype this will be great.

Worth noting is that there are currently on the market lots of other tools for interface prototyping ([InVision](www.invisionapp.com/), [Flinto](https://www.flinto.com/), [Mockup Builder](http://mockupbuilder.com/), etc.), however these tools don't (at least not yet) have a "one-click download and ready for your Ionic" option like Ionic Creator does.

### Creating calculator interface with Ionic Creator
In order to use [Ionic Creator](https://creator.ionic.io/) you have to signup/login on their website. The first time you log in, you'll be asked for additional info like Company or phone, but you can safely skip that. In the initial screen you will be asked to name your application (I chose *IonicCalculator*):
![](http://i.imgur.com/7yx5g5F.jpg)

The main screen of the Ionic Creator application looks like the one shown on the image below:
![](http://i.imgur.com/SB3E1QY.jpg)


In the upper left hand side you'll see the **PAGES** panel and inside it you'll see **Page** item. Click on **Page**, and on the right hand side change the **TITLE** to *Calculator*. Next, remove the **PADDING** by switching off the slider. The way this should look like now is shown on the image below:
![](http://i.imgur.com/T2uGCrr.jpg)

First, we need some kind of a "display" which will show the numbers that we're clicking. Ideally, we would use a `<label>` component, however it's not available in the Ionic Creator as of yet. So, for this we will use the `<input>` element, and we can easily adjust this later when we'll download the generated HTML code.

> Ionic Creator works in a drag&amp;drop kind of way, which basically means that you can drag any component from the **Componenes** pane on the left hand side and drop it on the actual Phone image in the center of the screen.

Since we concluded that we'll use the Input element, we can try to drag&amp;drop it to the Phone, but that won't work. First you need to drag the **Form** component on the page, and then on it you should drag&amp;drop the **Input** component as well.

Select the Input component (by clicking it in the PAGES panel in the upper left corner mentioned before) and change its PLACEHOLDER to **0** (zero digit) and NAME to **display**. The way this should look like now is shown on the image below:
![](http://i.imgur.com/4r10hfI.jpg)

Next, according to our mockup, we should add buttons that would represent the digits from **0** to **9**, and the buttons that would represent adding (**+**), substracting (**-**), multiplying (**x**) and dividing (**/**) operations. Also, we will need the equals button (**=**) and a clear button (**C**).

In the first row we need to have 4 buttons, the ones representing digits 7, 8, 9 and one representing a dividing operation. To acomplish this, we will use a **Button Bar** component from the Components panel so that we'll drag&amp;drop it just below the Input field on the screen. You should see something like this now in your Ionic Creator window:
![](http://i.imgur.com/CVcreWF.jpg)

As you can see from the previous image, three buttons were added inside the Button Bar component. Click on a button with text **1** and change it to **7**. Repeat the process for other two buttons by changing the text to 8 and 9, respectively.

Ionic Creator adds 3 buttons within the Button Bar, but we need the 4th one. So, to add it, drag&amp;drop a **Button** component next to the button which you labeled as 9. Click on this newly added button and change its text to **/**, and its style to **Energized**. Your interface should look like the one on the image below:
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
+ one button representing the addition operation (**+**), with the `Energized style

Your interface should look like the one on the image below:

![](http://i.imgur.com/Ci5bPkV.png.jpg)

Don't worry about the slight missalignement or paddings, we'll take care of this in the next tutorial when we'll be polishing our application and preparing it for the App/Play store.

So, basically this is it, we have our interface ready and now we can export it by clicking on the Export icon in the header, as shown on the image below:
![](http://i.imgur.com/BLBjCpu.jpg)

After this a popup, with the following content, appears:
![](http://i.imgur.com/BsQTesK.jpg)


## Starting a project with Ionic CLI by using the template made in Ionic Creator
From the previous image we see that we have three options (Ionic CLI, ZIP file, and Raw HTML) to start our application based on the mockup we created in Ionic Creator. We're going to use the Ionic CLI option, by running the following command from our Terminal/Command prompt:

`ionic start Ionic_2ndTutorial creator:167ff268a01d`

*Note: if you followed this tutorial by creating the interface in Ionic Creator yourself, then you will have a different creator number. Of course, you're free to use mine if you wish to do so. Also, you're free to name your project any way you want, I chose __Ionic_2ndTutorial__ in this example.*

After you execute the upper command in your terminal, you should see something similar to the following output:

```
C:\Users\Nikola\Desktop\IonicTesting>ionic start Ionic_2ndTutorial creator:167ff268a01d
Creating Ionic app in folder C:\Users\Nikola\Desktop\IonicTesting\Ionic_2ndTutorial based on creator:167ff268a01d project
Downloading: https://github.com/driftyco/ionic-app-base/archive/master.zip
[=============================]  100%  0.0s

Downloading Creator Prototype: https://apps.ionic.io\api\v1\creator\167ff268a01d\download\html
Updated the hooks directory to have execute permissions
Update Config.xml
Initializing cordova project

Your Ionic project is ready to go! Some quick tips:
 * cd into your project: $ cd Ionic_2ndTutorial
 * Setup this project to use Sass: ionic setup sass
 * Develop in the browser with live reload: ionic serve
 * Add a platform (ios or Android): ionic platform add ios [android]
   Note: iOS development requires OS X currently
   See the Android Platform Guide for full Android installation instructions:
   https://cordova.apache.org/docs/en/edge/guide_platforms_android_index.md.html
 * Build your app: ionic build <PLATFORM>
 * Simulate your app: ionic emulate <PLATFORM>
 * Run your app on a device: ionic run <PLATFORM>
 * Package an app using Ionic package service: ionic package <MODE> <PLATFORM>

For more help use ionic --help or ionic docs

Visit the Ionic docs: http://ionicframework.com/docs

New! Add push notifications to your Ionic app with Ionic Push (alpha)!
https://apps.ionic.io/signup
+---------------------------------------------------------+
+ New Ionic Updates for August 2015
+
+ The View App just landed. Preview your apps on any device
+ http://view.ionic.io
+
+ Invite anyone to preview and test your app
+ ionic share EMAIL
+
+ Generate splash screens and icons with ionic resource
+ http://ionicframework.com/blog/automating-icons-and-splash-screens/
+---------------------------------------------------------+
```

As per the instruction in the output above let's `cd` into our project:

`cd Ionic_2ndTutorial`

Now, let's see how this application looks like in the browser by executing the following command in the Terminal/Command prompt:

`ionic serve`

> You should be familiar with this command from the [first tutorial](firstTutLink), and if not please take a look at it first. For those short in time, basically `ionic serve` starts up a local web browser and shows how our application would look like; which, actually is great for rapid development since you get to see changes instantly without needing to reload your browser.

We should see something similar to the image below:

![](http://i.imgur.com/Kp49zVS.png)

Btw, the ruller and the Device setting (Apple iPhone 6) are a part of an awesome Developer tools in Chrome browser, which I highly recommend.

> If you get an error similar to this:
> `Unable to locate the ionic.project file. Are you in your project directory? (CLI v1.6.4)`
> then you didn't run the `ionic serve` command in the project directory (as the error clearly 
> says), and so you should first `cd` into that directory (as mentioned in the previous step).

Now, let's go and see what our generated folder structure looks like and what is each folder responsible for.

## Ionic application folder structure
If you open up the project in your editor (I'm using Sublime Text 3 on the image below), you will see something like:

![](http://i.imgur.com/WAqDPs3.jpg)

Now we're going to explain what each of these files and folders represent in Ionic framework.

### hooks folder
**hooks** folder contains code for *Cordova hooks*, which are used to execute some code during the Cordova build process. For example, Ionic uses Cordova's **after_prepare** hook to inject platform specific (iOS, Android) CSS and HTML code.

### plugins folder
**plugins** folder contains Cordova plugins which are added to the project. You can add [any available Cordova plugin](http://plugins.cordova.io/#/) to your project, and you can even create your own if you're technical enough. As you can see from the image above we currently have 5 installed plugins (com.ionic.keyboard, cordova-plugin-console, cordova-plugin-device, cordova-plugin-splashscreen, and cordova-plugin-whitelist).

### scss folder
**scss** folder contains the **ionic.app.scss** file which is based on [SASS](http://sass-lang.com/). This file contains default variables like for example theme colors.

> SASS is an extension of CSS which adds additional features like nested rules, variables, mixins, etc. There are few frameworks built with Sass like for example [Compass](http://compass-style.org/), [Bourbon](http://bourbon.io/), and [Susy](http://susy.oddbird.net/). You can learn more about how to use Sass in the [official guide](http://sass-lang.com/guide).
 
You don't need to use Sass in Ionic if you don't want to (it's actually turned off by default), but if you're familiar with it, you can initialize Ionic to use it with the `ionic setup sass`, and Gulp (more about it below) will automatically compile the Sass code into CSS.

### www folder
**www** folder is the most important since it contains all the files our application is made of, and we will be spending most of our development time in this folder. This folder contains few files and additional subfolders which we'll address in more detail in the next section titled [Refactoring our application](#refactoring).

### bower.json and .bowerrc files
**bower.json** is a configuration file for [Bower](http://bower.io/).

> Bower is a package manager for front-end modules that are usually comprised of JavaScript and/or CSS. It lets us easily search for, install, update, or remove these front-end dependencies. If you want to learn more about Bower, you can check out my post [How to Manage Front-End JavaScript and CSS Dependencies with Bower](http://www.nikola-breznjak.com/blog/codeproject/how-to-manage-front-end-javascript-and-css-dependencies-with-bower-on-ubuntu-14-04/).

The contents of my `bower.json` file is shown below (you will see the same content if you cloned the project from Github or started it from my Ionic Creator template):
```
{
  "name": "HelloIonic",
  "private": "true",
  "devDependencies": {
    "ionic": "driftyco/ionic-bower#1.1.0"
  }
}
```

Bower stores the downloaded front-end modules in a folder which is defined in a **.bowerrc** file, and is set to **www/lib** by default.

The contents of the `.bowerrc` file is shown below:
```
{
  "directory": "www/lib"
}
```

### config.xml file
**config.xml** is a configuration file for the [Cordova](https://cordova.apache.org/) project (as you may remeber from the [first post](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows), Ionic is built on top of Cordova). It contains some meta information about the app like permissions and a list of Cordova plugins which are used in the app. To learn more about available settings in the `config.xml` file, please refer to the [official documentation](https://cordova.apache.org/docs/en/4.0.0/config_ref_index.md.html).

### gulpfile.js file
**gulpfile.js** is a configuration file for [Gulp](http://gulpjs.com/).

> Gulp is a JavaScript task runner that helps with rapid development by making use of its multiple plugins for different tasks like concatenation, minifying, automatic unit test running, LESS/SASS to CSS compilation, copying modified files to different directories, etc...

Another popular task runner is [Grunt](http://gruntjs.com/). However, from my personal experience I tend to favor Gulp as well.

*Note: you don't need to learn Gulp in order to use Ionic. If, however, you want, you can check out the official [Getting started documentation](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md).*

### ionic.project
**ionic.project** is a configuration file for Ionic, used to store meta information about Ionic project and the associated Ionic.io cloud account. We're not going to use Ionic.io cloud account just yet, we'll show how to use it in the next tutorial.

### package.json
**package.json** is a file used by `npm` to store versions of the `npm` packages installed in the current project.

> `npm` (Node.js Package Manager) is a CLI tool which comes with Node.js installation and it's used for installing other tools like aftorementioned Bower, Gulp, etc...

Contents of my `package.json` file, shown below, is useful as we can see which Cordova plugins and dependencies are installed for the project, as well as some meta information like *name*, *version* and *description*:
```
{
  "name": "ionic_2ndtutorial",
  "version": "1.0.0",
  "description": "Ionic_2ndTutorial: An Ionic project",
  "dependencies": {
    "gulp": "^3.5.6",
    "gulp-sass": "^1.3.3",
    "gulp-concat": "^2.2.0",
    "gulp-minify-css": "^0.3.0",
    "gulp-rename": "^1.2.0"
  },
  "devDependencies": {
    "bower": "^1.3.3",
    "gulp-util": "^2.2.14",
    "shelljs": "^0.3.0"
  },
  "cordovaPlugins": [
    "cordova-plugin-device",
    "cordova-plugin-console",
    "cordova-plugin-whitelist",
    "cordova-plugin-splashscreen",
    "com.ionic.keyboard"
  ],
  "cordovaPlatforms": []
}
```

### .gitignore and README.md file
Both .gitignore and README.md are files related to [GitHub](https://github.com/).

I'm sure most of the readers know and use GitHub (and consequently [Git](https://git-scm.com/)), but just for brevity, let see [how Wikipedia defines GitHub](https://en.wikipedia.org/wiki/GitHub):

> GitHub is a Web-based Git repository hosting service, which offers all of the distributed revision control and source code management (SCM) functionality of Git as well as adding its own features. Unlike Git, which is strictly a command-line tool, GitHub provides a Web-based graphical interface and desktop as well as mobile integration. It also provides access control and several collaboration features such as bug tracking, feature requests, task management, and wikis for every project.

So, [Git]((https://git-scm.com/)) on the other hand is (quoted from the official site):

>a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.
Git is easy to learn and has a tiny footprint with lightning fast performance. It outclasses SCM tools like Subversion, CVS, Perforce, and ClearCase with features like cheap local branching, convenient staging areas, and multiple workflows.

Now that we got the basics out of the way, let's take a look at what the aftorementioned two files represent.

**.gitignore** is a configuration file for GitHub. Contents of the `.gitignore` file, shown below, basically instructs Git that it should not track folders **node_modules**, **platforms** and **plugins** when uploading the code to GitHub.
```
# Specifies intentionally untracked files to ignore when using Git
# http://git-scm.com/docs/gitignore

node_modules/
platforms/
plugins/
```

**README.md** is a [Markdown](http://daringfireball.net/projects/markdown/) file used on GitHub to explain the purpose and usage of your project.

> If you're not using Git (and Github) in your workflow, I highly encourage you to do so, as it will prove really useful in the long run.

## <a name="refactoring"></a>Refactoring our application
As said, we'll be spending most of our development time in the **www** folder and that's why we mentioned it just briefly in the previous section. We'll take a full deep dive in it here, by taking the two most imporant files (**index.html** and **app.js**) and explain their contents step by step.

Also, we'll refactor our code into separate files, since Ionic Creator put everything in one. Finally, we'll add the calculator logic so that our calculator will work as expected.

> Generally, putting all the code in one file and mixing logic and presentation (JavaScript and HTML code) is simply a big NO-NO, and often referred to as [spaghetti code](https://en.wikipedia.org/wiki/Spaghetti_code). You can learn more about refactoring your code from the cult book [Refactoring: Improving the Design of Existing Code](http://amzn.to/1LWo4Ow) by Martin Fowler, Kent Beck, and others. If you're searching for a bit lighter introduction to refactoring and good programing practices in general I can't recomment the classic [Code Complete 2](http://amzn.to/1LWomoq) by Steve McConnell enough.

### index.html file
The starting point of our Ionic application is `index.html`, whose contents is as follows:

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title></title>

    <style>
        .angular-google-map-container {
            width: 100%;
            height: 504px;
        }
    </style>

    <link href="lib/ionic/css/ionic.css" rel="stylesheet">
    <script src="lib/ionic/js/ionic.bundle.js"></script>

    <link href="http://code.ionicframework.com/ionicons/1.5.2/css/ionicons.min.css" rel="stylesheet">

    <!-- IF using Sass (run gulp sass first), then uncomment below and remove the CSS includes above
    <link href="css/ionic.app.css" rel="stylesheet">
    -->

    <script>
        // Ionic Starter App

        // angular.module is a global place for creating, registering and retrieving Angular modules
        // 'starter' is the name of this angular module example (also set in a <body> attribute in index.html)
        // the 2nd parameter is an array of 'requires'
        // 'starter.services' is found in services.js
        // 'starter.controllers' is found in controllers.js
        angular.module('app', ['ionic'])

        .run(function($ionicPlatform) {
          $ionicPlatform.ready(function() {
            // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
            // for form inputs)
            if(window.cordova &amp;&amp; window.cordova.plugins.Keyboard) {
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
    </script>
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

    <script id="page1.html" type="text/ng-template">
        <ion-view title="Calculator">
            <ion-content padding="false" class="has-header">
                <form>
                    <label class="item item-input" name="display">
                        <span class="input-label">Input</span>
                        <input type="text" placeholder="0" />
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
    </script>
</body>
</html>
```

> Here we'll go step by step throguh the `index.html` file by explaining the needed lines of code. I'm expecting that you have a decent understanding of HTML so I won't explain each and every HTML tag. Also, we'll remove the unnecessary pieces, and move some other parts of the code to different files.

The `<meta>` element:

`<meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">`

is used to properly resize our application on mobile devices.

Next, you can safely remove the `<style>` element completely out of the file:
```
<style>
    .angular-google-map-container {
        width: 100%;
        height: 504px;
    }
</style>
```
This CSS style is for the map template example, which we don't have here, so to be completely honest I'm not sure why they included it in this template. I'm guessing that if you'll be following this tutorial at a later stage this will be removed by the Ionic Creator team.

Since we're not going to use SASS in this example (we will cover this in the later tutorials), we can safely remove the following `<link>` tag:
```
<!-- IF using Sass (run gulp sass first), then uncomment below and remove the CSS includes above
<link href="css/ionic.app.css" rel="stylesheet">
-->
```

Next, as you can see, there is nowhere a reference to the `js/app.js` file inside the `index.html` file, thus meaning it's not used anywhere. If by any chance you've looked around the file structure of the application we created in the [first tutorial](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows) you will most likely find this strange. Indeed, Ionic Creator needs some additional tweaking, but I'm sure this will all be dealt with in the future.

Since the code inside the `<script>` element is JavaScript (AngularJS to be more exact) and it's mixed with HTML, we don't want to have that so we're going to copy the contents of the `<script>` element and paste it (by overwriting its whole contents) to the file `app.js` (resides in the `js` folder).

So, our `js/app.js` file should now look like this (first few lines of comments are ommited):
```
angular.module('app', ['ionic'])
.run(function($ionicPlatform) {
  $ionicPlatform.ready(function() {
    // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
    // for form inputs)
    if(window.cordova &amp;&amp; window.cordova.plugins.Keyboard) {
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

The next important thing is the **ng-app** directive which is added as an attribute on the `<body>` tag. This is just a way to let AngularJS know how to [bootstrap the application](https://docs.angularjs.org/guide/bootstrap). Important thing to note is that we have to use the same name in `app.js` file when we define the root angular module. As you can see from the `app.js` file:

`angular.module('app', ['ionic'])`

and from the `index.html` file:

`<body ng-app="app" animation="slide-left-right-ios7">`

in our example this name is simply **app**.

There's just one more thing we're going to do in the `index.html` file. See the another `script` tag inside our body tag:

`<script id="page1.html" type="text/ng-template">`

Well, this also looks like a good piece to take out and put into its own file. Again, if you saw the file structure from the application in the first tutorial you might remember that the default place to put the template files is inside the `templates` folder.

Since we don't have that folder in our example, let's create it inside the `www` folder. Then, create the `calculator.html` file and paste the contents of the previously mentioned `<script>` tag. This is how our `templates/calculator.html` file should look like now:

```
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

```
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
Contents of the **app.js** file bootstraps our Ionic (well, more acurately AngularJS) application, configures necessary plugings, and defines the states in our application. As we said before, the name of the root module matches the value given to the `ng-app` directive in the `index.html` file (**app** in our case). Also, we need to *inject* '`ionic`' module dependency to our root module so that we can use the Ionic library directives and other Ionic components.

`$ionicPlatform.ready` event is triggered after the device (your mobile phone on which the application is started) is all set up and ready to use. This includes plugins which are used with this project. If you would try to check if some plugin is available outside the `ready` function callback, you could get wrong results due to the fact that it is possible that some plugin would not have been set up just yet. Using the aforementioned `$ionicPlatform.ready` event and placing your code to check for plugin instantiations solves these issues. You can learn more about ionicPlatform utility methods here: [http://ionicframework.com/docs/api/utility/ionic.Platform/](http://ionicframework.com/docs/api/utility/ionic.Platform/).

Inside the `$ionicPlatform.ready` function's callback, we're detecting if a Keyboard plugin is available in our Ionic application. If so, we're hiding the keyboard accessory bar (buttons like `next`, `previous` and `done`). You can change these settings, and quite a bit of additional ones (see a [full list here](https://github.com/driftyco/ionic-plugin-keyboard)), as per your project's requirements.

As we mentioned before, you can check the list of installed plugins in your `package.json` file. But, also, you can check them from your Terminal/Command prompt by executing the following command:

`ionic plugin list`

In our case, the output should be as follows:

```
com.ionic.keyboard 1.0.4 "Keyboard"
cordova-plugin-console 1.0.1 "Console"
cordova-plugin-device 1.0.1 "Device"
cordova-plugin-splashscreen 2.1.0 "Splashscreen"
cordova-plugin-whitelist 1.0.0 "Whitelist"
```

From the listing above we can see that there is no StatusBar plugin (org.apache.cordova.statusbar), so we can safely remove the corresponding lines in the `app.js` file.

Inside the `config` function callback we're setting the routes for our application. Ionic uses [AngularUI Router](https://github.com/angular-ui/ui-router) which uses the concept of states. Since we moved our calculator UI into the `calculator.html` file inside the `templates` folder, we have to adjust the path to it accordingly. Also, we'll put our calculator logic inside the so called `CalculatorCtrl` controller. The state definition, with the acompanying controller definition should now look like this:

```
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

```
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

So, basically, what were doing here is we're defining a new AngularJS module called `calculator`, and we're defining a controller called `CalculatorCtrl`. In the `CalculatorCtrl` we're injecting AngularJS's $scope variable so that we can access the DOM in the `calculator.html` file.

Before explaining this rather simple code, we must not forget to inject the `calculator` module to our root app module in the `app.js` file, so that it will now look like this:

`angular.module('app', ['ionic', 'calculator'])`

Ok, so, back to our controller code; first, we're defining a variable `result` on the $scope, so that we can show it in our template file (`calculator.html`).

Next, we defined a new function called `btnClicked` which accepts one parameter named `btn`. Inside the function we're checking for the passed parameter and we have three cases based on the value of the parameter that is passed:

+ if the parameter equals `C` in that case we're clearing the result
+ if the parameter equals `=` in that case we calculate the equation and show the result
+ in any other case we're just appending the buttons value to the current equation

> Please note that because of brevity we haven't added any additional checks like if someone accidentally clicks two times on a button which represents some operation. We will "fix" this in the next tutorial when we'll be polishing the application. Also, you may have read on the Internet that using eval is wrong or just bad. True, it **can** be bad if used to evaluate some other user input, but in this case we'll be fine.

## Finishing the calculator template
The only thing which is left for us to do is to add the appropriate function calls on the buttons in the `calculator.html` template. Basically, all that we have to do is add the `ng-click="btnClicked()` function call to each button, and pass in the appropriate parameter. The way this should look like is as follows:

```
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

As mentioned in the begining of this tutorial, you can take a look at the source code on [GitHub](https://github.com/Hitman666/Ionic_2ndTutorial), or you can take a look at the [live example](http://nikola-dev.com/IonicCalculator/mobile.html) of this application.

## Conclusion
In this tutorial we showed you how to create a calculator application step by step. We showed how to create a mockup of our idea, then we showed how to create an interface by using Ionic Creator, and finally how to refactor our application and create the logic with controllers.

In the next tutorial we're going to take a look  at how to test your application on the real physical devices, and also how to use Ionic.io cloud service to share our application with other users without going through the app store. Also, we're going to take a look how to implement Google Analytics and Google Admob ads, and how to prepare our application for the stores. Also, we'll take a look at how to customize the icons and the application's splash screen.

If you have any questions about this tutorial please read the following help guides. Of course, I encourage you to ask a question in the comments so that anyone else who may have a silmiar question or doubt, learns something too.

With this I leave you and hope you had a great learning experience with following this tutorial. See you in the next installment of this series!

## How to get help with Ionic framework
Few of the resources that proved indispensable when I was learning about Ionic:

+ For a quick framework reference, I'm suggesting the [official documentation](http://ionicframework.com/docs/), which is indeed very good.
+ If you're in search for a good book about Ionic, then you don't need to look furhter than [Ionic in Action: Hybrid Mobile Apps with Ionic and AngularJS](http://amzn.to/1EjwYNJ)
+ One of the best resources on the net for programming related questions, about which you've no doubt heard, is [StackOverflow](http://stackoverflow.com/). You can view the specific Ionic tagged questions via [this link](http://stackoverflow.com/questions/tagged/ionic) or even [this one](http://stackoverflow.com/questions/tagged/ionic-framework). However, I urge you to please read their [help page](http://stackoverflow.com/help) before posting questions (especially if you're a new user to StackOverflow) because otherwise you may have a bad experience. If you're a new StackOverflow user who's in a hurry to get acquainted with it, make sure you at least go through this [2-minute tour](http://stackoverflow.com/tour).
+ To get started with AngluarJS try [these](http://angular.codeschool.com/) [few](https://hackhands.com/finishing-Angular-TODO-application-deploying-production/) [resources](https://docs.angularjs.org/tutorial), not necessarily in that particular order.
+ Let me just kind of repeat myself by saying that if you have **any questions** about Ionic framework, or you had trouble following this tutorial (you couldn't install something), or you would like to suggest what kind of tutorials you would like to see regarding Ionic in the future, then please **share it in the comments** below. Also, you can reach me personally via @[HitmanHR](https://twitter.com/hitmanhr) or [my blog](http://www.nikola-breznjak.com/blog). I'll do my best to  answer all of your questions.
+ If, however, you favor "one on one" help, you can reach me [via HackHands](https://hackhands.com/dashboard/request/hitman666/preferred).
