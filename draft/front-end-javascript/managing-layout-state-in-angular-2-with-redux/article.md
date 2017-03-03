# The benefits of controlling your application layout with Redux

# Setup
 The examples  below will be done with [ng-boostrap]() since it's one of the most popular libraries with Angular 2 components for Bootstrap 4. However, you can also implement these examples with other component libraries such as [Material Design](https://github.com/angular/material2) by following the same design principles and making small adjustments to the code so that it works with the API of the corresponding library.
 
### Dependencies
 Here's what packages you need to install in order to start working:
 
**Redux**
 - [@ngrx/store + @ngrx/core](https://github.com/ngrx/store)
 - [@ngrx/effects](https://github.com/ngrx/effects)
 - [ngrx-store-logger](https://github.com/btroncone/ngrx-store-logger)
 
** Bootstrap **
 - [Boostrap 4](https://github.com/twbs/bootstrap)
 - [ng-boostrap](https://github.com/ng-bootstrap/ng-bootstrap)
 
### Installing

To ensure a smooth setup, [Angular CLI](https://github.com/angular/angular-cli) will be used to initialize the applicaiton architecture. Make sure you have it installed globally before you proceed. In your terminal, write the folowing commands to initialize your Angular 2 app:

```
$ ng new redux-layout-tutorial-app
$ cd new redux-layout-tutorial-app
```


```
$ yarn add bootstrap@4.0.0-alpha.6
```
To add Bootstrap's assets to your project, open `angular-cli.json` in your app root and add the following:

```javascript
apps: [
  {
   //..
   "styles": [
        "../node_modules/bootstrap/dist/css/bootstrap.css"
    ],
    //...
    "environments": {
    //...
    "scripts": [
      "../node_modules/jquery/dist/jquery.js",
      "../node_modules/tether/dist/js/tether.js",
      "../node_modules/bootstrap/dist/js/bootstrap.js"
    ]
  }
```

This will let your `angular-cli` setup to locate the javascript and css files of your Boostrap installation and add them in the build of the project.

Next, add `ng-boostrap` to  your dependencies:
```
$ yarn add @ng-boostrap/ng-boostrap
```

And include the `NgbModule` in your app's root module (located in `app.module.ts`):
```
import {NgbModule} from "@ng-bootstrap/ng-bootstrap";

@NgModule({
    //..
    imports: [
        NgbModule
    ],
    //..
})
```


# Modals

# Sidebars

# Pagination

# Accordion 

# Confirmation dialogs

# Window size

# Events

# Colors
