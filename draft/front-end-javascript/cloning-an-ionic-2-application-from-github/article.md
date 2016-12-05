### In this tutorial I will explain how to clone an Ionic 2 project and get it running on a new machine.

When you first create an Ionic 2 application it creates a .gitignore file that ignores almost all folders except for src and resources. It leaves the basics to build the project, but if you don't know how to run them your porject will be left with a lot of errors and you will not be able to run the application. Here is what most projects will remain with after uploading to Github or another git repository. 


![This is an image of the structure that will be on Gihub after commiting](https://raw.githubusercontent.com/pluralsight/guides/master/images/faad4a92-5f68-4775-998d-6d90ced89624.png)


### Now to start the cloning process on a new machine. 

*All of the following is assuming you have the current versions of Ionic 2, Cordova and Node/NPM installed on the machine you are working on. If not please do so before preceding.*

So you have cloned your repository to your machine through whatever means you do that through and now you have a project that won't work. So there is a procedure you need to run before your project will be a complete project again. Now once you do this on the new machine you will be fine unless you add another plug-in or platform. If you do it will be added to the package.json file and you will have to run the following commands again in on the other machince to update the application. 

#### The steps

1. Now either open your application in your favorite IDE that has a terminal window or CD into the new cloned project directory through terminal.

2. Next you will need to get all your node_modules back into your application. All these modules are based on your package.json file. You may want to look at your file if you are getting errors and if you see a ^ before any of the versions like, "typescript": "^2.0.6", remove the ^ and that should fix the error. Here is the command to run in terminal. 
````` 
npm install
`````

3. Once that is complete you should notice that you now have a node_modules directory in your project structure. Now you will need to serve the app, run this command in the terminal.
`````
ionic serve
`````
This will create your www folder and your build folder needed to treat the project like an Ionic or Cordova project. 

4. Now you are ready to restore the state of your project. You need to run 
`````
ionic state restore
`````
in terminal and this will restore the platforms and all the plugins you had installed based again on your package.json file. Now you should see all the platforms and plugins that you had previously installed now in your project folder. 


#### Conclusion

Now your project should be restored to the original creation state. Remember, you will need Ionic, Cordova, Node/NPM installed on the computer you are cloning to in order to run this project. 

If you don't want to have to re-intsall all your plugins and platforms you could remove the directories from the .gitignore file and they would be included in your git commit. 


Happy Coding