In this tutorial, I will go over how to clone an Ionic 2 project from Github and get it running on a new machine.

When you first create an Ionic 2 application, it creates a .gitignore file that ignores almost all folders except the src and resources directories. This leaves the basics to build the project. However, if you don't know how to run these files, your project will contain numerous errors and you will be unable to proceed.

THe pictures show the contents of most projects after uploading to Github or to another git repository. 

![This is an image of the structure that will be on Gihub after commiting](https://raw.githubusercontent.com/pluralsight/guides/master/images/faad4a92-5f68-4775-998d-6d90ced89624.png)


### Start the cloning process on a new machine

#### Setup

*For this tutorial, you need to have set up the current versions of Ionic 2, Cordova and Node/NPM on the machine you are using.* In case these tools aren't ready, please see the links below:
- [Ionic 2](http://www.gajotres.net/ionic-2-installation-guide/)
- [Cordova](https://cordova.apache.org/)
- [NPM](https://docs.npmjs.com/cli/install)

If you don't want to have to re-install all your plugins and platforms you could remove the directories from the .gitignore file. That way, they would be included when you git commit. 

Let's say that you have cloned your repository to your machine, and now you have a project that does not work. There is a procedure you need to run to finish your project off. After this procedure, you will be fine, unless you add another plug-in or platform, at which point it will be added to the `package.json` file, and you will have to run the following commands again on the other machine to update the application. 

#### The steps

1. In CMD or Terminal, run `git clone <your-repository-address-here>` in the folder in which you'd like to contain the project.

2. Now either open your application in your favorite IDE (that has a terminal window) or `cd` into the new cloned project directory through terminal.

3. Next you will need to get all your `node_modules` back into your application. All these modules are based on your `package.json` file. You may want to look at your file if you are getting errors and if you see a ^ before any of the versions like, `typescript: ^2.0.6`, remove it to fix the error. Here is the command to run in terminal. 
````` 
npm install
`````

4. Once that is complete you should notice that you now have a node_modules directory in your project structure. Now you will need to serve the app. Run this command in the terminal.
`````
ionic serve
`````
This will create your `www` folder and your `build` folder which are necessary for a project to be treated like an Ionic or Cordova project. 

5. Now you are ready to restore the state of your project. Use the following command: 
`````
ionic state restore
`````
This will restore the platforms and all the plugins you had installed based on your `package.json` file. Now you should see all the platforms and plugins that you had previously installed now in your project folder. 

#### Conclusion

Now your project should be restored to the original creation state. Remember, you will need Ionic, Cordova, and Node/NPM installed on the computer you are cloning to in order to run this project. 
___

Hopefully this guide erases some of the confusion behind cloning Ionic 2 apps via Github. Thanks for reading. Please leave all comments and feedback below. 

Happy Coding.