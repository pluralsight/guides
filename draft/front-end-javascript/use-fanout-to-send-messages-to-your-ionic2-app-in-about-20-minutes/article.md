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


### **Let's Start Coding**

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


