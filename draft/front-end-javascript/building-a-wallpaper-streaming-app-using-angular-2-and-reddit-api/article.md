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
// src/app/app.component.ts
import { Component, OnInit } from '@angular/core';

@Component({
    selector: 'app',
    template: 'root component',
   
})
export class AppComponent {
    constructor() { }



}
```
We start off by implorting <code> Component </code> and <code> OnInit</code> from the Angular 2 core.
Next, we use the <code> @Component </code> decorator to define metadata for the component. In the code snippet above, we define the component's selector (```<app></app>```) and its template content(```'root component'``` .). If you'd like to learn more about component decorator properties, [this](https://angular.io/docs/ts/latest/guide/cheatsheet.html) cheatsheet provides a list of all <code>@Component</code> decorator properties.



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

To create a model, we need to think what kind of attributes it has. We need something to represent the list of all wallpapers and a wallpaper as a single entity. Here is how we can think about the different models:

**Wallpaper**

Our wallpaper needs three attributes:
- A descriptive title
- An URL of the preview image
- An URL of the full-resolution image

Here is how this would look in TypeScript:

```typescript
// src/app/wallpaper.model.ts
export class Wallpaper {
    url: string;
    previewUrl: string;
    title: string;
}
```

**Listing of wallpapers**

The wallpaper listing contains the wallpapers themselves plus additional information about the listing as a whole. To get an idea how a wallpaper listing would look like, we need to visit the [Reddit API documentation](https://www.reddit.com/dev/api/) to get a feel how the information is structured. By reading it, we can infer that we can use the <code> before </code> and <code> after </code> to paginate through the results. So 


Therefore, the listing will contain

- An array of wallpapers
- An attribute denoting pagination/what part of the total collection is being viewed (<code> after </code>)


```typescript
// src/app/wallpaper-listing.model.ts
import { Wallpaper } from './wallpaper.model';//Don't forget import the Wallpaper model

export class WallpaperListing {
    wallpapers: Array<Wallpaper>;//array of wallpapers
    after: string;
}
```
### GETing data

The data models are figured out and we are going to put them to use now. We'll do this by creating an injectable service that will parse the data and assign it to our models.

```typescript
// src/app/reddit.service.ts

import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';

import { Observable }     from 'rxjs/Observable';//using the RxJS observables
import { Wallpaper } from './wallpaper.model'; //improting the wallpaper model
import { WallpaperListing } from './wallpaper-listing.model';//importing the wallpaper listing model

@Injectable()
export class RedditService {
    
    constructor(private http: Http) {
    }

}

```

We make the <code>RedditService</code> class injectable by adding <code> @Injectable() </code> before its definition. In order to make the Http module available inside the class, we have to  inject it in it via the <code> constructor </code> function.

Next, we add the <code> getWallpapers() </code> function():

```
// src/app/reddit.service.ts

export class RedditService {


    //
    getWallpapers(after: string) : Observable<WallpaperListing> {
        let path = '//www.reddit.com/r/wallpapers.json?raw_json=1';

        // continues from last item loaded
        if(after) path += '&after=' + after;

        return this.http
            .get(path)
            .map(this.mapWallpapers);
    }

    mapWallpapers(res: Response) {
        let body = res.json();
        let listing = new WallpaperListing();

        //iterate through the data
        return listing;
    }
    
    //
    
    }
```

The <code> getWallpapers </code> function accepts one parameter (<code> after</code> as string) . The second part is its ```: Observable<WallpaperListing> ```. It denotes what type of variable is expected to be returned when the function <code>return</code>s. In it, we use Angular 2's http library to get a response from [https//www.reddit.com/r/wallpapers.json?raw_json=1](https//www.reddit.com/r/wallpapers.json?raw_json=1) and map it through the <code>mapWallpapers</code> function.


<code> mapWallpapers </code> gets the response data. Let's have a look at it [here](//www.reddit.com/r/wallpapers.json?raw_json=1). It might seem like a lot of information, but we only need a small chunk of it. We can notice that the post items are contained in the <code>children</code> attribute, which is an array. Let's  iterate through them!
We also need to visualize only posts that are images, so while iterating, we'll be chosing only posts who have their <code>post_hint</code> attribute set to <code> "image" </code>. Here is how the code looks like:

```ts
    // src/app/reddit.service.ts
    
    mapWallpapers(res: Response) {
        let body = res.json();
        let listing = new WallpaperListing();
        
        //iterate through the data
        let wallpapers = new Array<Wallpaper>();
        
        body.data.children.forEach(post => {
            if (post.data.post_hint === 'image') {
                let item = new Wallpaper();

                item.url = post.data.url;
                item.title = post.data.title;

                let previewImages = post.data.preview.images;
                let resolutions = post.data.preview.images[0].resolutions;

                let previewImage = resolutions.filter(m => m.width === 960)[0];               

                item.previewUrl = previewImage ? previewImage.url : item.url;

                wallpapers.push(item);
            }
        });
            
        listing.wallpapers = wallpapers;
        listing.after = body.data.after;

        return listing;

      ;
    }

