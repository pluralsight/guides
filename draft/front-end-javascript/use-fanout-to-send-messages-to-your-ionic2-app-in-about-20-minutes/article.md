## **Fanout**
[Fanout](https://fanout.io/) is a great service that allows you to quickly and easily send messages to clients applications. In this tutorial we will use Fanout to send messages to an Ionic2 application.

## **Getting Set Up**

Ok, first things first - go to the [Fanout](https://fanout.io/) site and create your **FREE** account - to ensure that is it free make sure you click on the **"Hacker"** type of account. Next, while you are log into the the site nagivate to the [account page](https://fanout.io/account/panel/) and make a note of both your Reaml ID and your Realm Key - we will need both of these for wiring up the Ionic2 and Nodejs pub-sub mechanism. 

Next, to create an Ionic2 app that receive messages and the Nodejs app that publishes messages, you need to install Nodejs. Download and install it [here](https://nodejs.org/). 

Once the installation is complete, open a command prompt and execute:

```
npm install -g ionic@beta
```
This command uses the node package manager to install the Ionic2 node package globally. That packate allows us to create Ionic2 applications. 

If you run this:
```
Ionic -v
```
you should see something like this which means the package installed successfully and you are good to go (don't worry if your version numbers may be different from mine):

```
2.0.0-beta.32
```
If you get an error, please go back through the steps above or follow the more detailed installation steps on the [Ionic2 installation page](http://ionicframework.com/docs/v2/getting-started/installation/).

## **Creating the Base Application**

Ok, if you are in impatient type all of the source code is available in Github [here](https://github.com/sethbunke/Ionic2NodejsFanoutTutorial). 

To create the base Ionic2 application, go to the command line and execute:
```
ionic start FanoutIonicNodeTutorial blank --v2 --ts
```
With that command we are creating a "blank" Ionic version 2 application named **FanoutIonicNodeTutorial** that uses TypeScript. Don't worry if you aren't familiar with TypeScript - we only have to get our hands a little dirty with it for this tutorial. 

Once the installation is complete, change your directory to the folder that was created for your app and start running your app:
```
cd FanoutIonicNodeTutorial
ionic serve

```

## **Adding Our Code**
To get our app receiving messages from [Fanout](https://fanout.io/) and displayed on the screen we will need to create three "providers" in our application. Don't be put off by the name "provider" in Ionic2 and Angular2 as they are really just another name for what are referred to as "services" in other frameworks - objects that encasulate a specific set of functionality in order to have better separation of concerns and avoid duplication of code in an application.

For our app, those 3 providers are for alerts, configuration, and Fanout. We can add them easily using the ionic "generator" command, like this:
```
ionic generate provider AlertProvider
ionic generate provider ConfigurationProvider
ionic generate provider FanoutProvider

```
With our providers created we will need to modify them to meet our needs. I use [VS Code](https://code.visualstudio.com) for developing Ionic2 applications due to its good support for TypeScript, but you can use whatever editor you feel most comfortable with. 

In your favorite editor, open up the **alert-provider.ts** file in the **/app/providers/alert-provider** folder. Replace the contents of that file with this:
```
import { Injectable } from '@angular/core';
import {NavController,Alert} from 'ionic-angular';

@Injectable()
export class AlertProvider {  
  constructor() {}  
  
  presentDismissAlert(title: string, subtitle: string, navCtrl: NavController) {    
    let alert = Alert.create({
      title: title,
      subTitle: subtitle,
      buttons: ['Dismiss']
    });
    navCtrl.present(alert);  
  }  
}
```
Our alert provider will allow us to easily display alert messages on the screen to the user when messages are received from [Fanout](https://fanout.io/).

Next, open the **configuration-provider.ts** file in the **/app/providers/configuration-provider** folder and replace that file's contents with:
```
import { Injectable } from '@angular/core';

@Injectable()
export class ConfigurationProvider {   
  public fanoutUrl: string;
  
  constructor() {
    this.fanoutUrl = 'http://your-realm-id-here.fanoutcdn.com/bayeux';
  }  
}
```
>Remember the "Realm ID" from the Fanout account screen? Replace **"your-realm-id-here"** with your actual Fanout Realm ID. ***This is crucial - if your don't replace this with your correct realm ID your app will never be able to connect to the proper Fanout endpoint.***

Now open the **fanout-provider.ts** file in the **/app/providers/fanout-provider** folder and replace the contents with:
```
import { Injectable } from '@angular/core';
import { Http } from '@angular/http';
import 'rxjs/add/operator/map';
import { ConfigurationProvider } from '../configuration-provider/configuration-provider';

@Injectable()
export class FanoutProvider {
  data: any;
  client: any;
  clientUrl: string;

  constructor(private http: Http, private configurationProvider: ConfigurationProvider) {
    this.data = null;
    this.clientUrl = configurationProvider.fanoutUrl;
    this.client = new Faye.Client(this.clientUrl);
  }

  subscribe(callback:any, channelName:string) {      
      this.client.subscribe('/' + channelName, function (data) {          
          callback(data);
          console.log('received data: ' + data);
      });
  }  
}
```
This file is a bit more complicated as this is where our application actually connects to the [Fanout](https://fanout.io/) service. This provider uses the ConfigurationProvider to get the URL that is used to connect to Fanout. However, in order to for different parts of our application to able to access the providers we created, we need to regisgter those providers the the framework. 

To do so, navigate to the file **app.ts** in the **/app** folder. Update the contents of that file to this:
```
import {Component} from '@angular/core';
import {Platform, ionicBootstrap} from 'ionic-angular';
import {StatusBar} from 'ionic-native';
import {HomePage} from './pages/home/home';
import {ConfigurationProvider} from './providers/configuration-provider/configuration-provider';
import {AlertProvider} from './providers/alert-provider/alert-provider';

@Component({
  template: '<ion-nav [root]="rootPage"></ion-nav>',
  providers: [ConfigurationProvider,AlertProvider]
})
export class MyApp {
  rootPage: any = HomePage;  

  constructor(platform: Platform) {
    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      StatusBar.styleDefault();
    });
  }
}

ionicBootstrap(MyApp);
```
Specifically, what we are doing here is showing the framework where the providers are located, the names of the providers that we want loaded, and then registering them as providers so that they are accessible throughout the framework? 

***Why isn't the FanoutProvider also getting registered here?*** That's because we only want to register providers globally that are likely to be used in many different parts of the application - the alert and configuration providers are good examples widely used providers. The FanoutProvider on the other hand may only be used in a few places so we will only register it where it is needed - such as in our Home component which is what we will work on next. 

## **The Home Component/Page**


## **One Last Thing to Take Care Of**
