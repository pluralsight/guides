## Introduction 

In this tutorial we are going to see how to use the InAppBrowser Cordova plugin with Ionic Native 3.x+ to open external URLs in Ionic 2+ apps or use third party services which require opening a webview such as implementing social authentication and payment gateways.

Ionic Native is a set of wrappers around Cordova plugins which allow Ionic developers to use the native plugins in an Angular way i.e using observables instead of ugly callbacks.

## Requirements 

Before you can build Ionic 2+ apps you need to have a few requirements:

* Node.js and NPM.
* Java and Android SDKs for Android apps.
* Setup JAVA_HOME and ANDROID_HOME system variables for Android apps. 
* A MAC system and XCode for iOS apps.
* Windows for UWP apps.
* Ionic CLI and Cordova.

For installing Node.js and NPM you can head over to their [official website](https://nodejs.org/en/) and grab the installer for your operating system.

If you need to target Android platform first install Java by going to their [official website](https://java.com/en/download/) and follow the instructions. Then install the Android SDK available from this [link](https://developer.android.com/studio/index.html).

If you successfully installed Node.js and NPM the next step is to install Cordova and Ionic CLI.

## Installing Cordova and Ionic CLI

If you don't have Ionic CLI and Cordova installed, then you need to start by installing them first:

Go ahead, open your terminal or command prompt and enter:

    npm install -g cordova 
    npm install -g ionic 
    
These commands should install cordova and ionic globally on your development machine.

Now you should be able to generate a new Ionic project.

## Generating an Ionic 2+ Project 

Lets start by creating a new Ionic 2+ application. Open your terminal or command prompt then run the following commands:

    ionic start InAppBrowserDemo blank 

After generating a new Ionic project,navigate inside your project root folder then add your target Cordova platform. In my case i can only add Android since i'm working in Ubuntu but if you are working in a MAC system you can target iOS too. So go ahead and run:

    ionic cordova platform add android 
    ionic cordova platform add ios
    
P.S if you are under Windows, you can also target the Universal Windows Platform (UWP):

    ionic cordova platform add windows

Now after adding your target platform, you can install the InAppBrowser plugin.


## What is InAppBrowser?

InAppBrowser is a native Cordova plugin which can be used to add an in-app browser to your hybrid mobile application created with Cordova framework or any Cordova based framework such as Ionic.

You can find more information about InAppBrowser from this [Github repository](https://github.com/apache/cordova-plugin-inappbrowser).


## Installing InAppBrowser Cordova Plugin and The Corresponding Ionic Native Wrapper

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

You are now ready to use the InAppBrowser API via dependency injection which provides you with an instance of InAppBrowser that can be used to call diffrent methods to create and open in app browsers inside your Ionic 2+ app.
    
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


So we first import InAppBrowser from @ionic-native/in-app-browser then inject it via component constructor.

Then we call the .create() method to create and open an in app browser which navigates to [techiediaries.com](https://www.techiediaries.com) website when the component OnInit hook is called i.e when the component is completely initialized.

You can find the complete documentation for InAppBrowser in [Ionic Native official website](https://ionicframework.com/docs/native/in-app-browser/).

## Running the App 

Now that you have created your simple example that uses InAppBrowser to open an external URL inside your mobile application it's time to run the app in an actual device:

Since i'm targetting Android i will use an Android phone but feel free to use any supported platform and use the corresponding command to run your application.

    ionic run android 

For iOS use:
    
    ionic run ios 
    
For Windows use:

    ionic run windows
    
Please note that you can also test your application in the browser using the serve command:

    ionic serve 

Then, using your browser, visit [http:localhost:8200](http:localhost:8200).


## Conclusion 

So we have seen how to use the InAppBrowser Cordova plugin with Ionic 2+ and Ionic Native to add an in app browser which can be used for different things such as opening external URLs, adding third party authentication to your app or implementing any services which need to connect to external gateways etc. 
    




