[Fanout](https://fanout.io/) is a great service that allows you to quickly and easily send messages to clients applications. In this tutorial we will use Fanout to send messages to an Ionic2 application.

### **Getting Set Up**

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

### **Creating the Base Application**

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

### **Adding Our Code**
To get our app receiving messages from [Fanout](https://fanout.io/) and displayed on the screen we will need to create three "providers" in our application. Don't be put off by the name "provider" in Ionic2 and Angular2 as they are really just another name for what are referred to as "services" in other frameworks - objects that encasulate a specific set of functionality in order to have better separation of concerns and avoid duplication of code in an application.

For our app, those 3 providers are for alerts, configuration, and Fanout. We can add them easily using the ionic "generator" command, like this:
```
ionic generate provider AlertProvider
ionic generate provider ConfigurationProvider
ionic generate provider FanoutProvider

```
With our providers created we will need to modify them to meet our needs. I use [VS Code](https://code.visualstudio.com) for developing Ionic2 applications due to its good support for TypeScript, but you can use whatever editor you feel most comfortable with. 

In your favorite editor, open up the alert-provider.ts file in the /app/providers/alert-provider folder. Replace the contents of that file with this:
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

Next, open the configuration-provider.ts file in the /app/providers/configuration-provider folder and replace that file's contents with:
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

