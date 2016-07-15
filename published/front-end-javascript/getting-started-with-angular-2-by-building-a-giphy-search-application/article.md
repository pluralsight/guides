In this post you'll learn how to get started with Angular 2 by building a Giphy search application. You don't need to be familiar with Angular 1, although some familiarity with JavaScript is needed [there](https://www.pluralsight.com/courses/javascript-from-scratch) [are](https://developer.mozilla.org/en-US/Learn/JavaScript) plenty resources online to get you started; even [free](https://github.com/getify/You-Dont-Know-JS) [books](http://jsbooks.revolunet.com/)). Everything else we'll cover here. For those who come from the Angular 1 world, I will draw a comparison in certain areas.

## TL;DR

Learn how to use [angular-cli](http://ngcli.github.io/) to build an Angular 2 application for searching [Giphy's gifs](http://giphy.com/) by using [their API](https://github.com/Giphy/GiphyAPI). Main new Angular 2 concepts are explained along the way.

## Introduction
You've all heard the news; Angular 2 is/will be the next best thing in the **web and mobile** (with [Ionic framework](http://www.nikola-breznjak.com/blog/javascript/ionic/ionic-framework-a-definitive-10000-word-guide/)) development world. You've waited for some time for it to become stable, and now as the release date approaches ([**RC4**](http://angularjs.blogspot.hr/2016/06/rc4-now-available.html) is already out as of this writing) you're finally ready to invest your valuable time to learn what's new and ever so slightly cool in this new reincarnation of the AngularJS framework.

Now, at this point you may still be something like:

> Yeah, sure, but I've **heard** that React/Ember/NameYourPoison is way better!?
 
I will answer this question similarly as I did on a [conference talk](http://www.nikola-breznjak.com/blog/miscellaneou/weblica-2016/) I gave a few months ago:

> Please just stop with the [analysis paralysis](https://en.wikipedia.org/wiki/Analysis_paralysis) already. Pick a framework (**any** framework for that matter) that community is using, use it and see how far it gets you. All these talks about X being slow or Y being better just make no sense until you try it yourself for **your use case and preference**. Besides, you should always be learning, but that's for some other occasion to talk about (_in case you're curious, start [here](http://jamesclear.com/marginal-gains) or [here](http://jamesclear.com/self-improvement)_)...

With all this said, fasten your seatbelts, take a venti (or trenta) sized cup of coffee and let's go!

## Demo app
As said in the TL;DR, in this introduction tutorial we'll build an application for searching (and showing) gifs from the [Giphy](http://giphy.com/) website by using [their API](https://github.com/Giphy/GiphyAPI). In the end, we'll also deploy our app to [Github pages](https://pages.github.com/).

You can [try the app here](http://hitman666.github.io/giphy-search/), and you can fork the complete source code on [Github](https://github.com/Hitman666/GiphySearch_Angular2).

## Prerequisites
I'm pretty sure that if you're reading this tutorial that you have the following tools installed (or that you've at least heard of them), but in case you don't please install them:

+ [Node.js](https://nodejs.org/en/) - step by step [guide](https://hackhands.com/how-to-get-started-on-the-mean-stack/#installing-node-js) for both Windows and Mac
+ [Git](https://git-scm.com/) - free interactive tutorial [here](https://try.github.io/)
+ [Bower](https://bower.io/) - comprehensive tutorial [here](https://www.digitalocean.com/community/tutorials/how-to-manage-front-end-javascript-and-css-dependencies-with-bower-on-ubuntu-14-04)
+ [Gulp](http://gulpjs.com/) - getting started tutorial [here](https://github.com/gulpjs/gulp/blob/master/docs/getting-started.md)

## Transitioning from Angular 1?
So, you've skimmed a few lines of this new shiny Angular 2.0 thing, and you were like

![](http://i.imgur.com/YbL08jU.jpg)

Trust me; I share your pain. When I first looked at it, I was pretty comfortable with how Angular 1 did things and then I saw this, arguably better way, and I thought I must be loosing the grip.

However, now a converted man, I'll tell you the same thing you've probably heard before:

> Angular 2.0 is so much better, and you should definitely use it :).
 
But, OK, I won't stop here, through these series I'll show you why I hold these views now.

All jokes aside, personally I know that any change (be it for better or worse) is hard, and we tend to be reluctant towards it. However, as with any new tool or technology; once you get to use it some time, you get familiar with it, and eventually you can't remember why you didn't like it in the beginning.

_If you want to get into the details of how to migrate your Angular 1 app to Angular 2, start [here](https://angular.io/docs/ts/latest/guide/upgrade.html)._

## Angular CLI
Setting up an Angular 2 application (at least at this point in time) from the ground up is actually a lot of work in terms of installing and configuration settings.

Folks at [Ember](http://emberjs.com/) had one great advantage; a tool called `ember-cli`, that bootstraps the application and helps with some common project operations and scaffolding.

Sure, we had [Yeoman](http://yeoman.io/) and its [Angular generators](https://github.com/yeoman/generator-angular), but still that wasn't something that was a convention, and definitely it wasn't used as a defacto tool as in Ember's case. Proof that Ember CLI is a way forward is that angular-cli's [Github page](https://github.com/angular/angular-cli) says:

> Prototype of a CLI for Angular 2 applications based on the [ember-cli](http://www.ember-cli.com/) project.

Now, enter [Angular CLI](https://cli.angular.io/). Although this project currently is in beta, we can do things like:

+ Create a scaffolded app with proper directory structure and included tests, with one simple command (`ng new`)
+ Generate components, routes, services and pipes (the CLI will also create simple test shells for all of these)
+ Easily put your application in production (`ng serve`)
+ Deploying the app via GitHub Pages (`ng github-pages:deploy`)
+ Run unit tests or your end-to-end tests
+ Execute the official Angular2 linter (`ng lint`)
+ [...](https://github.com/angular/angular-cli)

Cool, this got us excited, as now we'll have the folks from Ember becoming converts (teasing, don't judge me :)), so now let's see how to install this CLI.

### Installing angular-cli
It's as simple as running:

`npm install -g angular-cli`

You can confirm that the installation went well if you run:

`ng --help`

and get a bunch of output and help on various angular cli commands.

Just for reference (in case you follow this tutorial at a later stage and something is not the same as I output it here), my version (`ng --version`) as of this writing is `angular-cli: 1.0.0-beta.9`.

### Starting a new app with angular cli
We'll call our app, originally, `GiphySearch`. So, let's start a new app using angular-cli:

`ng new GiphySearch`

You should get an output similar to this:

```
# nikola in ~/DEV/Angular2 [13:57:55]
→ ng new GiphySearch
Could not start watchman; falling back to NodeWatcher for file system events.
Visit http://ember-cli.com/user-guide/#watchman for more info.
installing ng2
  create .editorconfig
  create README.md
  create src/app/app.component.css
  create src/app/app.component.html
  create src/app/app.component.spec.ts
  create src/app/app.component.ts
  create src/app/environment.ts
  create src/app/index.ts
  create src/app/shared/index.ts
  create src/favicon.ico
  create src/index.html
  create src/main.ts
  create src/system-config.ts
  create src/tsconfig.json
  create src/typings.d.ts
  create angular-cli-build.js
  create angular-cli.json
  create config/environment.dev.ts
  create config/environment.js
  create config/environment.prod.ts
  create config/karma-test-shim.js
  create config/karma.conf.js
  create config/protractor.conf.js
  create e2e/app.e2e-spec.ts
  create e2e/app.po.ts
  create e2e/tsconfig.json
  create e2e/typings.d.ts
  create .gitignore
  create package.json
  create public/.npmignore
  create tslint.json
  create typings.json
Successfully initialized git.
⠸ Installing packages for tooling via npm
├── es6-shim (ambient)
├── angular-protractor (ambient dev)
├── jasmine (ambient dev)
└── selenium-webdriver (ambient dev)

Installed packages for tooling via npm.
```

After this command finishes, let's `cd` into the project and run it:

```
cd GiphySearch
ng serve
```

You should get ...

```
# nikola in ~/DEV/Angular2/GiphySearch on git:master ● [14:05:03]
→ ng serve
Could not start watchman; falling back to NodeWatcher for file system events.
Visit http://ember-cli.com/user-guide/#watchman for more info.
Livereload server on http://localhost:49154
Serving on http://localhost:4200/

Build successful - 889ms.

Slowest Trees                                 | Total               
----------------------------------------------+---------------------
BroccoliTypeScriptCompiler                    | 484ms               
vendor                                        | 332ms               

Slowest Trees (cumulative)                    | Total (avg)         
----------------------------------------------+---------------------
BroccoliTypeScriptCompiler (1)                | 484ms               
vendor (1)                                    | 332ms               
```

and your browser should show you the string `app works!` when you visit the app on the link: [http://localhost:4200/](http://localhost:4200/).

> In case you're curious about the 'Could not start watchman' output above. You can learn more on the [link](https://ember-cli.com/user-guide/#watchman), but TL;DR is that you can just do `brew install watchman` if you're on Mac.

## Folder structure
Now, let's open this project in the editor of your choice (I'm using Sublime Text 3), and you should see something like this:

![](http://i.imgur.com/liUz0aS.png)

As I said, this is an introduction tutorial to get you running fast, so I won't be going into any specific details this time (this is up for some other posts), so here we'll only focus on the `src` folder. The contents of that folder should be something like this:

![](http://i.imgur.com/GYMJgEQ.png)

## What is this TypeScript thing?
Now, you might be wondering, what are all these `.ts` files?

Well, these are TypeScript files, and even though you don't need to use TypeScript with Angular 2, probably everyone on this Earth does. So, we won't play dumb, and we'll just go with the flow. _Yeah, it's better, you'll see._

So, what is it? On [their website](http://www.typescriptlang.org) they state that:

> TypeScript is a superset of JavaScript that compiles to clean JavaScript output.

Huh?

You may have already seen this image before:

![](http://i.imgur.com/GApAyU4m.png)

From it, one can simply (_don't you just love it how most tutorials refer to everything being simple and done in only 564 steps?_) conclude that this TypeScript thing contains the ES6, which then again contains ES5, which yet again means that TypeScript contains ES5.

Since most browsers today can't run the ES6 or yet alone TypeScript, we use the so-called [transpilers](https://en.wikipedia.org/wiki/Source-to-source_compiler) which turn our TypeScript or ES6 code into ES5 code which is a 'plain ol' JavaScript that you're most probably familiar with and that basically all of today's browsers understand.

Some of the features that TypeScript brings to the table are:

+ [types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
+ [classes](https://www.typescriptlang.org/docs/handbook/classes.html)
+ [decorators](https://www.typescriptlang.org/docs/handbook/decorators.html)
+ [annotations](https://www.typescriptlang.org/docs/tutorial.html#type-annotations)
+ [imports](https://www.typescriptlang.org/docs/handbook/module-resolution.html)
+ [generics](https://www.typescriptlang.org/docs/handbook/generics.html)
+ [...](https://www.typescriptlang.org/docs/tutorial.html)

I won't go into details here; you can learn about each of them by clicking on the links. However, let's just take a look at a few things to get you started.

You now define variable as:

`var myVar: string;`

So, the new thing here is that we define the type of the variable. _Finally we won't be laughed at anymore for not having a strongly typed language :)_

Similar, you now have the ability to specify the return value type of the function:

```
function myFunc (msg: string): string {
    return 'I like to repeat what you said, therefore: ' + msg;
}
```

Just for reference, we have the following types:

+ string
+ number
+ array
+ enum
+ any
+ void

---
If you want to play with TypeScript on your own (in a separate project) you have to install it by running the following command:

`npm install -g typescript`

Then, you can write any TypeScript code, save it in a file, run it through the TypeScript compiler, and finally run the output of the TypeScript compiler with Node.

For a quick example, let's create a file named `tsTest.ts` and put the function from above in it, along with the call of the function like `console.log(myFunc('hello'));`. For a full reference, our file should look like this:

```
function myFunc (msg: string): string {
    return 'I like to repeat what you said, therefore: ' + msg;
}

console.log(myFunc('hello'));
```

Then run this file through the TypeScript compiler like this:

`tsc tsTest.ts`

Now you should see a `tsTest.js` file beside your `tsTest.ts` file, and now you can run it with node like this:

`node tsTest.js`

and as an output, you should get `I like to repeat what you said, therefore: hello`.

---

## Adding content
Ok, so, let's finally add something to our app.

But, where to start?

Well, one of the things I usually do when I get to a project for the first time is I look at the generated output and then I try to find the corresponding strings in the source code (Sublime has `Search` command, and I'm sure other editors have a similar thing).

So, if we search for the string `app works!` you'll see it's in the `app.component.ts` file (in the `app` folder). This file has the following content:

```
import { Component } from '@angular/core';

@Component({
  moduleId: module.id,
  selector: 'app-root',
  templateUrl: 'app.component.html',
  styleUrls: ['app.component.css']
})
export class AppComponent {
  title = 'app works!';
}
```

### Imports
Whoa, what's with those email characters?

Without knowing anything about this, we can see where we would change the `app works!` text. So, let's change that to `Welcome to GiphySearch`.

Now, feeling better? Ok, let's tackle what all this new code means here:

`import { Component } from '@angular/core';`

The `import` command defines which parts we'll _import_ and use in our code. In our case, we're importing the `Component` from the `@angular/core` module.
    
### Components
If we would have a newspaper for Web development, you would probably read titles like:

> Directives are so 2015, Components FTW

> All aboard with the Components
 
> Components, the new, ever so slightly awesome and shiny thing that you should be using....

I would probably not do so good as a journalist, so lets summarize: components are the new directives (in case you're familiar with Angular 1).

And, this is how a simplest component is written:

```
import { Component } from "@angular/core";

@Component({
  selector: 'awesome',
  template: `
      <div>
          Awesome content right here!
      </div>
    `
})

class Awesome { }
```

The Component Decorator (`@Component`) adds additional data to the class that follows it (`Awesome`), like:

+ `selector` - tells Angular what element to match
+ `template` - defines the view

The Component controller is defined by the `Awesome` class.

We'll be able to use this new tag (`awesome`) in our HTML like this: `<awesome></awesome>`, and once rendered to the user in the browser, it will be shown as `Awesome content right here!`.

You may have noticed that in this example we defined the template using backticks (`). Backticks in the template allow us to define multiline strings. 

Even though there are some people who define the template within the `@Component`, I tend to avoid that and have the template's HTML defined in a separate file (this is also how angular-cli prefers it, so Q.E.D).

### Adding input and button
So, what do we want to have? Well, quite simply, we want to have one input field and one button, and that's all.

To do this, add the following code to the `app.component.html` file:

```
<input name="search">
        
<button>Search</button>
```

### Give me some interAction
Having a simple search input field and a button doesn't help much. We want to click the button, and we want to output something to the console just to verify it's working correctly.

So, this is how you we define a function that will handle button click in Angular 2:

`<button (click)="performSearch()">Search</button>`

But, now if you click this button you will get an error. That's because we haven't defined the `performSeach` function anywhere.

Let's do that now. In the `app.component.ts` file, add the following function definition inside the `AppComponent` class:

```
performSearch(): void {
    console.log('button click is working');
}
```

With this, we have defined the `performSearch` function, which doesn't accept any parameters and it does not return anything (`void`). Instead, it just prints `button click is working` to the console (open up your developer tools to access it).

### Taking the input
What if we would like to print to the console the string that someone typed in the input field?

Well, first we need to add a new attribute to the `input` field:

`<input name="title" #searchTerm>`

The `#searchTerm` (a.k.a _resolve_) tells Angular to bind the `input` to the new `searchTerm` variable.

Then, we need to pass this variable in the `performSearch` function like this:

`<button (click)="performSearch(searchTerm)">Search</button>`

Finally, in the `app.components.ts` file change the `performSearch` function like this:

```
performSearch(searchTerm: HTMLInputElement): void {
    console.log(`User entered: ${searchTerm.value}`);
}
```

So, we added a new parameter to the function (`searchTerm`), which is of the `HTMLInputElement ` type. And we used the backticks to output the string in the `console.log`. You should notice how we used `${}` to output the variable inside the backtics. Because `searchTerm` is an object, we had to get the value of it by referencing its `value` property (`${searchTerm.value}`).

## Giphy search API
Finally, we come to the cool part, and that is to fetch some data from the service and to show it in our app (in our case, we'll show images).

So, how do we get this API? Well, if you do a simple [google search](https://www.google.com/search?q=giphy+api) for `giphy api` and open the [first link](https://github.com/Giphy/GiphyAPI) you'll get the documentation for their API.

So, what do we need? Well, we need the search API. If you scroll a bit, you'll find the following link:

`http://api.giphy.com/v1/gifs/search?q=funny+cat&api_key=dc6zaTOxFJmzC`

Great, now we see what kind of a request we need to create to search Giphy's gif database for a certain term.

If you open this link in the browser, you'll see what the service returns. Something like:

![](http://i.imgur.com/skihplVl.png)

Ok, and now what? So, now we want to fetch this data from within our app. Ok, but how do we do that?

### Angular HTTP requests
> I hope that this part will be improved in the future, but I have to share with you my pain in getting this to work properly. I've sifted through various StackOverflow [answers](http://stackoverflow.com/questions/33721276/angular-2-no-provider-for-http) to get this, seemingly easy, task done.

In Angular 2 we need to import the HTTP service from `@angular/http` module. We do that like this:

`import { HTTP_PROVIDERS } from '@angular/http';`

And now we can inject `Http` into our component.

So, let's translate this into our code. In the `app.components.ts` file, add the following code after the first import:

`import { Http, Response } from '@angular/http';`

Then, define two variables:

```
link = 'http://api.giphy.com/v1/gifs/search?api_key=dc6zaTOxFJmzC&q=';
http: Http;
```

Then, add the constructor like this:

```
constructor(http: Http) {
    this.http = http;
}
```

And, finally, add this to the `performSearch` function:

```
var apiLink = this.link + searchTerm.value;

this.http.request(apiLink)
    .subscribe((res: Response) => {
        console.log(res.json());
    });
```

Just for reference, to put it all in one listing, the contents of the `app.component.ts` file should be:

```
import { Component } from '@angular/core';
import { Http, Response } from '@angular/http';

@Component({
    moduleId: module.id,
    selector: 'app-root',
    templateUrl: 'app.component.html',
    styleUrls: ['app.component.css']
})

export class AppComponent {
    title = 'Welcome to GiphySearch';
    link = 'http://api.giphy.com/v1/gifs/search?api_key=dc6zaTOxFJmzC&q=';
    http: Http;

    constructor(http: Http) {
        this.http = http;
    }

    performSearch(searchTerm: HTMLInputElement): void {
        var apiLink = this.link + searchTerm.value;

        this.http.request(apiLink)
            .subscribe((res: Response) => {
                  console.log(res.json());
            });
    }
};
```

If you run this now, you'll get `No provider for Http!` error. As mentioned, after some long searching I made this work by adding this import line to the `main.ts` file (in the `app` folder):

```
import { HTTP_PROVIDERS } from '@angular/http';
```

And, also, in the same file, at the end you should add `bootstrap(AppComponent, [HTTP_PROVIDERS]);`.

Just for reference, to put it all in one listing, the contents of the `app.ts` file should be:

```
import { bootstrap } from '@angular/platform-browser-dynamic';
import { enableProdMode } from '@angular/core';
import { HTTP_PROVIDERS } from '@angular/http';
import { AppComponent, environment } from './app/';

if (environment.production) {
  enableProdMode();
}

bootstrap(AppComponent, [HTTP_PROVIDERS]);
```

Now, if you run the app and enter something in the search term and click the search button you'll see something like this in your console log:

![](http://i.imgur.com/N2Zs7Ch.png)

Here you can see that we're getting back the result object and that in it's `data` property there are 25 objects, which hold information about the images that we want to show in our app.

But, how do we show these images in our application?

We see that the api call returns 25 images, let's save this in some variable for later use:

`giphies = [];`

And, let's store the results from the api call to this variable:

`this.giphies = res.json().data;`

Just for reference, to put it all in one listing, the contents of the `app.component.ts` file should be:

```
import { Component } from '@angular/core';
import { Http, Response } from '@angular/http';

@Component({
    moduleId: module.id,
    selector: 'app-root',
    templateUrl: 'app.component.html',
    styleUrls: ['app.component.css']
})

export class AppComponent {
    title = 'Welcome to GiphySearch';
    link = 'http://api.giphy.com/v1/gifs/search?api_key=dc6zaTOxFJmzC&q=';
    http: Http;
    giphies = [];

    constructor(http: Http) {
        this.http = http;
    }

    performSearch(searchTerm: HTMLInputElement): void {
        var apiLink = this.link + searchTerm.value;

        this.http.request(apiLink)
            .subscribe((res: Response) => {
                  this.giphies = res.json().data;
                  console.log(this.giphies);
            });
    }
};
```

And, well, this is all great now, but we don't want to be logging out our objects to the console, we want to show them in our app.

After examining the result object for a while, we can conclude that to show the image, we need to position ourselves on the object `images`, then `original`, and finally on the `url` property.

Also, we don't want to just show one image. Instead, we want to show all of the images, so we'll use Angular 2's version of the ng-for from Angular 1. It now looks like this:

`<img *ngFor="let g of giphies" src="{{g.images.original.url}}">`

For reference, here's the full listing of `app.component.html`:

```
<h1>
  {{title}}
</h1>
        
<input name="search" #searchTerm>
        
<button (click)="performSearch(searchTerm)">Search</button>

<br>

<div *ngFor="let g of giphies">
    <img src="{{g.images.original.url}}">
</div>
```

At this point, if we take a look at the app and search for example for 'funny cats' we'll get this:

![](http://i.imgur.com/hAGImyAm.png)

Sure, this doesn't look 'nice', but you're free to add your own CSS in the `app.components.css` file if you're feeling designer-ish.

## Deploying to Github pages
Angular CLI had made things really easy for us, as all we have to do is run the following command:

`ng github-pages:deploy`

and you should get an output similar to this:

```
# nikola in ~/DEV/Angular2/GiphySearch on git:master
→ ng github-pages:deploy
Built project successfully. Stored in "dist/".
Deployed! Visit https://hitman666.github.io/giphy-search/
Github pages might take a few minutes to show the deployed site.
```

However, in my case, this just wasn't working. When I opened the [link](http://hitman666.github.io/giphy-search/) I got the 404. After searching, I found [this solution](http://developer.telerik.com/featured/quick-angular-2-hosting-angular-cli-github-pages/). But again, it didn't the issue.

Then, I figured that I already have the Github pages repo from way back. This command may work in your case, but if it doesn't then first create your own Github pages, by following the [official instructions](https://pages.github.com/).

Then, in your Github pages repo, just create a new folder (I called it `giphy-search`).

In the index.html file just change the base url to the name of your folder. In my case that's `giphy-search`:

`<base href="/giphy-search/">`

Then, just copy the contents of the `GiphySearch/dist` folder to `giphy-search` folder.

Add new files:

`git add .`

commit them:

`git commit -m 'adding new project to Github pages'`

and, finally, push them:

`git push origin master`

Now you'll have your app accessible at yourusername.github.io/giphy-search. In my case, the link is [http://hitman666.github.io/giphy-search/](http://hitman666.github.io/giphy-search/).

In case you would like to deploy this to your own server, just make sure you execute `ng build` and then copy the contents of the `dist` folder to your web server's proper folder. Also, don't forget about the proper base url setting.

## Conclusion
In this tutorial you've learned how to get started with Angular 2 by building an application for searching Giphy's gifs by using their API.

Also, you've learned about some new concepts like Component, and you've written some code in the new TypeScript language.

You've learned how using angular-cli helps take the pain of starting and getting everything together.

In the following tutorials we'll go into more detail, but this should give you enough to start and continue exploring on your own until next time.