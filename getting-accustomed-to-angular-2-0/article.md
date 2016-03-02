## Angular 2.0 is finally Here! Well, Almost
At the end of the year, Angular's dev team was happy to announce that [Angular 2.0 was officially in beta](http://bit.ly/1Lw7K7S). This means that although it may not be ready for production, the core concepts that the framework was built upon are not going too change, at least not significantly, so those who plan to use Angular 2.0 can now begin to learn about its different features without having to worry about them becoming obsolete. In turn, I decided to create this guide dedicated towards bringing developers up to speed with the new features of ng2 and how the framework works altogether. 

Specifically, this guide aims to provide readers with knowledge regarding the following areas of Angular 2.0, and how they differ from ng1.(More to be added)

* Components in Angular 2.0
* New Template Syntax
* Data Binding
* How Angular 2.0 Uses Modules 
* Client Side Routing
* Resources for getting started

### Components in Angular 2.0
Components are the building block of all Angular 2.0 applications. They are what directives were in Angular 1, however they are much, much more simple in every sense. Simply put, a component is just an encapsulated piece of code that we can reuse throughout our application. 

I have included an example component from [learnangular2.com]() below.

```typescript
//src/my-component.ts
import {Component} from 'angular2/angular2'

@Component({
  selector: 'my-component',
  template: '<div>Hello my name is {{name}}. <button (click)="sayMyName()">Say my name</button></div>'
})
export class MyComponent {
  constructor() {
    this.name = 'Max'
  }
  sayMyName() {
    console.log('My name is', this.name)
  }
}
```

Applications built with Angular 2 are made up of a whole bunch of different components. In order to create a component, we must use the `@` decorator which will tell Angular that we are trying to build a class of something; in this case, Angular will know we are trying to build a component. Furthermore, Angular will render this template everywhere that `<my-component></my-component>` appears in an application, granted that the component is imported into the app's main component. 

#### Top Level Component 
Those who used Angular 1 will remember the main application file, or `angular.module` that served as our top-level module. It was inside this file that we would add our dependencies that our application relied on.

In Angular 2, we use a top-level component in the same way. To help show how this works, I've included another piece of code which imports `MyComponent` into our main app files, and then grants the `AppComponent` access to it. That said, there are a few important pieces of information here that we should be focusing on.

1. Our file MUST import the file that exports `MyComponent`. 
2. `MyComponent` will only be accessible if it is included as a directive of our component. If we forget to do this, our <my-component></my-component> tags will not render anything. 
  * *Don't let the word directive confuse you! Components are built from directives, and are known and "component directives"* 
3. The top-level component of an Angular 2 application must be passed into this `bootstrap` function in order for our application to run. Often times, we will see an application's `bootstrap` function being used in a separate file.


```typescript
/* src/app.ts
** Don't get caught up with these import statements
*/ We will learn more about them in the module section
import {Component} from 'angular2/core';
import {RouteConfig, ROUTER_DIRECTIVES} from 'angular2/router';
import {FORM_PROVIDERS} from 'angular2/common';

//#1 
import {MyComponent} from './my-component';

@Component({
  selector: 'app', // <app></app>
  providers: [...FORM_PROVIDERS],
  // #2 
  directives: [...ROUTER_DIRECTIVES, MyComponent],
  pipes: [],
  styles: [require('./app.scss')],
  template: `
  <h2>{{message}}</h2>
  <my-component></my-component>
  `
})
export class AppComponent {
 message:string = "Welcome to my app";
}
bootstrap(AppComponent);
```



