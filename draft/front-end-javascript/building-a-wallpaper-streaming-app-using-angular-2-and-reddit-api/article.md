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
    resolveLoader: {
        alias: {
            'component-style-loader': require.resolve('./component-style-loader')
        }
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
### The first component

### Bootstrapping
```ts
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