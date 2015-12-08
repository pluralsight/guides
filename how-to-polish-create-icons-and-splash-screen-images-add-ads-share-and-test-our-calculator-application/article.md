![How to polish, create icons and splash screen images, add ads, share and test our calculator application](http://i.imgur.com/uiU4uOs.png)

This is the third post in a series of posts which will teach you how to take advantage of your web development knowledge in building hybrid applications for iOS and Android. The first post in this series was all about [How to get started with Ionic framework on Windows and Mac](http://blog.pluralsight.com/ionic-framework-on-mac-and-windows), the second one was about [How to create a calculator application with Ionic framework by using Ionic Creator for UI](http://tutorials.pluralsight.com/review/how-to-create-a-calculator-application-with-ionic-framework-by-using-ionic-creator-for-ui/article.md).

This third post explains:

+ How to polish our existing calculator application
+ How to create icons and splash screen images automatically in Ionic framework
+ How to implement Google AdMob ads
+ How to use Ionic.io cloud service to share our application with other users without going through the app stores
+ How to test our application on the real physical devices and emulators


# How to polish our existing calculator application
So, if you've been following this tutorial series then you probably already have the Calculator application ready and running on your machine in the web browser by using the `ionic serve` command.

If you just dropped by, or you would like to "start anew" you can clone the finished version of the 2nd tutorial from [Github](https://github.com/Hitman666/Ionic_2ndTutorial). After cloning, you should run `npm install` to install all the dependencies. After this the command `ionic serve` should run the application locally on your computer, and you should see it open up in your default browser.

So, what we have at this point is a working simple calculator application that lacks some sanitization checks and design improvements. We'll fix this in this section.

## Sanitization check
In our current application, we don't have a security measure against the possible malformed input. For example, one could enter two plus (+) signs one after another, which would consequently produce an error.

So, in order to handle this, we're going to wrap our evaluation line of code in the try/catch block and we're going to show an alert to the user using the [$ionicPopup](http://ionicframework.com/docs/api/service/$ionicPopup/) service (injected as a dependency in the `CalculatorCtrl` controller). The whole contents of the `CalculatorCtrl.js` file should look like this now:

```
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

Additionally, we're checking to see if the `$scope.result` variable contains anything at the time the equals button is pressed, in order to avoid the `undefined` error that would otherwise happen. *You can test this yourself by running the application and then first clicking on the equal button (=) and then on for example button 5, and you would see the text 'undefined5' appear in the Result area.*

> Also, you may have noticed that we added the [toFixed](http://www.w3schools.com/jsref/jsref_tofixed.asp) after our `eval` function call, in order to show no decimal places in case of a division operation. At this point you certainly realize the "problem" of our SuperSimple Calculator - it has no way of entering the point (.), but I hope that by now you learned enough to add this option yourself, if you would need it. If, however, you would like me to add this, please request it in the comments and I'll make sure to add it in the update of this tutorial.

The way this should look like now if we intentionally make an error while inputting the formula is shown on the image below.

![](http://i.imgur.com/RNQ2yOS.jpg)

> Clearly, there are multiple ways you could approach this problem further in order to create a better user experience. One idea is to check on every button click if the current formula would execute correctly with `eval` and if not then immediately inform the user, or completely ignore the last clicked button. I encourage you to create your own way of the "improved [UX](https://en.wikipedia.org/wiki/User_experience)" and share it in the comments.

## Design changes
If we're being honest, our app currently doesn't look quite *representative*. So, we're going to change that in order to make it a bit nicer. Most of the changes that we're going to make will be in the CSS files. However, if you talk with any web designer these days they will tell you that using purse CSS is, well, outdated. Ionic, as awesome as it is, has a support for [SASS](http://sass-lang.com/). So, in order to take advantage of the variables, nesting, mixins and all other great stuff that SASS provides, we're going to prepare our project to use SASS by entering the following command in the Terminal/Command prompt:

`ionic setup sass`

Along with some other npm install related output, you should see something like this:

```
Successful npm install
Updated C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\www\index.html <link href> references to sass compiled css

Ionic project ready to use Sass!
 * Customize the app using scss/ionic.app.scss
 * Run ionic serve to start a local dev server and watch/compile Sass to CSS

[14:56:01] Using gulpfile ~\Desktop\IonicTesting\Ionic_3rdTutorial\gulpfile.js
[14:56:01] Starting 'sass'...
[14:56:02] Finished 'sass' after 1.76 s

Successful sass build
Ionic setup complete
```

Our main starting point in changing how our application looks like is the `scss/ionic.app.scss` file. It's contents is short, and even though you may not have used SASS before, I'm sure it will look logical to you:

```
/*
To customize the look and feel of Ionic, you can override the variables
in ionic's _variables.scss file.

For example, you might change some of the default colors:

$light:                           #fff !default;
$stable:                          #f8f8f8 !default;
$positive:                        #387ef5 !default;
$calm:                            #11c1f3 !default;
$balanced:                        #33cd5f !default;
$energized:                       #ffc900 !default;
$assertive:                       #ef473a !default;
$royal:                           #886aea !default;
$dark:                            #444 !default;
*/

// The path for our ionicons font files, relative to the built CSS in www/css
$ionicons-font-path: "../lib/ionic/fonts" !default;

// Include all of Ionic
@import "www/lib/ionic/scss/ionic";
```

As you can see, here we can override the default Ionic colors. Since you most probably have programming background, I'm sure you realize how important and useful SASS is.

When, for example, you want to change the color you only need to change it in one place in one variable, without having to trace it through all the files and change the certain color in all the places that it is used. 

Also, here you can see that we're including (better said importing) the other Ionic SASS files, which are nicely structured in different files. You don't have to worry about multiple files because the gulpfile task produces a single minified version of the CSS on every save. *I encourage you to check out the contents of the gulpfile.js file and see for yourself*.

On the image below you can see what I came up with after changing few of the CSS rules (and a slight addition to our HTML template that I'll address additionally):
![](http://i.imgur.com/QijEnmp.png)

I'm sure that more design-inclined readers will come up with way more slick design than this and I encourage you to share your images/scss changes with the rest of us, and I promise to incorporate the best one in the tutorial (with the proper credits, of course).

Now we're going to go through the parts that were changed so that you can follow along in your example and see for yourself. But, before you start doing the changes you'll need to add the following link code to your `index.html` file just below the `<title>` tag in order to load the CSS file generated through SASS (we removed this in the [2nd tutorial]()):

`<link href="css/ionic.app.css" rel="stylesheet">`

As for the changes, here is the final content of the `templates/calculator.html` file:
```
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
Here is the breakdown of the changes that I made:

+ I first removed the `<span class="input-label">Result</span>` and edited the `form` element to just contain the following line of code: 

`<form><input type="text" placeholder="0" ng-model="result" class="myResultInput"></form>`

+ Notice the added class `myResultInput` on the `input` element
+ Also, notice the `scroll="false"` on the `ion-content` tag, which disables content scrolling
+ I changed the title to SuperSimple Calculator from just Calculator on the `ion-view` tag
+ Then I wrapped this `form` element in a `div` element with the class `myInputRow`
+ Next, on the every `div` element with the class `button-bar` I added the class `row`
+ Finally, I wrapped this whole content in the `div` element with class `home-container`

In the `scss/ionic.app.scss` file I added the definitions for these new classes:

```
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

```
$button-block-margin: 0px !default;
$button-font-size: 32px !default;
$button-height: 70px !default;
```

Finally, in the `lib/scss/_button-bar.scss` file I changed the value of the following rule inside the `button-bar .button` rule:

`border-width: 1px;`

This way we now have an interface which fills the whole available content.

# How to create icons and splash screen images automatically in Ionic framework
The icon is an important part of your application because it represents your application's brand, and it helps to identify quickly where the app is on your phone. In case you're familiar with creating apps then you will remember that it is a tedious process to create a lot of different size images both for iOS and Android platforms. Also, the same goes for the so-called splash screen that shows up every time the application starts. Although having a splash screen is not mandatory, it certainly adds up to the feeling of a complete and professional application, which one would certainly want to convey with his application.

Ionic helps tremendously with this by providing a single Ionic CLI command to generate all the needed icon and splash screen sizes for us automatically. Also, Ionic created [Photoshop Icon Template](http://code.ionicframework.com/resources/icon.psd) and [Photoshop Splash Screen Template](http://code.ionicframework.com/resources/splash.psd), which you can download for free and use as a guideline for creating an icon.

For when you're creating a branded product having a custom made icon is definitelly a must. However, in this case, we'll show how to use one of the free services to search for a free icon which we can use in our application (even if our application is a commercial application). I tend to use [IconFinder](https://www.iconfinder.com) a lot, and here are the setting which you have to use in order to filter out the calculator images which are Free (PRICE) and can be used in commercial applications and which do not even require a link back (LICENSE TYPE). *Of course, you can also choose to buy an image if you happen to find one that you like*. You can additionally search by format, size and background. The filters should look like this:
![](http://i.imgur.com/TDkaEbp.jpg)

We're going to use the first one in the second row. Simply click on it, and you should get to the download page that looks like it's shown on the image below:
![](http://i.imgur.com/zTVcOqT.jpg)

To download it just click on the green PNG button. We're going to alter it just a bit by making it a size of 128x128px, so that it will finally look like this:
![](http://i.imgur.com/8pl8utM.jpg)

Now (download if you haven't yet and) open up the [Photoshop Icon Template](http://code.ionicframework.com/resources/icon.psd) and you should see:
![](http://i.imgur.com/bVtqbnW.jpg)

> Don't worry if you don't have Photoshop, you can use open source tools like for example  [Gimp](http://www.gimp.org/) or [Paint.NET](http://www.getpaint.net/index.html).

Now drag the calculator icon into the Photoshop and make sure you place the icon within the bounds of the blue guides. You should have something like this:
![](http://i.imgur.com/JROeVCX.jpg)

Next, remove the guides by going to `View->Show` and remove the `Guides` option.
![](http://i.imgur.com/JKgvmFu.jpg)

Also, you can remove the Grid in the same way if you like. Finally, remove the `icon-cut-guide` layer by clicking on the eye icon next to it:
![](http://i.imgur.com/x2l1INq.jpg)

Now save the file by going to `File->Save for Web` and select PNG-24 and after that click the `Save...` button and save the image to Desktop as `icon.png`:
![](http://i.imgur.com/wincuBx.jpg)

Repeat the process to create the splash screen image and name is `splash.png`. The image I came up with looks like this in Photoshop:
![](http://i.imgur.com/3hPXcza.jpg)

> If you've followed the [2nd tutorial](), you should have the app. In case you don't have it, you can [clone it from Github]().

Now that you have both **icon.png** and **splash.png** images ready, you have to copy them in the **resources** folder in the root of the application (next to the folders like `www` and `plugins`). If you don't have this folder, then create it. Next, navigate with your Terminal/Command prompt to the root folder of the application and run the following command:

`ionic resources`

If at this point you get an error like this:

```
C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial>ionic resources
 ? No platforms have been added.
 ? Please add a platform, for example: ionic platform add ios
 ? Or provide a platform name, for example: ionic resources android
```

that means that you haven't added any platforms yet to which Ionic should build. Since we're going to build the app for both Apple Store and Android Play Store we're going to use the following two commands:

```
ionic platform add android
ionic platform add ios
```

You should see an output similar to this:

```
C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial>ionic platform add android
Updated the hooks directory to have execute permissions
Adding icons for platform: android
npm http GET https://registry.npmjs.org/cordova-android
npm http 304 https://registry.npmjs.org/cordova-android
npm http GET https://registry.npmjs.org/cordova-android
npm http 304 https://registry.npmjs.org/cordova-android
Adding android project...
Creating Cordova project for the Android platform:
        Path: platforms\android
        Package: com.ionicframework.ionic2ndtutorial616047
        Name: Ionic_2ndTutorial
        Activity: MainActivity
        Android target: android-22
Copying template files...
Android project created with cordova-android@4.0.2
Running command: C:\NodeJS\node.exe C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\hooks\after_prepare\010_add_platform_class.js 

C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial
add to body class: platform-android
Installing "com.ionic.keyboard" for android
Installing "cordova-plugin-console" for android
Installing "cordova-plugin-device" for android
Installing "cordova-plugin-splashscreen" for android
Installing "cordova-plugin-whitelist" for android
Saving platform to package.json file

C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial>ionic platform add ios
Updated the hooks directory to have execute permissions
Adding icons for platform: ios
WARNING: Applications for platform ios can not be built on this OS - win32.
npm http GET https://registry.npmjs.org/cordova-ios
npm http 200 https://registry.npmjs.org/cordova-ios
npm http GET https://registry.npmjs.org/cordova-ios
npm http 200 https://registry.npmjs.org/cordova-ios
npm http GET https://registry.npmjs.org/cordova-ios/-/cordova-ios-3.8.0.tgz
npm http 200 https://registry.npmjs.org/cordova-ios/-/cordova-ios-3.8.0.tgz
Adding ios project...
iOS project created with cordova-ios@3.8.0

Running command: C:\NodeJS\node.exe C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\hooks\after_prepare\010_add_p
latform_class.js C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial
add to body class: platform-ios
Installing "com.ionic.keyboard" for ios
Installing "cordova-plugin-console" for ios
Installing "cordova-plugin-device" for ios
Installing "cordova-plugin-splashscreen" for ios
Installing "cordova-plugin-whitelist" for ios
Saving platform to package.json file
```

After this you can safely run the `ionic resources` command and you should get the following output:

```
C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial>ionic resources
Ionic icon and splash screen resources generator
 uploading icon.png...
 uploading splash.png...
 ? icon.png (192x192) upload complete
 ? splash.png (2208x2208) upload complete
 generating splash ios Default~iphone.png (320x480)...
 generating splash ios Default@2x~iphone.png (640x960)...
 generating splash ios Default-Portrait~ipad.png (768x1024)...
 ? splash ios Default~iphone.png (320x480) generated
 generating splash ios Default-Portrait@2x~ipad.png (1536x2048)...
 ? splash ios Default@2x~iphone.png (640x960) generated
 generating splash ios Default-Landscape~ipad.png (1024x768)...
 ? splash ios Default-Portrait~ipad.png (768x1024) generated
 generating splash ios Default-Landscape@2x~ipad.png (2048x1536)...
 ? splash ios Default-Landscape~ipad.png (1024x768) generated
 generating splash ios Default-Landscape-736h.png (2208x1242)...
 ? splash ios Default-Portrait@2x~ipad.png (1536x2048) generated
 generating splash ios Default-736h.png (1242x2208)...
 ? splash ios Default-Landscape-736h.png (2208x1242) generated
 generating splash ios Default-667h.png (750x1334)...
 ? splash ios Default-667h.png (750x1334) generated
 generating splash ios Default-568h@2x~iphone.png (640x1136)...
 ? splash ios Default-736h.png (1242x2208) generated
 generating splash android drawable-port-xxxhdpi-screen.png (1280x1920)...
 ? splash android drawable-port-xxxhdpi-screen.png (1280x1920) generated
 generating splash android drawable-port-xxhdpi-screen.png (960x1600)...
 ? splash android drawable-port-xxhdpi-screen.png (960x1600) generated
 generating splash android drawable-port-xhdpi-screen.png (720x1280)...
 ? splash ios Default-568h@2x~iphone.png (640x1136) generated
 generating splash android drawable-port-hdpi-screen.png (480x800)...
 ? splash android drawable-port-hdpi-screen.png (480x800) generated
 ? splash android drawable-port-mdpi-screen.png (320x480) generated
 generating splash android drawable-port-ldpi-screen.png (240x320)...
 ? splash android drawable-port-xhdpi-screen.png (720x1280) generated
 generating splash android drawable-land-xxxhdpi-screen.png (1920x1280)...
 ? splash android drawable-port-ldpi-screen.png (240x320) generated
 generating splash android drawable-land-xxhdpi-screen.png (1600x960)...
 ? splash ios Default-Landscape@2x~ipad.png (2048x1536) generated
 generating splash android drawable-land-xhdpi-screen.png (1280x720)...
 ? splash android drawable-land-xhdpi-screen.png (1280x720) generated
 generating splash android drawable-land-hdpi-screen.png (800x480)...
 ? splash android drawable-land-xxxhdpi-screen.png (1920x1280) generated
 generating splash android drawable-land-mdpi-screen.png (480x320)...
 ? splash android drawable-land-hdpi-screen.png (800x480) generated
 generating splash android drawable-land-ldpi-screen.png (320x240)...
 ? splash android drawable-land-ldpi-screen.png (320x240) generated
 generating icon ios icon-small@3x.png (87x87)...
 ? splash android drawable-land-xxhdpi-screen.png (1600x960) generated
 generating icon ios icon-small@2x.png (58x58)...
 ? icon ios icon-small@3x.png (87x87) generated
 generating icon ios icon-small.png (29x29)...
 ? icon ios icon-small.png (29x29) generated
 generating icon ios icon-76@2x.png (152x152)...
 ? splash android drawable-land-mdpi-screen.png (480x320) generated
 generating icon ios icon-76.png (76x76)...
 ? icon ios icon-small@2x.png (58x58) generated
 generating icon ios icon-72@2x.png (144x144)...
 ? icon ios icon-76@2x.png (152x152) generated
 generating icon ios icon-72.png (72x72)...
 ? icon ios icon-76.png (76x76) generated
 generating icon ios icon-60@3x.png (180x180)...
 ? icon ios icon-72@2x.png (144x144) generated
 generating icon ios icon-60@2x.png (120x120)...
 ? icon ios icon-60@3x.png (180x180) generated
 generating icon ios icon-60.png (60x60)...
 ? icon ios icon-72.png (72x72) generated
 generating icon ios icon-50@2x.png (100x100)...
 ? icon ios icon-60@2x.png (120x120) generated
 generating icon ios icon-50.png (50x50)...
 ? icon ios icon-60.png (60x60) generated
 generating icon ios icon-40@2x.png (80x80)...
 ? icon ios icon-50@2x.png (100x100) generated
 generating icon ios icon-40.png (40x40)...
 ? icon ios icon-40@2x.png (80x80) generated
 generating icon ios icon@2x.png (114x114)...
 ? icon ios icon-50.png (50x50) generated
 generating icon ios icon.png (57x57)...
 ? icon ios icon-40.png (40x40) generated
 generating icon android drawable-xxxhdpi-icon.png (192x192)...
 ? icon ios icon@2x.png (114x114) generated
 ? icon android drawable-xxhdpi-icon.png (144x144) generated
 generating icon android drawable-xhdpi-icon.png (96x96)...
 ? icon ios icon.png (57x57) generated
 ? icon android drawable-hdpi-icon.png (72x72) generated
 generating icon android drawable-mdpi-icon.png (48x48)...
 ? icon android drawable-xxxhdpi-icon.png (192x192) generated
 generating icon android drawable-ldpi-icon.png (36x36)...
 ? icon android drawable-xhdpi-icon.png (96x96) generated
 ? icon android drawable-ldpi-icon.png (36x36) generated
 ? icon android drawable-mdpi-icon.png (48x48) generated
```

From the output you can see how much icons were created and hopefully now you can see how much time this saved. All the needed configuration regarding the icons and splash screens was generated by Ionic and placed in the **config.xml** file.

It's worth noting that you will not see the icon nor the splash screen when using the browser testing or Ionic View testing (discussed in more detail in the [How to use Ionic.io cloud service to share our application with other users without going through the app store](#) section below). Instead, you will only see these once you deploy them to the actual physical device or the emulator (which we'll cover in the [How to test our application on the real physical devices and emulators](#) section below).

You can add an iOS platform if your developing on a Windows machine, and `ionic resources` command will generate icons and splash screens for it. However, keep in mind that you will not be able to build the project for iOS on your Windows machine. Instead, you'll need a Mac computer in order to do so. We'll cover building the app in more detail in the aforementioned [How to test our application on the real physical devices and emulators](#) section below.

# How to implement Google AdMob ads
There are multiple ways you can earn money with your app these days and here are just a few:

+ Paid app - ask for a flat out price for your app on the App/Play Store
+ Freemium - give the app for free but charge for in-app purchases like adding some extra features (think more gold or faster production in game apps)
+ Ad-based - show ads inside your application. Potentially offer the in-app purchase to remove the adds

What we're going to cover here is the Ad-based monetization option where we'll show how to add Google AdMob ads to our calculator application. There are two parts to implementing Google AdMob ads to our Ionic project. I broke down the steps in two parts: AdMob settings and Ionic settings.

## AdMob settings

Let's start with AdMob settings:

1.  Sign in/Sign up for AdMob at[https://www.google.com/admob/](https://www.google.com/admob/)
2.  Click the **Monetize new app **button:

[![Screen Shot 2015-05-05 at 23.21.06](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.21.06.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.21.06.png)

3.  If your app is not yet published you can add it manually:

[![Screen Shot 2015-05-05 at 23.23.07](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.23.07.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.23.07.png)

4.  Create new tracking ID:

[![Screen Shot 2015-05-05 at 23.25.20]
(http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.25.20.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.25.20.png)

5.  Configure the adds type, size, placement, style, name:

[![Screen Shot 2015-05-05 at 23.26.29]
(http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.26.29.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.26.29.png)

6.  You can read additional info on how to implement GA and AdMob, but for now let's just click Done:

[![Screen Shot 2015-05-05 at 23.28.10](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.28.10.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.28.10.png)

7.  You will now see the following similar screen: 

[![Screen Shot 2015-05-05 at 23.30.11](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.30.11.png)](http://www.nikola-breznjak.com/blog/wp-content/uploads/2015/05/Screen-Shot-2015-05-05-at-23.30.11.png)

The most important thing to note here is this **Ad unit ID**, which in my test case is **ca-app-pub-7957971173858308/3599533362**. Please make a note of this string as it's the most important part of this setting.

8. Create as much of the Ad units as you may need (for each platform[iOS, Android] and ad format [Banner, Interstitial]). In my case, I just created the additional Interstitial ad and will use them in both iOS and Android devices.

### Ionic settings
Navigate to the root of the application with your Terminal/Command prompt and execute the following command to add the [cordova-plugin-admobpro](https://github.com/floatinghotpot/cordova-admob-pro) plugin:

`ionic plugin add cordova-plugin-admobpro`

You should see the following output after running the command:

```
C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial>cordova plugin add cordova-plugin-admobpro
Fetching plugin "cordova-plugin-admobpro" via npm
npm http GET https://registry.npmjs.org/cordova-plugin-admobpro
npm http 200 https://registry.npmjs.org/cordova-plugin-admobpro
npm http GET https://registry.npmjs.org/cordova-plugin-admobpro/-/cordova-plugin-admobpro-2.9.6.tgz
npm http 200 https://registry.npmjs.org/cordova-plugin-admobpro/-/cordova-plugin-admobpro-2.9.6.tgz
Installing "cordova-plugin-admobpro" for android
Fetching plugin "cordova-plugin-extension" via npm
npm http GET https://registry.npmjs.org/cordova-plugin-extension
npm http 304 https://registry.npmjs.org/cordova-plugin-extension
Installing "cordova-plugin-extension" for android
Fetching plugin "cordova-plugin-googleplayservices" via npm
npm http GET https://registry.npmjs.org/cordova-plugin-googleplayservices
npm http 200 https://registry.npmjs.org/cordova-plugin-googleplayservices
npm http GET https://registry.npmjs.org/cordova-plugin-googleplayservices/-/cordova-plugin-googleplayservices-19.0.3.tgz
npm http 200 https://registry.npmjs.org/cordova-plugin-googleplayservices/-/cordova-plugin-googleplayservices-19.0.3.tgz
cordova-plugin-googleplayservices" will not install due to "C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\plugins\com.google.playservices" being installed.
Installing "cordova-plugin-admobpro" for ios
Installing "cordova-plugin-extension" for ios
```

> If you get an error like: `Error: 404 Not Found: cordova-plugin-admobpro` then please check that you indeed have the latest version of cordova installed. You can update to the newest version by executing `npm install -g cordova`

Now, add the following code to your app.js file, inside the `.run` function so that the run function would now look like:

```
.run(function($ionicPlatform) {
    $ionicPlatform.ready(function() {
        // Hide the accessory bar by default (remove this to show the accessory bar above the keyboard
        // for form inputs)
        if (window.cordova &amp;&amp; window.cordova.plugins.Keyboard) {
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

Of course, change it according to your own `admob_key` which you obtained in the first part (step 8). Also, you may want to have a different add for each platform and the way how you would initialize the `admobid` object is this:

```
// select the right Ad Id according to platform
var admobid = {};
if( /(android)/i.test(navigator.userAgent) ) { // for android &amp; amazon-fireos
    admobid = {
        banner: 'ca-app-pub-xxx/xxx',
        interstitial: 'ca-app-pub-xxx/yyy'
    };
}
else if(/(ipod|iphone|ipad)/i.test(navigator.userAgent)) { // for ios
    admobid = {
        banner: 'ca-app-pub-xxx/zzz',
        interstitial: 'ca-app-pub-xxx/kkk'
    };
}
else { // for windows phone
    admobid = {
        banner: 'ca-app-pub-xxx/zzz',
        interstitial: 'ca-app-pub-xxx/kkk'
    };
}
```

The documentation of this plugin is awesome with detailed examples and finished example projects, so if you run into any problems make sure to check it out. Of course, you can always ask below in the comments if you get stuck. One common thing that you might have to do is to install some extras in Android SDK manager (type `android sdk` in your Terminal/Command prompt to launch it) which are marked as **Installed** on the image below:
![](https://cloud.githubusercontent.com/assets/2339512/8176143/20533ec0-1429-11e5-8e17-a748373d5110.png)

For more information about the Android SDK manager, take a look at the section about [How to test our application on the real physical devices and emulators](#).

This plugin's documentation states an interesting fact that it is strongly recommend to use the Interstitial ad type, because it brings more than 10 times profit than the banner Ad. Here's the table from the [official documentation](https://github.com/floatinghotpot/cordova-admob-pro#tips):

| Ad Format             | Banner    | Interstitial   |
| --------------------- | --------- | -------------- |
| Click Rate            |   < 1%    |     3-15%      |
| RPM (1k impressions)  | 0.5$ - 4$ |    10-100$     |

> Banner ad is the small add that is usually placed in the bottom of the screen, whereas Interstitial ads are full screen ads that cover the interface of their host app. Therefore, you may want rather to opt for this kind of an ad instead for the Banner one. 

Of course, you can choose to use both also, which is exactly what we will do. First, add the following line of code in the `if (Admob)` case in the `app.js` file which prepares the Interstitial Ad for showing:

```
AdMob.prepareInterstitial( {adId:admobid.interstitial, autoShow:false} );
```

Now, lets say we want to show the Interstitial ad every 10th time the user clicks on any of our calculator buttons. To achieve this, we're going to set up a counter in our `CalculatorCtrl.js` file and increase it every time the button is pressed. Once the counter reaches the value 10, we will show the Interstitial Ad and reset the counter. This is how this looks like in the code:

```
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
        }
        else{
            $scope.result += btn;    
        }        

        if ($scope.counter++ == 10){
            AdMob.showInterstitial();
            $scope.counter = 0;
        }
    };
});
```

So, the only novelties in this file are the `$scope.counter` variable and the added `if` statement inside the `$scope.btnClicked` function.

We may need to play with the setting of after how many button presses we want to show the Interstitial ad. We may even add it only after the users presses the equals (=) button every 5th time, or we may even add it on a timer. Any way you choose, make sure you don't get too jumpy on your users or they'll quit using your app.

Just for reference, here is the code which you would have to add to the `if (AdMob)` case in `app.js` file in order to show the add every two minutes:

```setInterval(AdMob.showInterstitial, 2*60*1000);```

You can look at the accompanying [StackOverflow question](http://stackoverflow.com/questions/31095303/show-interstitial-ad-via-admob-in-ionic-every-2-minutes) which I answered which discusses this matter in more detail.

Of course, you would then probably want to remove the Interstitial ad showing on every 10-th click. Again, it's important to note that there's probably no exact formula here, but there are some best practices, and this is what Google has to say about it:

>Interstitial ads work best in apps with natural transition points. The conclusion of a task within an app, like sharing an image or completing a game level, creates such a point. Because the user is expecting a break in the action, it's easy to present an interstitial without disrupting their experience. Make sure you consider at which points in your app's workflow you'll display interstitials, and how the user is likely to respond.

You can learn a bit more about it [here](https://developers.google.com/admob/android/interstitial).

**The solution for which I personally opted for in the end** was to show the banner ads all the time and to show the Interstitial ad every 5th time when the equals (=) button has been pressed. Additionally I wait for 1 second after the 5th button click has been reached before showing the Interstitial ad. Here's how my CalculatorCtrl.js file looks like now:

```
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

# How to use Ionic.io cloud service to share our application with other users without going through the app store
Having the ability to immediately get feedback on something that you're working on, without having to install the app manually on your clients devices or waiting for the App Store/Play Store approval is indispensable for constant feedback loop which is crucial in rapid application development. *This is something that Eric Ries stresses a lot in his book [The Lean Startup](http://www.nikola-breznjak.com/blog/books/the-lean-startup-eric-ries/), which I already mentioned in the [2nd tutorial](#)*.

In Ionic this is actually easier than you may have thought. All you have to do is create the account on [https://apps.ionic.io](https://apps.ionic.io) and after that in your Terminal/Command prompt in the root of the project just type the following command:

`ionic upload`

This will prompt you to login with your username and password combination which you created in the previous step. After successful login the app will be uploaded to the Ionic.io cloud service and you will be able to view the app through an app called [Ionic View](http://view.ionic.io/), which you can download for free on your smartphone from the App Store/Play Store.

The output of the above command is short:
```
> ionic upload
Uploading app....
Saved app_id, writing to ionic.io.bundle.min.js...
Successfully uploaded (a82ec345)

Share your beautiful app with someone:

$ ionic share EMAIL

Saved api_key, writing to ionic.io.bundle.min.js...
```

And, as stated in the output, to share your app with a client or a friend, without first having to go through the App Store/Play Store, just type:

`ionic share EMAIL`

where, of course, you change EMAIL with an actual email address.

On the image below it's shown how the interface of [https://apps.ionic.io/apps](https://apps.ionic.io/apps) looks like once you're logged in and you can see the list of your uploaded apps:
![](http://i.imgur.com/cD57c7Q.png)

After you entered the command for sharing your application the person gets an email like this:
![](http://i.imgur.com/ZGkLwvn.png)

After your client/friend clicks on the `View app` link he's taken to the landing page shown on the image below where they basically have to login or create an account:
![](http://i.imgur.com/tPClB78.jpg)

Once the client/friend creates the account or logs in he will see the same app listed as we showed before. As noted before, in order for them to test this application on their phone, all they have to do is install the [Ionic View](http://view.ionic.io/) application, run it, login, click on the application and select `View App` as shown on the image below:
![](http://i.imgur.com/Ahce98q.jpg)

# How to test our application on the real physical devices and emulators
If you run the application (to which you added AdMob) in the browser, you will see an error in the Console output (if you're using [Chrome](https://www.google.com/chrome/) or [Firebug](http://getfirebug.com/)) that AdMob is not defined, as shown on the image below:
![](http://i.imgur.com/8hgXqoM.png)

Also, if you view the app in Ionic View you will notice that no ads will show up. This is because **Cordova plugins don't work in the browser, nor in the Ionic View**. Instead, they have to be tested on the real device or in an emulator (in iOS terminology the word simulator is used).

In this section, we're going to show how to install the needed prerequisites and how to run our application in an emulator and on the actual physical device for both iOS and Android.

Before we go any further, we have to add the `cordova.js` file.

> In the previous versions of Ionic, during development in the browser, this file would have resulted in a `404 Not found` error, but once deployed on a real physical device or emulator it would work fine. As of new versions of Ionic this is fixed so that the `cordova.js` file has a simple content (when included in `index.html` and tested in browser):
> 
> `// mocked cordova.js response to prevent 404 errors during development`

Add the following `script` tag just before the js/app.js script tag:

`<script src="cordova.js"></script>`

Just for reference, the whole contents of my `index.html` file is now as follows:

```
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

## iOS settings
The main tool for developing native iOS applications is [Xcode](https://developer.apple.com/xcode/). Xcode is praiseworthily free for download and it comes with a lot of simulators in which you can see how your app would look like on a real device.

However, if you want to build and deploy iOS applications to the App Store, you need to have a [Mac computer](http://www.apple.com/mac/) since Xcode only works on their operating system. Yes, **even if you're using Ionic, you need to use Xcode** to build the application for iOS.

There are some ways around this, like for example with using a so-called [Hackintosh](http://www.hackintosh.com/) computer, but honestly from my experience this is just not worth it. 

If you're really anti-Apple :), then you'll be happy to know that there are services which allow you to [rent a Mac in the cloud](http://www.macincloud.com/), just for the time you need to build your application. Also, Ionic promissed to build a service that will allow you to build mobile apps for any supported platform even if you don’t have a Mac, but we have to wait for this to become available.

> To **publish an app** in the App Store (or **test it on your own iOS device**) you'll need to purchase the *Apple Developer Program* license which costs **US $99 per year**. You’ll need to sign up at [http://developer.apple.com](http://developer.apple.com), but don't worry about this for now; we'll cover all the steps in our next tutorial where I'll guide you through the application deploy process for both the Apple's App Store and Google's Play Store.

### Installing the needed tools
As we've mentioned before, you need to install Xcode, and you can do that by opening up the App Store application on your Mac and search for Xcode. You install it as any other application by clicking on the download button and following the NextNextNext type of installation just like any other.
![](http://i.imgur.com/yS2WuBx.png)

After the installation is finished, open up Xcode, and navigate to `Xcode -> Preferences -> Downloads` tab. Here you have to download the latest version of the Simulator available. In my case, at the time of this writing, the version is iOS 8.4 Simulator, as you can see on the image below:
![](http://i.imgur.com/iah66tM.png)

Also, we need to install the ios-sim package with npm:

`npm install -g ios-sim`

### Running our application in a simulator
Now, in order to test our application in a simulator all we have to do is execute the following command:

`ionic emulate ios`

After a bunch of cryptic messages you should see an Xcode simulator open up and show your awesome application along with a splash screen and ads:
![](http://i.imgur.com/uiU4uOs.png)

If you test the app by doing five calculations, then after the fifth time you press the equals (=) button the Interstitial ad will appear, as shown on the image below:
![](http://i.imgur.com/LlVjkW9.png)

If you exit the simulator back to the home screen (by pressing `COMMAND + SHIFT + H) you will see the application's icon:
![](http://i.imgur.com/5KrFOmO.png)

When we're testing our application in browser with `ionic serve` we have that great live realod feature. We can have that too in the emulator, if we run the emulator with the extra flag `–l`. If we also want to see the console.log output, as we would see it in the browser, then we have to add the flag `–c`:

```
ionic emulate ios -lc
```

> Since we need to have an Apple Developer account in order to run the application on our phsyical phone device, we will leave this for our next tutorial where we'll also show how to deploy the application to the actual App Store.

## Android settings
There are no prerequisites for Android development regarding the operating systems; you can develop for Android on Mac, Linux, and Windows computers.

Android provides a number of tools for developers that are available for free from their [website](http://developer.android.com/).

> To publish an app in the Play Store you'll need to purchase the *Developer Program* license which costs only **US $25 one-time fee**. You’ll need to sign up at [https://play.google.com/apps/publish/signup/](https://play.google.com/apps/publish/signup/), but as said before - don't worry; we'll cover all the steps in our next tutorial.

### Installing the needed tools
Here you basically have two options:

+ Android Studio - full IDE with the [SDK](https://en.wikipedia.org/wiki/Software_development_kit) built in.
+ Android SDK Tools - only the SDK

Both can be downloaded from [http://developer.android.com/sdk/index.html](http://developer.android.com/sdk/index.html) but I recommend that you only download the SDK since we will not be using the actual Android IDE at all.
![](http://i.imgur.com/s4bis7N.png)

The installation of the SDK is also a simple NextNextNext type of installation on Windows machine (if you downloaded the .exe version), and if you downloaded the zip file (on both Mac or Windows) then you need to extract them to a certain folder (`C:\Dev` or `/Users/nikola/Dev` for example) and set up the PATH variable correctly.

On Mac the PATH variable is set like this:

`export PATH=$PATH:/Users/nikola/Dev/android-sdk/tools/`

You may want to put this line to your .bash_profile (or any related that you may have) file. For any further details about how to set the PATH variable you can take a look at [this tutorial](http://www.cyberciti.biz/faq/appleosx-bash-unix-change-set-path-environment-variable/).

As for Windows, the command would be:

`set PATH=%PATH%;C:\Dev\android-sdk\tools`

For more information on how to set the PATH variable on Windows you can take a look at [this tutorial]().

> You may get a message that you don't have java tools installed:
> ![](http://i.imgur.com/GnNWJkp.png)
> In that case simply click the More info button and you'll go to their website where you just have to click on the "Agree and start free download" button. After the download run the .dmg file (on Mac OS) and finish the NextNextNext installation. In that case, you will also have to download the JDK and you can do so from the [official site](http://www.oracle.com/technetwork/java/javase/downloads/index.html).

*To be honest, you may still encounter some errors while trying to get this to work and I've answered quite a few issues on StackOverflow. So, if you get stuck at this point, just drop me a line in the comments and I'll do my best to help.*

Once you've set the PATH variable correctly for your operating system, you can check if it's OK by opening up your Terminal/Command prompt and typing:

`android --help`

You should get an output similar to this one:

```
C:\Users\Nikola\Desktop>android
Usage:
    android [global options] action [action options]

Global options:
    -h --help       : Help on a specific command.
    -v --verbose    : Verbose mode, shows errors, warnings and all messages.
    --clear-cache   : Clear the SDK Manager repository manifest cache.
    -s --silent     : Silent mode, shows errors only.

Valid actions are composed of a verb and an optional direct object:

-    sdk              : Displays the SDK Manager window.
-    avd              : Displays the AVD Manager window.
-   list              : Lists existing targets or virtual devices.
-   list avd          : Lists existing Android Virtual Devices.
-   list target       : Lists existing targets.
-   list device       : Lists existing devices.
-   list sdk          : Lists remote SDK repository.
- create avd          : Creates a new Android Virtual Device.
-   move avd          : Moves or renames an Android Virtual Device.
- delete avd          : Deletes an Android Virtual Device.
- update avd          : Updates an Android Virtual Device to match the folders of a new SDK.
- create project      : Creates a new Android project.
- update project      : Updates an Android project (must already have an AndroidManifest.xml).
- create test-project : Creates a new Android project for a test package.
- update test-project : Updates the Android project for a test package (must already have an AndroidManifest.xml).
- create lib-project  : Creates a new Android library project.
- update lib-project  : Updates an Android library project (must already have an AndroidManifest.xml).
- create uitest-project: Creates a new UI test project.
- update adb          : Updates adb to support the USB devices declared in the SDK add-ons.
- update sdk          : Updates the SDK by suggesting new platforms to install if available.
```

Finally, you need to set up the SDK packages, and to do so run the following command in the Terminal/Command prompt:

`android sdk`

The SDK Manager allows you to download the platform files for any version
of Android. I recommend that you download just the most recent packages as shown on the image below:
![](http://i.imgur.com/0bTd3N0.png)

As you can see from the image, the things you need to check and then click on Install are:

+ Android SDK Tools
+ Android SDK Platform-tools
+ Android SDK Build-tools (choose the newest one)
+ Android 6.0 (API 23) - here you can actually go with 5.1

As mentioned in the [How to implement Google AdMob ads](#How to implement Google AdMob ads) section, you need to install some extra tools as shown on the image below:
![](http://i.imgur.com/4q5kL62.png)

Android SDK has an emulator that can emulate the screen size and resolution of most Android devices, but to be honest it's way too slow and klunky and I recommend that you do not use it. Instead, download [Genymotion](https://www.genymotion.com/#!/) which is free for personal use.

After you install and run Genymotion you will see an iterface like this:
![](http://i.imgur.com/4Fh0rJb.jpg)

Now you have to add an emulator, and you can do this by clicking on the `Add (+)` button where you'll get a broad selection of options as shown on the image below.
![](http://i.imgur.com/776dCgo.jpg)

Once you select one (I'm using a Samsung Galaxy S5) just click Next and confirm on the confirmation window:
![](http://i.imgur.com/eLajDKu.jpg)

Once you have the device all set up you can run it by clicking the Start button and Android will start up quickly:
![](http://i.imgur.com/l9MGObI.jpg)

Since Ionic sees the Genymotion device as a real device attached to your computer, you can't run `ionic emulate android`, as we did in the iOS section (with the obvious ios/android change). Instead, you need to run the Ionic's `run` command (before you do the build phase) like this:

`ionic build android &amp;&amp; ionic run android`

This would now run the application on the Genymotion emulator and it would look like shown on the image below:
![](http://i.imgur.com/I2cHeH4.jpg)

If you would like to run the application on your own Android device then you first have to run the following command to build the project:

`ionic build android`

The output of the command will be something like:

```
BUILD SUCCESSFUL

Total time: 5.536 secs
Built the following apk(s):
    C:\Users\Nikola\Desktop\IonicTesting\Ionic_3rdTutorial\platforms\android\build\outputs\apk\android-debug.apk
```

> If you get an error saying something about graddle, please visit [this StackOverflow question](http://stackoverflow.com/questions/29874564/ionic-build-android-error-when-download-gradle) which shows how to resolve it by installing graddle manually and changing the path to it in the `build.js` file.

To be able to test your application on your Android device, you have to enable the developer settings and you can do that in few steps:

+ Open the Settings view and scroll to `About Phone`
+ At the bottom of the About Phone view find a `Build Number` and tap on it  seven times to enable the developer mode.
+ After this you can go back to the `Settings` view and you'll see a new option called `Developer Options`

Now you have to enable the USB debugging, and you can do that in the next few steps:

+ Select `Developer Options` item in the `Settings` view
+ Scroll down until you see the USB debugging option
+ Toggle it ON
+ Now your device is set up for debugging, and when it's connected to your computer you'll be able to deploy the application to the device

Now connect your device and execute the following command in your Termina/Command prompt:

`adb devices`

You should see a list of devices that are attached to your computer (Genymotion emulators will also show up here), among which the one that you just attached.

Now, to install the application on your attached device first make sure that you're in the root directory of the project (Ionic_3rdTutorial in our case) and then run the following command from your Terminal/Command prompt: 

`adb –d install platforms\android\build\outputs\apk\android-debug.apk`

Find the application in your application list and open it. You should see the ads in the bottom and after the fifth calculation you should see an Interstitial ad appear.

> If for some reason the adb command wouldn't work for you, you can always just copy the generated .apk file manually to your connected device (for example in the Downloads folder). After that open the Downloads folder on your phone with the file manager application that you use and open the .apk file that you pasted. This way you'll be prompted to install the application on your phone and then you'll be able to find it in the programs list:
> ![](http://i.imgur.com/wyzbnys.jpg)

# Conclusion
In this tutorial, we showed you how to polish our existing calculator application by improving the design and user experience. Next, we showed how to automatically create icons and splash screen images. Then we covered how to make money with our application by using Google AdMob ads. Also, we showed how to share our application with other users without going through the app stores. Finally, we showed how to test our application on the real physical devices and emulators.

In the next installment of this tutorial we're going to show how to prepare the application for the Apple's App Store and Google's Play Store and finally how to publish the application to both Apple's App Store and Google's Play Store. Also, we'll create a small landing page of our application with the basic information and download instructions (links to the application in both stores).

If you have any questions about this tutorial, please read the following help guides. Of course, I encourage you to ask a question in the comments so that anyone else who may have a similar question or doubt, learns something too.

With this, I leave you and hope you had a great learning experience with following this tutorial. See you in the next installment of this series!

# How to get help with Ionic framework
Few of the resources that proved indispensable when I was learning about Ionic:

+ For a quick framework reference, I'm suggesting the [official documentation](http://ionicframework.com/docs/), which is indeed very good.
+ If you're in search for a good book about Ionic, then you don't need to look further than [Ionic in Action: Hybrid Mobile Apps with Ionic and AngularJS](http://amzn.to/1EjwYNJ)
+ One of the best resources on the net for programming related questions, about which you've no doubt heard, is [StackOverflow](http://stackoverflow.com/). You can view the specific Ionic tagged questions via [this link](http://stackoverflow.com/questions/tagged/ionic) or even [this one](http://stackoverflow.com/questions/tagged/ionic-framework). However, I urge you to please read their [help page](http://stackoverflow.com/help) before posting questions (especially if you're a new user to StackOverflow) because otherwise you may have a bad experience. If you're a new StackOverflow user, who's in a hurry to get acquainted with it, make sure you at least go through this [2-minute tour](http://stackoverflow.com/tour).
+ To get started with AngluarJS try [these](http://angular.codeschool.com/) [few](https://hackhands.com/finishing-Angular-TODO-application-deploying-production/) [resources](https://docs.angularjs.org/tutorial), not necessarily in that particular order.
+ Let me just kind of repeat myself by saying that if you have **any questions** about Ionic framework, or you had trouble following this tutorial (you couldn't install something), or you would like to suggest what kind of tutorials you would like to see regarding Ionic in the future, then please **share it in the comments** below. Also, you can reach me personally via @[HitmanHR](https://twitter.com/hitmanhr) or [my blog](http://www.nikola-breznjak.com/blog). I'll do my best to  answer all of your questions.
+ If, however, you favor "one on one" help, you can reach me [via HackHands](https://hackhands.com/dashboard/request/hitman666/preferred).
