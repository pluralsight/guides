With Angular 2 [being almost ready for release](https://splintercode.github.io/is-angular-2-ready/), it is definitely time to start experimenting with it. If you are out of ideas for what to build after completing the main Angular 2 tutorial, this guide is an excellent supplemental material that will deepen your knowledge in the new framework without going into too advanced techniques.

In this guide, you are going to build an Angular 2 app that loads data from Reddit's [/r/wallpapers](https://www.reddit.com/r/wallpapers/) community. In the end, you'll have a wall of beautiful images that scrolls infinitely. Sounds cool, doesn't it? On top of this, here is what you are going to learn:

- Setting up Angular 2 with Webpack
- Loading data from external services
- Using components to transfer and render data
- Using external components


# Setting up 


To install Angular 2 and Webpack, you need [Node Package Manager (NPM)](https://www.npmjs.com/) and [Node.js](https://nodejs.org/en/). Any version of Node 4 and up and NPM 3 and up would suffice. Here is how you can check your versions in the terminal:

```bash
$ node --version
v6.2.0
$ npm --version
3.8.9
```

Start off by creating a directory for your application:

```bash
$ mkdir reddit-wallpaper-wall
```
Next, go to your project folder and create a <code>package.json</code> file by writing [npm init](https://docs.npmjs.com/cli/init) to manage the dependencies. Here's how you can do it through the terminal:
```bash
$ cd reddit-wallpaper-wall
$ npm init
```


**package.json**

Open your <code>package.json</code> file and replace its contents with the following snippet:


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


Do the same for the rest of the configuration files:

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

 [typings.json](https://angular.io/docs/ts/latest/guide/typescript-configuration.html#!#typings) another typescript configuration file. It is used to declare which external libraries are going to read TypeScript files.
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
And all of the dependencies will be installed. In case you run into any problems, try to install TypeScript, typings and Webpack as global dependencies:

```bash
$ npm install --global webpack
$ npm install --global typings
$ npm install --global typescript
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
[Webpack](https://webpack.github.io/) uses [loaders](https://webpack.github.io/docs/loaders.html) and [plugins](https://webpack.github.io/docs/using-plugins.html) as its main building blocks. They are used to load project dependencies and treats them as modules with a particular functionality. We utilize the Webpack loaders we listed in <code>package.json</code> to organize the different assets in the application.

Webpack's configuration for entry and exit will give you a hint about the next step:
```
//webpack.config.js
module.exports = {
    entry: './src/main.ts',
    ...
    }
```
 That's right - we need to make a directory for the project and to make a <code>main.ts</code> file in it for bootstrapping the Angular 2 application
## Initializng your Angular 2 application

In the root folder, create a new directory, <code> src </code>.  This is where the Angular 2 application will reside:

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
There are a few things to note here: We import [RxJs](https://github.com/Reactive-Extensions/RxJS), which will let us use observables later on in the application. We are including <code>HTTP_PROVIDERS</code> from the Angular 2 HTTP library in order to construct HTTP requests to Reddit's API.

The last thing we import is the root component, <code>AppComponent</code>, that we are going create in the next step.


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

In the index page, we simply add [Material Icons](https://fonts.googleapis.com/icon?family=Material+Icons) and the [Open Sans](https://fonts.googleapis.com/css?family=Open+Sans) font to add extra style to our app.
In the <code>body</code> section, we add the selector for the root component -  ```<app></app>``` .



### The first component

In the <code>src</code> directory, create another directory, named <code>app</code>:
```bash
$ mkdir app
```
Next, create the file for <code>AppComponent</code> :

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
We start off by importing <code>Component</code> and <code>OnInit</code> from the Angular 2 core.
Next, we use the <code>@Component</code> decorator to define metadata for the component. In the code snippet above, we define the component's selector (```<app></app>```) and its template content(```'root component'``` .). If you'd like to learn more about component decorator properties, [this](https://angular.io/docs/ts/latest/guide/cheatsheet.html) cheatsheet provides a list of all <code>@Component</code> decorator properties.



### Running

 With the application's most essential components in place, you can now run it and preview it. To do so, run the following commands in your root directory:
 
```bash
 $ npm run build
 $ npm run serve
```
Go to [http://localhost:8080](http://localhost:8080) and you will see <code>'root component'</code> displayed in the upper left side of the window.

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

The wallpaper listing contains the wallpapers themselves plus additional information about the listing as a whole. To get an idea how a wallpaper listing would look like, we need to visit the [Reddit API documentation](https://www.reddit.com/dev/api/) to get a feel for how the information is structured. By reading it, we can infer that we can use the <code>before</code> and <code>after</code> to paginate through the results.


Therefore, the listing will contain

- An array of wallpapers
- An attribute denoting pagination/what part of the total collection is being viewed (<code>after</code>)


```typescript
// src/app/wallpaperlisting.model.ts
import { Wallpaper } from './wallpaper.model';//Don't forget import the Wallpaper model

export class WallpaperListing {
    wallpapers: Array<Wallpaper>;//array of wallpapers
    after: string;
}
```
### GETing data

The data models are figured out and we are going to put them to use. We'll do this by creating an injectable service that will parse the data and assign it to our models.

```typescript
// src/app/reddit.service.ts

import { Injectable } from '@angular/core';
import { Http, Response } from '@angular/http';

import { Observable }     from 'rxjs/Observable';//using the RxJS observables
import { Wallpaper } from './wallpaper.model'; //improting the wallpaper model
import { WallpaperListing } from './wallpaperlisting.model';//importing the wallpaper listing model

@Injectable()
export class RedditService {
    
    constructor(private http: Http) {
    }

}

```

We make the <code>RedditService</code> class injectable by adding <code>@Injectable()</code> before its definition. In order to make the HTTP module available inside the class, we have to inject it in via the <code>constructor</code> function.

Next, we add the <code> getWallpapers() </code> function:

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

The <code> getWallpapers </code> function accepts one parameter (<code>after</code> as string). The second part is its ```: Observable<WallpaperListing> ```. It denotes what type of variable is expected to be returned when the function <code>return</code>s. In it, we use Angular 2's HTTP library to get a response from [https//www.reddit.com/r/wallpapers.json?raw_json=1](https//www.reddit.com/r/wallpapers.json?raw_json=1) and map it through the <code>mapWallpapers</code> function.


<code> mapWallpapers </code> gets the response data. You can look at the data  [here](//www.reddit.com/r/wallpapers.json?raw_json=1). It might seem like a lot of information, but we only need a small chunk of it. Notice that the post items are contained in the <code>children</code> attribute, which is an array. Let's  iterate through them!

We only need to visualize posts that are images, so while iterating, we'll be choosing only posts that have their <code>post_hint</code> attribute set to <code> "image" </code>. Here is how the code looks:

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

 In the <code>forEach</code> loop, we also [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) the preview images to choose the ones which are 960 pixels wide. The ternary operator checks if such preview image is found. if not, the url is replaced with the full-resolution url:
```ts
  let previewImage = resolutions.filter(m => m.width === 960)[0]; 
  item.previewUrl = previewImage ? previewImage.url : item.url;
```
 
 At the end of the function, we attach the pagination attribute from the JSON data (  <code> after </code>)  to the <code>WallPaperListing</code> model and return the results.
 
 Here is the complete code for the two functions ([link to the whole service file](https://github.com/hggeorgiev/angular2-wallpaper-wall/blob/master/src/app/reddit.service.ts)):
```ts
// src/app/reddit.service.ts
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

### Displaying results


```typescript
//  src/app/wallpaperlisting.component.ts

import { Component, OnInit } from '@angular/core';
import { RedditService } from './reddit.service';
import { WallpaperListing } from './wallpaperlisting.model';


@Component({
    selector: 'wallpaper-listing',
    template: require('./wallpaperlisting.component.html'),
    providers: [RedditService],
    directives: []
})

export class WallpaperListingComponent implements OnInit  {

    private wallpaperListing: WallpaperListing;
    private error: boolean;

    constructor(private service: RedditService) { }

    ngOnInit() {
        this.wallpaperListing = new WallpaperListing();
        this.loadWallpapers(null);
    }

    open(url:string) {
        window.open(url,'_blank');
    }

    loadWallpapers(after:string) {
        this.service
            .getWallpapers(after)
            .subscribe(result => {
                    if(after === null) {//loading for the first time - no pagination available!
                        this.wallpaperListing = result;
                    } else {
                        this.wallpaperListing.wallpapers = this.wallpaperListing.wallpapers.concat(result.wallpapers);
                        this.wallpaperListing.after = result.after;//update the pagination
                    }

                }, error => this.error = true
            );
    }
}


```

We put <code>RedditService</code> as a provider in the component decorator and we inject it in the constructor to make it available inside.
   
    constructor(private service: RedditService) { }

We use <code> implements  OnInit </code> and <code>ngOnInit() {} </code> to implement a [lifecycle hook](https://angular.io/docs/ts/latest/guide/lifecycle-hooks.html#!#hooks-overview) that is executing when the component loads. For now we'll leave it blank and use it when we're going have to load the data from Reddit.

The <code>loadWallpapers</code> function is used to fill up the local <code>wallpaperListing</code> class with data. We pass <code>null</code> to the function to indicate that <code>wallpaperListing</code> is being filled with data for the first time.

The <code>open</code> function will open a new tab with a URL for a selected wallpaper. We will do so by binding the function to a <code>click</code> event in the template.
 
 

Next, we will add a template to <code>WallpaperListingComponent</code>:

```html
<!-- src/app/wallpaperlisting.component.html -->
<div class="main-container">
    <div class="image-container" *ngFor="let wallpaper of wallpaperListing.wallpapers">
        <img src="{{ wallpaper.previewUrl }}" class="image-item" />
        <div class="actions">
            <button class="btn" (click)="open(wallpaper.url)"><i class="material-icons">open_in_new</i></button>
        </div>
    </div>
    <div *ngIf="error" class="image-container load-more">There was an error connecting to reddit. Please try again later.</div>
</div>
```
We use [ngFor](https://angular.io/docs/ts/latest/api/common/index/NgFor-directive.html) to loop through the <code>wallpapers</code> array of the <code>wallpaperListing</code> to display the attributes of each item.

We also utilize the [click](http://learnangular2.com/events/) event handler to invoke the <code>open</code> function with the current wallpaper's URL as an argument.

Add the template's URL to the decorator of the <code>WallpaperListingComponent</code>:


```ts
// src/app/wallpaperListing.component.ts
@Component({
//
template: require('./wallpaperlisting.component.html'),
//
})
```
### Updating  AppComponent

In order for the <code>WallpaperListingComponent</code> to work, we must:

- Import it in AppComponent
- Add it as a directive in the AppComponent decorator
- Put its selector in the template



```ts
// src/app/app.component.ts
import { Component } from '@angular/core';
import {WallpaperListingComponent} from './wallpaperlisting.component' //importing the component 
@Component({
    selector: 'app',
    template: '<wallpaper-listing></wallpaper-listing>',//adding its selector to the template
    directives: [ WallpaperListingComponent ] //adding the component as a directive
})
export class AppComponent {
    constructor() { }
    
}
```

Open your app in [http://localhost:8080](http://localhost:8080) (don't forget to run <code>npm run serve</code> before that!) and you'll see some the images being displayed right from reddit's /r/wallpapers!

#### Adding infinite scroll
Right now, we can only see a few wallpapers. Let's add [angular2-infinite-scroll](https://www.npmjs.com/package/angular2-infinite-scroll) to our application to make use of the pagination. Here's how you can do it:

1. Open your terminal and add it as a dependency to your project.
```bash
$ npm install angular2-infinite-scroll --save
```
2. Import it in <code>WallpaperListingComponent</code>:
```ts
// src/app/wallpaperlisting.component.ts
import { InfiniteScroll } from 'angular2-infinite-scroll';
```
3. Add it as a directive in <code>WallpaperListingComponent</code>'s decorator:

    ```ts
    // src/app/wallpaperlisting.component.ts
    
    @Component({
        selector: 'wallpaper-listing',
        template: require('./wallpaperlisting.component.html'),
        providers: [RedditService],
        directives: [InfiniteScroll] //add  InfiniteScroll as a directive
    })
    import { InfiniteScroll } from 'angular2-infinite-scroll';
    ```

4.  Add the directive's selector and the <code>onScroll</code> event listener to the template of <code>WallpaperListingComponent</code>:
    ```html
     <!-- src/app/wallpaperlisting.component.html -->
     <div class="main-container" infinite-scroll (scrolled)="loadMore()">
    ```
5. Add the <code>loadMore</code> function:

    ```ts
    // src/app/wallpaperlisting.component.ts 
    
    loadMore() {
        if ((window.innerHeight + window.scrollY) >= document.body.offsetHeight) {
            this.loadWallpapers(this.wallpaperListing.after);
        }
    }
    ```

```loadMore``` is simple - if the window's vertical scroll position (```codeY```) and the browser window's ```innerHeight``` is greater or equal than the document's offset height, then the bottom of the document has been reached and ```loadWallpapers``` is called with the pagination as an argument.

## Styling

 We'll use the <code>styles</code> attribute in the component's decorators to add references to each of the components. If you've looked closely at the project's devDependencies or the Webpack configuration, you've probably figured out that we'll be using [LESS](http://lesscss.org/).
 
 First, add the file:
 
 **app.component.less**
```less
// src/app/app.component.less

wallpaper-listing {
  background: #000;
  font-family: 'Open Sans', sans-serif;
  margin: 0;
  padding: 0;
}

body {
  margin: 0;
  padding: 0;
  background: #363636;
}
```

Then reference it in <code>AppComponent</code>
```ts
// src/app/app.component.ts
@Component({
    //..
    styles: [require('./app.component.less').toString()],
    //.. 
})
```

Do the same with <code>WallpaperListingComponent</code>:

**wallpaperlisting.component.less**
```less
// src/app/wallpaperlisting.component.less
.main-container {
  margin: 0 auto;
  max-width: 960px;
  -webkit-box-shadow: 0px 0px 10px 1px rgba(0,0,0,0.75);
  -moz-box-shadow: 0px 0px 10px 1px rgba(0,0,0,0.75);
  box-shadow: 0px 0px 10px 1px rgba(0,0,0,0.75);

  .image-container {
    position: relative;
    line-height: 0;
    -webkit-box-shadow: inset 0px 0px 31px -6px rgba(0,0,0,0.75);
    -moz-box-shadow: inset 0px 0px 31px -6px rgba(0,0,0,0.75);
    box-shadow: inset 0px 0px 31px -6px rgba(0,0,0,0.75);

    &:hover, &:active {
      transition: 0.5s ease-out all;

      .image-item {
        max-height: 600px;
      }

      .actions {
        visibility: visible;
        opacity: 1;
      }
    }

    .image-item {
      width: 100%;
      max-height: 50vh;
      object-fit: cover;
      transition: 0.5s ease-in all;
    }

    .actions {
      position: absolute;
      z-index: 10;
      right: 5px;
      bottom: 9px;
      visibility: hidden;
      transition: 0.5s ease-out all;
      opacity: 0;

      .btn {
        padding: 15px 25px;
        background: rgba(0,0,0,0.5);
        color: #fff;
        border: none;
        cursor: pointer;
        font-size: 18px;
        border-radius: 5px;

        &:hover {
          background: rgba(0,0,0,0.9);
        }

        &:active, &:focus {
          outline: none;
        }
      }
    }

    &:hover {
      background: #37474F;
    }
  }
}


```

```ts
// src/app/wallpaperlisting.component.ts
@Component({
    //..
    styles: [require('./wallpaperlisting.component.less').toString()],
    //.. 
})
```

And that's it! Now you have a beautiful wall of wallpapers that you built with Angular 2!
In case you missed something, I uploaded all the code for the project in a [Github repository](https://github.com/hggeorgiev/angular2-wallpaper-wall).