```

 In the <code>forEach</code> loop, we also [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) the preview images to chose the ones which are 960 pixels. I wide. The ternary operator checks if such preview image is found. if not, the url is replaced with the full-resolution url:
```ts
  let previewImage = resolutions.filter(m => m.width === 960)[0]; 
  item.previewUrl = previewImage ? previewImage.url : item.url;
```
 
 In the end of the function, we attach the pagination attribute from the JSON data (  <code> after </code>)  to the <code>WallPaperListing</code> model and return the results.
 
 Here is the complete code for the two functions. [link to the whole service]():
```ts
    getWallpapers(after:string):Observable<WallpaperListing> {
        let path = '//www.reddit.com/r/wallpapers.json?raw_json=1';

        // continues from last item loaded
        if (after) path += '&after=' + after;

        return this.http
            .get(path)
            .map(this.mapWallpapers);
    }

    mapWallpapers(res:Response) {
        let body = res.json();
        let listing = new WallpaperListing();

        //iterate through the data
        let wallpapers = new Array<Wallpaper>();

        body.data.children.forEach(post => {
            if (post.data.post_hint === 'image') {
                let item = new Wallpaper();

                item.url = post.data.url;
                item.title = post.data.title;

                let previewImages = post.data.preview.images;
                let resolutions = post.data.preview.images[0].resolutions;

                let previewImage = resolutions.filter(m => m.width === 960)[0];

                item.previewUrl = previewImage ? previewImage.url : item.url;

                wallpapers.push(item);
            }
        });

            listing.wallpapers = wallpapers;
            listing.after = body.data.after;

            return listing;
    }
}
```

## Building the components

Normally, we'd continue building the rest of the logic in the <code>AppComponent</code>, but that would be a bad practice. Instead, we'll make a <code>WallpaperListing</code> component, which will represent the list of wallpapers:


**wallpaperlisting.component.ts**
```typescript
import { Component, OnInit } from '@angular/core';
import { RedditService } from './reddit.service';
import { WallpaperListing } from './wallpaper-listing.model';


@Component({
    selector: 'wallpaper-listing',
    template: require('./main.component.html'),
    providers: [RedditService],
    directives: []
})

export class WallpaperListingComponent implements OnInit  {

    private wallpaperListing: WallpaperListing;
    private error: boolean;

    constructor(private service: RedditService) { }

    ngOnInit() {
        this.loadWallpapers(null);
    }


    loadWallpapers(after:string) {
        this.wallpaperListing = new WallpaperListing();
        this.service
            .getWallpapers(after)
            .subscribe(result => {

                    if(this.wallpaperListing.wallpapers === undefined) {
                        this.wallpaperListing = result
                    } else {
                        this.wallpaperListing.wallpapers = this.wallpaperListing.wallpapers.concat(result.wallpapers);
                    }

                }, error => this.error = true
            );
    }

}

```

In order to mkae server available, we put <code>RedditService</code> as a provider in the component decorator, and we inject it in the constructor of the component to make it available inside.
   
    constructor(private service: RedditService) { }

We use <code> implements  OnInit </code> and <code>ngOnInit() {} </code> to implement a  [lifecycle hook](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html#!#hooks-overview) that is executing when the compoenent loads. For now we'll leave it blank and use it when we're going have to load the data from Reddit.

 The <code>loadWallpapers</code> function is used to fill up the local <code>wallpaperListing</code> class with data. We pass <code>null</code> to the function to indicate that <code>wallpaperListing</code> is being filled with data for the first imte.


### Updating  AppComponent

In order for the <code>WallpaperListingComponent</code> to work, we must:

- Import it in AppComponent
- Add it as a directive in the AppComponent decorator
- Put its selector in the template



```ts
// src/app/app.component.ts
import { Component } from '@angular/core';
import {WallpaperListingComponent} from './wallpaper-listing.component' //importing the component 
@Component({
    selector: 'app',
    template: '<wallpaper-listing></wallpaperListing>',//adding its selector to the template
    directives: [ WallpaperListingComponent ] //adding the component as a directive
})
export class AppComponent {
    constructor() { }
    
}
```