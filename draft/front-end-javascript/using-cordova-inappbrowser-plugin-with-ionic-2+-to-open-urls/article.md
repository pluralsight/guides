## Introduction 

In this tutorial we are going to see how to use the InAppBrowser Cordova plugin to open external URLs in Ionic 2+ apps or use third party services which require opening a webview such as implementing social authentication and payment gateways. So lets get started.

Ionic Native is a set of wrappers around Cordova plugins which allow Ionic developers to use the native plugins in an Angular way i.e using observables instead of ugly callbacks.

## Requirements 

Before you can build Ionic 2+ apps you need to have a few requirements:

* Node.js and NPM.
* Java and Android SDKs for Android apps.
* Setup JAVA_HOME and ANDROID_HOME system variables. 
* A MAC system and XCode for iOS apps.
* Windows for UWP apps.
* Ionic CLI and Cordova.

For installing Node.js and NPM you can head over to their [official website]() and grab the installer for your operating system.

If you need to target Android platform first install Java by going to their [official website]() and follow the instructions. Then install the Android SDK available from this [link]().

If you successfully installed Node.js and NPM the next step is to install Cordova and Ionic CLI.

## Installing Cordova and Ionic CLI

If you don't have Ionic CLI and Cordova installed, then you need to start by installing them first:

Go ahead, open your terminal or command prompt and enter:

    npm install -g cordova 
    npm install -g ionic 
    
These commands should install cordova and ionic globally on your development machine.

Now you should be able to generate a new Ionic project.

## Gnerating an Ionic 2+ Project 

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

You can find more information about InAppBrowser from this [Github repository]().
## Installing Cordova and Ionic CLI


## Installing InAppBrowser Cordova Plugin and Corresponding Ionic Native Wrapper

InAppBrowser plugin is available from npm so to install it simply run this command inside your project root folder:
    
    
    ionic plugin add cordova-plugin-inappbrowser --save

    npm install --save @ionic-native/in-app-browser
    
    

Start by importing the InAppBrowser native plugin :

    import { InAppBrowser } from '@ionic-native/in-app-browser';
    Then add it to the list of providers :
    
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

    
Open src/pages/home/home.ts and add

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


ionic run android 





