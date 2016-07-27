## Angular 2.0 is finally here! Well, almost...
At the end of the year, Angular's dev team was happy to announce that [Angular 2.0 was officially in beta](http://bit.ly/1Lw7K7S). This means that although it may not be ready for production, the core concepts that the framework was built upon are not going too change, at least not significantly, so those who plan on using Angular 2.0 can now begin to learn about its different features without having to worry about the framework becoming obsolete. In turn, I decided to create this guide to bring developers up to speed with the new features of Angular 2.0 (also known as ng2) and how the framework works altogether. 

Specifically, this guide aims to provide readers with knowledge about the following areas of Angular 2.0, and how they've changed from ng1. (More to be added)

* Components
* New Template Syntax
* Data Binding
* Modules 
* Client Side Routing
* Resources for getting started

##### Quick Note
Throughout this guide, I will be referring to Angular 1 as ng1 and Angular 2 as ng2. Also, it will deal primarily with TypeScript as it is the official language used to build the framework and makes the development process loads easier

### Components
Components are the building block of all Angular 2.0 applications. They build on the directives from Angular 1, however they are much simpler. A component is just an encapsulated piece of code that we can reuse throughout our application. 

Check out this sample component from [learnangular2.com](learnangular2.com) below.

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

Applications built with ng2 are made up of several components. In order to create a component, we must use the `@` decorator which will tell the framework that we are trying to build a certain kind of class; in this case, ng2 will know we are trying to build a component. 

Notice the line `selector: 'my-component'`. This means that Angular will render this component template everywhere that `<my-component></my-component>` appears in the application, granted that the component is imported into the app's main component. 

#### The Top Level Component 

Those who used Angular 1 will remember the main application file, or `angular.module`, that served as our top-level module. It was inside this file that we would add an app's dependencies.

In Angular 2, we use a top-level component in the same way. To demonstrate this process, I've included another piece of code which imports `MyComponent` into our main app files, and then grants the `AppComponent` access to it. 

That said, there are a few important pieces of information here that we should be focusing on.

1. Our file MUST import the file that exports `MyComponent`. 
2. `MyComponent` will only be accessible if it is included as a directive of our component. If we forget to make the target component into a directive, our `<my-component></my-component>` tags will not render anything. 
  * **Don't let the word directive confuse you! Components are built from directives, and are known as "component directives"** 
3. The top-level component of an Angular 2 application must be passed into this `bootstrap` function in order for our application to run. Often times, we will see an application's `bootstrap` function being used in a separate file.


```typescript
/* src/app.ts
** Don't get caught up with these import statements
** We will learn more about them in the module section
*/
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

## Template Syntax
### Tying Templates to Components
Angular's template syntax has change quite a bit in ng2, but these changes are nothing to be afraid of. In ng2, the Angular team took advantage of JavaScript's new [template strings](http://mzl.la/1QKCT3g) to provide developers with the choice of defining their template within the TypeScript file that holds their component like we saw in the examples above. Additionally, ng2 provides developers with the option of using the `templateUrl` to specify `.html` file. 

This choice is ultimately up to the developer. Personally, when a template becomes large, I think using an external html file is much easier because we are unable to use tools such as Emmet or HTMLBeautify inside a TypeScript file. On the other hand, if a template is merely a few lines long then it's best to keep the TypeScript file and have all your code in one place.

### No
In ng1, we are given a set of directives (such as `ng-click`) that allow us to interact with different attributes in our application. In Angular 2, we can now directly interact with the DOM, and bind both actions and  Although ng2 ships with a set of built-in component directives, we no longer have to use these directives, 

## Data Binding 
Two years ago, if you were to ask any Angular developer about the benefits of using the framework, the developer would probably refer to its ability to perform two-way data binding. When learning ng1, the very first example was always very similar to this [jsfiddle example](https://jsfiddle.net/Lt7aP/4/) in which an input tag has an `ng-model` appended inside of it. Additionally, the `ng-model` directive would be assigned to the `$scope` of a particular object, and from there we would deal with it how we wanted, hence  the concept of two-way binding.

Although this provided a solution for dealing with user-inputted data, many soon realized that the idea of two-way data binding wasn't necessarily the best way of doing things because it overworked the application.

That said, Angular 2.0 takes a uni-directional approach to the way in which it binds data and events to an application. Later in this guide, we will speak more about [RxJS](http://bit.ly/1pns9SF) and how ng2 uses observables rather than promises to deal with events. At the moment, we simply need to focus on two different things that trigger events: Inputs and Outputs. 

### Inputs
To deal with inputs,

  



If you are unfamiliar with this example, and are coming into Angular 2 without any past knowledge of directives, than you can consider yourself lucky as you no longer have to use this method 

Although this providing a solution for dealing with user-inputted values, it was no