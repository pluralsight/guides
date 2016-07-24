# Ionic 2 database management and versioning

![Ionic 2 + Angular 2](http://ionicframework.com/img/blog/ionic-angular-v2.jpg)

Hybrid mobile apps have been growing a lot lately since the release of `Ionic 2` which is built with `Angular 2`, however there's not a lot of standards on how a hybrid app should be structured in Ionic 2.

In this guide we will learn how to deal with an essential part of the app, `databases`, it may not be a need in your app, but if you choose the path of `offline first` for mobile app (and is hard not to since a mobile hardly get a good signal when used out in the streets or public transports) you'll have to rely on a local database to store data.

## Guide structure

This guide will teach you the following treats of database management in Ionic 2:

1. Setting up the guide environment.
2. Creating and opening the database.
2. Query the database.
3. Raw queries for getting insertions ids.
4. Database versioning.
5. Database `online` backup.
6. Database reset.

With that we should start with the first point.

## Setting up the guide environment.

The first thing to do is creating a new ionic 2 app, if you are adding this feature to an existing app jump to the next section.

### Prerequisits

First we need to have `Node` and `NPM` installed, since that's OS specific and there's a lot of tutorials out there for that matter i'll leave it as the first homework of the guide.

Next let's install Ionic 2 with npm, open a console and type the following commands which will install both `cordova` and `ionic@beta` which is the current version of ionic 2 since it's still in beta as of 24/06/2016.

```sh
npm i -g cordova ionic@beta
```

### Creating new app

After ionic cli is installed we need to create a new app, for this purpose we will use the side menu starter repo, paste this in your terminal:

```sh
ionic start ionicDBManagerGuide sidemenu --v2 --ts && cd ionicDBManagerGuide && ionic serve
```

That will start an Ionic 2 `typescript` app using the side menu starter, and after it finishes it will open the folder and start the app, your browser will open and show you your new app.

You'll probably see a lot of output in the console from ionic about downloads, installing modules, and building your app before showing it in the browser, the whole process could take between 1 - 5 min depending on your ethernet bandwith and your pc `cpu / ram / hd` speed, at the end your browser should show something similar to this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/136c91ef-0b9a-4b18-a39d-d2476d4096e5.png)

**Note 1:** In case you're not very skilled with the terminal, using `&&` between commands ensure all of them are run in serial way, and if one fails the next ones doesn't execute.

**Note 2:** Also note this command is made to work in unix based terminals, so if you're in a diferent OS like `Windows` you'll need to edit that command accordingly.



