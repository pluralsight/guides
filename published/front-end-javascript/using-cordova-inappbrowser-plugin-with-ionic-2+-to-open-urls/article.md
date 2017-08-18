# Introduction 

This tutorial will cover how to use [Cordova's InAppBrowser plugin](https://cordova.apache.org/docs/en/latest/reference/cordova-plugin-inappbrowser/) with [Ionic Native 3.x+](http://blog.ionic.io/ionic-native-3-x/) to open external URLs in Ionic 2+ apps or use third party services which require opening a webview such as implementing social authentication and payment gateways.

Ionic Native is a set of wrappers around Cordova plugins. These wrappers allow Ionic developers to handle native plugins using observables instead of ugly callbacks.

# Requirements 

Before you can build Ionic 2+ apps you need to have a few requirements:

* Node.js and NPM (Node Package Manager).
* Java and Android SDKs for Android apps.
* Setup JAVA_HOME and ANDROID_HOME system variables for Android apps. 
* A MAC system and XCode for iOS apps.
* Windows for UWP apps.
* Ionic CLI and Cordova.

## Installation guide

For instructions on installing **Node.js** and *NPM**, head over to their [official website](https://nodejs.org/en/) and grab the installer for your operating system.

If you need to target the Android platform, first install **Java** by going to their [official website](https://java.com/en/download/) and follow the instructions. Then install the Android SDK available from this [link](https://developer.android.com/studio/index.html).

If you successfully installed Node.js and NPM the next step is to install Cordova and Ionic CLI.

### Installing Cordova and Ionic CLI

If you don't have Ionic CLI and Cordova installed, then you need to start by installing them first:

Go ahead, open your terminal or command prompt and enter:

    npm install -g cordova 
    npm install -g ionic 
    
These commands should install cordova and ionic globally on your development machine.

Now you should be able to generate a new Ionic project.

# Getting started

### Generating an Ionic 2+ Project 

Let's start by creating a new Ionic 2+ application. Open your terminal or command prompt then run the following commands:

    ionic start InAppBrowserDemo blank 

After generating a new Ionic project, navigate inside your project root folder. Add your target Cordova platform. In my case, I can only add Android since I'm working in Ubuntu. However, if you are working on OSX, you can target iOS too. 

Go ahead and run:

    ionic cordova platform add android 
    ionic cordova platform add ios
    
If you are using Windows, you can also target the Universal Windows Platform (UWP):

    ionic cordova platform add windows

Now after adding your target platform, you can install the InAppBrowser plugin.


#### What is InAppBrowser?

InAppBrowser is a native Cordova plugin which can be used to add an in-app browser to your hybrid mobile application created using any Cordova-based framework, such as Ionic.

You can find more information about InAppBrowser from this [Github repository](https://github.com/apache/cordova-plugin-inappbrowser).


### Installing InAppBrowser Cordova Plugin and The Corresponding Ionic Native Wrapper

InAppBrowser plugin is available from npm so to install it simply run this command inside your project root folder:

    ionic plugin add cordova-plugin-inappbrowser --save

    npm install --save @ionic-native/in-app-browser
    
    

Start by importing the InAppBrowser native plugin:

    import { InAppBrowser } from '@ionic-native/in-app-browser';
    
Then add it to the list of providers:
    

    @NgModule({
    declarations: [
        MyApp,
        HomePage 
    ],
    imports: [
        BrowserModule,
        IonicModule.forRoot(MyApp)
    ],
    bootstrap: [IonicApp],
    entryComponents: [
        MyApp,
        HomePage 
    ],
    providers: [
        StatusBar,
        SplashScreen,
        InAppBrowser,
        {provide: ErrorHandler, useClass: IonicErrorHandler}
    ]
    })
    export class AppModule {}  

You are now ready to use the InAppBrowser API via dependency injection. This API provides you with an instance of InAppBrowser that can be used to call different methods that create and open in-app browsers within your Ionic 2+ app.
    
So open <em>src/pages/home/home.ts</em> and add:



    import { Component , OnInit } from '@angular/core';
    import { NavController  } from 'ionic-angular';
    import { InAppBrowser } from '@ionic-native/in-app-browser';
    
    @Component({
    selector: 'page-home',
    templateUrl: 'home.html'
    })
    export class HomePage implements OnInit{
    
        constructor(public navCtrl: NavController,private iab: InAppBrowser) {
    
        }
    
        ngOnInit(){
    
            const browser = this.iab.create('https://www.techiediaries.com','_self',{location:'no'}); 
    
        }
    
    }    


Let's break down the code. First, we import InAppBrowser from @ionic-native/in-app-browser then inject it via component constructor. Then we call the `.create()` method to create and open an in-app browser which navigates to [techiediaries.com](https://www.techiediaries.com) when the component has been fully initialized.

You can find the complete documentation for InAppBrowser in [Ionic Native official website](https://ionicframework.com/docs/native/in-app-browser/).

## Running the App 

Now that you have created your simple example that uses InAppBrowser to open an external URL inside your mobile application it's time to run the app in an actual device:

Since I'm targeting Android, I will use an Android phone. However, Ionic runs on numerous platforms, as mentioned above. Feel free to use a device running on any of these supported platforms.

For Android, I'll use the following command to run my app:

    ionic run android 

For iOS use:
    
    ionic run ios 
    
For Windows use:

    ionic run windows
    
### Testing the app on a browser
    
One of the cool features of Ionic is that you can also test your application **in the browser** using the serve command:

    ionic serve 

Then, in a browser of your choice, visit [http:localhost:8200](http:localhost:8200).


## Conclusion 

This guide covered using the InAppBrowser Cordova plugin with Ionic 2+ and Ionic Native to add an in-app browser that can open external URLs, add third party authentication to your app or implement services that need to connect to external gateways. As seen by the quick solution presented in this tutorial, using the plugin and the Ionic framework streamlines a process that would otherwise be time-intensive.

Thanks for reading this tutorial! I hope you found it both informative and interesting. Please drop your feedback in the discussion section below. I look forward to seeing your apps (and in-app browsers) on an app store soon!
    




