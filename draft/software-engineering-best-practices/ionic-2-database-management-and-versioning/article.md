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

***

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

***

## Creating and opening the database

Now here's where the fun starts, first let's talk a bit of theory, the way angular 2 scopes works makes each component be isolated, that's a great thing since errors in one component will not affect the other, thus allowing to find and fix bugs easyer, however this also mean that if we want something to be available through the app, we need to use a provider (or a service, this days both are almost the same).

Next in the line is the database library selection, theres a few options here:

1. Local Storage (not good for critical or persistent data, can be erased by OS when running out of memory).
2. Cache Storage (better used with Service Workers, however that's an advanced use case and most likely not very useful in case of tabular data).
3. SqLite with WebSql fallback (our choice and the defacto for many production ready hybrid apps).

We will be using `SqLite` for this purpose with `WebSql` fallback when in developing.

**Note:** The hidden reason to use the `WebSql` fallback is to be able to deploy the app both as a mobile app and as a web app, some times the app isn't supported by the device so with this fallback, as good as the device have `Chrome` installed, they would be able to open the web version and use the app like that, also note i say `Chrome` because `Firefox` doesn't support `WebSql`, and other browsers lack `internationalization api` support, thus you'll need to use polyfills to bridge the gap between browsers.

### Creating the database provider

`Ionic CLI` comes with a handy utility called a [generator](http://ionicframework.com/docs/v2/cli/generate/) that allows for quick prototyping, in order to use it firs list the available generators and then choose the one you want, like this (note the shorthand of `generator` as `g`):

**Listing:**

```sh
ionic g --list
```

Which should output something like this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fc40b883-6892-4805-a038-41046b4f7794.png)

We need a provider, so we execute this:

```sh
ionic g provider DbProvider
```

Which generates the following folder structure in your app:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b02ca5fa-978f-47f3-955a-1d2bfb9aab3b.png)

Now let's delete unnecesary code in there to slim it down, now yours should look like this, that's the basic class we start from:

```ts
import { Injectable } from '@angular/core';

@Injectable()
export class DbProvider {
  constructor() {
  }
}
```

### Adding dependencies an, creating and opening the database

Now here's one of the first design decisions we'll make, we're going to use `SqLite` as the database technology and `WebSql` as fallback in order to support databases during development, as well as allow the app to be published as a web app as explained previously to support devices cordova won't, taking in account the device neet to support at least Chrome.

But that's not the decision we need to take, here's the deal, there's currently 2 ways to deal with `SqLite` databases in Ionic 2, one is through [Ionic native's SqLite adapter](http://ionicframework.com/docs/v2/native/sqlite/) and the other is through [Ionic's SqlStorage Api](http://ionicframework.com/docs/v2/api/platform/storage/SqlStorage), the diference between both is that the first has nice transaction methods bridged directly of cordova's SqLite plugin, while the other automathically uses `SqlStorage` in mobile and `WebSql` in web browsers.

It's hard to deal with the `Ionic native`'s implementation since it's database is 
