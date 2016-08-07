With Angular 2 [being almost ready for release](https://splintercode.github.io/is-angular-2-ready/), it is definitely time to start experimenting with it. If you are out of ideas what to build after completing the main Angular 2 tutorial, this guide is an excellent supplemental material that will deepen your knowledge in the new framework without going into too advanced techniques.

We are going to build a Angular 2 app that loads data from Reddit's [/r/wallpapers](https://www.reddit.com/r/wallpapers/) thread. In the end, you'll have a wall of beutiful images that scroll infinitely. Sounds cool, doesnt it? On top of this, here is what you are going to learn:

- Setting up Angular 2 with Webpack
- Loading data from extrnal services
- Using components to transfer and render data


# Setting upAngular 2 with Webpack


To install Angular 2 and Webpack, you need [Node Package Manager (NPM)](https://www.npmjs.com/) and [Node.js](https://nodejs.org/en/). Any version of Node 4 and up and NPM 3 and up would suffice. Here is how you can check your versions in the terminal:

```bash
$ node --version
v6.2.0
$ npm --version
3.8.9
```

Start off by reating a directory for your application:

```bash
$ mkdir reddit-wallpaper-wall
```
Next, go to your project folder and create a <code>package.json</code> file by writing [npm init](https://docs.npmjs.com/cli/init) to manage the dependencies. Here's how you can do it through the terminal:
```bash
$ cd reddit-wallpaper-wall
$ npm init
```


**package.json**

Open up your <code>package.json</code> file and replace its contents with the following snippet:


```json
{
  "name": "reddit-wallpaper-wall",
  "version": "1.0.0",
  "description": "",
  "author": "",
  "repository": {
    "url": ""
  },
  "license": "MIT",
  "scripts": {
    "build": "webpack --progress",
    "build:prod": "webpack -p --progress",
    "serve": "webpack-dev-server --inline --progress",
    "postinstall": "typings install"
  },
  "devDependencies": {
    "css-loader": "^0.23.1",
    "extract-text-webpack-plugin": "1.0.1",
    "file-loader": "^0.9.0",
    "html-loader": "^0.4.3",
    "html-webpack-plugin": "2.20.0",
    "img-loader": "1.2.2",
    "less": "^2.7.1",
    "less-loader": "^2.2.3",
    "raw-loader": "0.5.1",
    "style-loader": "^0.13.1",
    "ts-loader": "0.8.2",
    "typescript": "1.8.10",
    "typings": "1.0.5",
    "url-loader": "^0.5.7",
    "webpack": "1.13.1",
    "webpack-dev-server": "1.14.1"
  },
  "dependencies": {
    "@angular/common": "2.0.0-rc.1",
    "@angular/compiler": "2.0.0-rc.1",
    "@angular/core": "2.0.0-rc.1",
    "@angular/http": "2.0.0-rc.1",
    "@angular/platform-browser": "2.0.0-rc.1",
    "@angular/platform-browser-dynamic": "2.0.0-rc.1",
    "@angular/router": "2.0.0-rc.1",
    "core-js": "2.4.0",
    "reflect-metadata": "0.1.3",
    "rxjs": "5.0.0-beta.6",
    "zone.js": "0.6.12"
  }
}
```
***
talk about deps?
***


**tsconfig.json**

[tsconfig.json](http://www.typescriptlang.org/docs/handbook/tsconfig-json.html) is a TypeScript configuration file that is used to declare the transpiler configuration for the <code>.ts</code> files.
```json
{
    "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }   
}
```

**typings.json**

 [typings.json](https://angular.io/docs/ts/latest/guide/typescript-configuration.html#!#typings)  another typescript configuration file. It is used to declare which extral libraries are going to read TypeScript files.
 In it paste the following contents:
 
 

```json
 {
  "globalDependencies": {
    "core-js": "registry:dt/core-js#0.0.0+20160602141332",
    "require": "registry:dt/require#2.1.20+20160316155526"
  },
  "dependencies": {
    "webpack": "registry:npm/webpack#1.12.9+20160418172948"
  }
}
```




Then, in the same directory, run:

```bash
$ npm install
```
And all the dependency will be installed. In case you run in any problems, try to install typescript, typings and webpack as global dependencies:

```bash
$ npm install --global webpack
$ npm install --global typings
# npm install --global typescript
```

**webpack.config.js**

The last configuration file to finish the setup is the Webpack configuration. 
```javascript
var webpack = require('webpack');
var htmlWebpackPlugin = require('html-webpack-plugin')
var ExtractTextPlugin = require('extract-text-webpack-plugin');

module.exports = {
    entry: './src/main.ts',
    output: {
        path: './dist',
        filename: 'app.bundle.js',
    },
    module: {
        loaders: [
            { test: /\.ts$/, loader: 'ts' },
            { test: /\.html$/, loader: 'html' },
            {
                test: /\.less$/,
                loader: ExtractTextPlugin.extract('style', 'css!less'),
                exclude: /node_modules/
            },
            {
                test: /\.(png|jpe?g|gif|svg|woff|woff2|ttf|eot|ico)$/,
                loader: 'file?name=assets/[name].[hash].[ext]'
            },
        ]
    },
    resolve: {
        extensions: ['', '.js', '.ts', '.less']
    },
    plugins: [
        new htmlWebpackPlugin({
            template: './src/index.html'
        }),
        new ExtractTextPlugin('styles.css')
    ]
};
```
[Webpack](https://webpack.github.io/) uses [loaders](https://webpack.github.io/docs/loaders.html) and [plugins](https://webpack.github.io/docs/using-plugins.html) as its main building blocks. They are used to load project depenendices and treat them as modules with a particular functionality. We utilize the Webpack loaders we listed in <code>package.json</code> to organize the different assets in the application.

Webpack's configuration for entry and exit will give you a hint about the next step:
```
//webpack.config.js
module.exports = {
    entry: './src/main.ts',
    ...
    }
```
 The next step is to make a directory for the project and to make a <code>main.ts</code> file for bootstrapping the Angular 2 application
## Initializng your Angular 2 application

With all the configuration done, we can start our project. In the root folder, create a new directory, <code> src </code>.  This is where the Angular 2 application will reside:

```bash
$ mkdir src
```

### Bootstrapping
 Next, create a file named <code>main.ts</code> that will bootstrap the application and its dependencies:
```ts
//main.ts
import 'core-js';
import 'reflect-metadata';
import 'zone.js/dist/zone';
import 'rxjs/Rx';


import { bootstrap } from '@angular/platform-browser-dynamic';
import { HTTP_PROVIDERS } from '@angular/http';

import { AppComponent } from './app/app.component';


bootstrap(AppComponent, [HTTP_PROVIDERS])
    .then(success => console.log(`Bootstrap success`))
    .catch(error => console.log(error));

```
There are a few things to note here: We import [RxJs](https://github.com/Reactive-Extensions/RxJS), whic whill let us use observables later on in the application. We are including <code>HTTP_PROVIDERS</code> form the Angular 2 http library so in order to construct HTTP requests to Reddit's API.

The last thing we import is the root component, <code>AppComponent</code>, that we will create in the next step.
Every web application has its <code>index.html</code>. Let's add ours:

```html
<!-- src/index.html -->
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf­8">
    <title>Wallpapers</title>
    <link href='https://fonts.googleapis.com/css?family=Open+Sans' rel='stylesheet' type='text/css'>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  </head>
  <body>
    <app>Loading...</app>
  </body>
</html>
```

In the index page, we simply add [Material Icons](https://fonts.googleapis.com/icon?family=Material+Icons) and the [Open Sans](https://fonts.googleapis.com/css?family=Open+Sans) font to make the app look better.
In the <code>body </code> section, we add the selector for the root component -  ```<app></app>``` .



### The first component

In the <code>src</code> directory, create another directory, named <code> app </code>:
```bash
$ mkdir app
```
Next, create <code>app.component.ts</code>:

```typescript
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'app',
    template: 'root  component',
   
})
export class AppComponent implements OnInit {
    constructor() { }

    ngOnInit() { }

}
```
We start off by implorting <code> Component </code> and <code> OnInit</code> from the Angular 2 core.
Next, we use the <code> @Component </code> decorator to define metadata for the component. In the code snippet above, we define the component's selector (```<app></app>```) and its template content(```'root component'```). If you'd like to learn more about component decorator properties, [this](https://angular.io/docs/ts/latest/guide/cheatsheet.html) cheatsheet provides a list of all <code>@Component</code> decorator properties.

In the code for the component itself, we use <code> implements  OnInit </code> and <code>ngOnInit() {} </code> to implement a  [lifecycle hook](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html#!#hooks-overview) that is executing when the compoenent loads. For now we'll leave it blank and use it when we're going have to load the data from Reddit.

### Running

 With the application's most essential components in place, you can now run it and preview it. To do so, run the following commands in your root directory:
 
```bash
 $ npm run build
 $ npm run serve
```
Go to [http://localhost:8080](http://localhost:8080) and you will see <code>'root component' </code> displayed in the upper left side of the window.

## Building a service

In this step, we will extract and model the JSON data from the Reddit API. Before we go ahead and start making HTTP requests, we have to do some data modelling.

### Modelling

To create a model, we need to think what kind of attributes it has. We need something to represent the list of all wallpapers and a wallpaper as a single entity. 

**Wallpaper**

Our wallpaper needs three attributes:
- A descriptive title
- An URL of the preview image
- An URL of the full-resolution image

**Listing of wallpapers**

The wallpaper listing contains the wallpapers themselves plus additional information about the listing as a whole. To get an idea how a wallpaper listing would look like, we need to visit the [Reddit API documentation](https://www.reddit.com/dev/api/) to get a feel how the information is structured. By reading it, we can infer that we can use the <code> before </code> and <code> after </code> to paginate through the results. So 




Therefore, the listing will contain

- An array of wallpapers
- An attribute denoting pagination/what part of the total collection is being viewed (<code> after </code>)




