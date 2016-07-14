In this article I am going to introduce step-by-step the basics of using AngularJS. Lets start with the basic file structure in AngularJS.

* app.js
* controller.js
* index.html

#### Quick note
Here, `app.js` is a JavaScript file in which I created an instance of an [Angular module](http://www.w3schools.com/angular/angular_modules.asp). The `controller.js` Javascript file has an [Angular controller](http://www.w3schools.com/angular/angular_controllers.asp) that is registered with the `app.js` Angular module and contains business logic (programming between end UI and database). `index.html` is the view page where I place my html code and loaded `app.js` and then `controller.js` script files.

## app.js

Let's implement `app.js` first. 

```
var app = angular.module('myApp',[]);
```
The [angular.module](https://docs.angularjs.org/api/ng/function/angular.module) is a global place for creating, registering, and retrieving Angular modules. Modules are containers for the various parts of an application, such as controllers and directives. 

#### Module

```
// Create a new module
var myModule = angular.module('myModule', []);

// Register a new controller
myModule.controller('myController',['$scope',function($scope){}])

// Configure existing services inside initialization blocks.
myModule.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {

}]);
```

The `angular.module` is the entry point of Angular applications. Each application has just one module that gets the rootElement  `<html>` or `<body>` tag.

We created an `angular.module` called `myApp` in app.js file.

## controller.js
```
app.controller('myCtrl',['$scope', function($scope) {
  $scope.name = 'Angular ';
}]);
```

In `controller.js`, we register our Controller with the `angular.module` (i.e `app`) initialized in `app.js`

`app.controller` has two parameters: the name of our controller and the required `angular services/dependencies`. For example, we loaded the `$scope` service and created an instance in the function.



#### $scope

[Scope](https://docs.angularjs.org/guide/scope) is a pre-defined object in Angular which uses `$watch` to watch the status of model. It's importnant to know that `$scope` has hierarchical structure, so scope inherits from its parent scope and is capable of creating child scopes.

```
// parent scope
var parent = $rootScope();

// child scope
var child = parent.$new();
```

Using `$scope` (and its built-in `$watch` command), we can bind html and js, causing changes in html to translate to changes in our js, and vice versa. In our example, we created a scope object called `$scope.name` and finished binding in view(html page).

## index.html

```html
<html ng-app="myApp">

    <head>
        <title>AngularJS App</title>
        <script data-require="angular.js@1.4.x" src="https://code.angularjs.org/1.4.9/angular.js" data-semver="1.4.9"></script>
        <script src="app.js"></script>
        <script src="controller.js"></script>
    </head>

    <body ng-controller="myCtrl">
    
        <input ng-model="name" />
        <p>Hello {{name}} !</p>
        
    </body>

</html>
```

When `index.html` is loaded, it injects `angular.js`, `app.js`, and `controller.js` into the browser per our defined script from `index.html`. As a result, `angular.module` is placed at rootElement, and `myApp` and `MyCtrl` become available for `index.html`.

```html
<html ng-app="myApp">
```

#### Directive

Angular has [Directive](https://docs.angularjs.org/api/ng/directive) components such as `ngBind, ngValue, nclass` and many more. In this article I explained just some of the `ngApp` and `ngController` directives. I will talk about other directives in my next tutorial.


#### ng-app

[ngApp](https://docs.angularjs.org/api/ng/directive/ngApp) is Angular's pre-defined directive, meaning that Angular will default to ngApp if no other directive is present.

> If `ngApp` is not placed in `rootElement`, then the controller will fail to load. This failure arises because `controller(myCtrl)` is registered with `ngApp("myApp")`, the unspecified default option.


#### ng-controller

```html
 <body ng-controller="myCtrl">
```

The [ngController](https://docs.angularjs.org/api/ng/directive/ngController) directive binds the controller class to the view. This is a key aspect of how Angular supports the principles behind the [Model-View-Controller (MVC)](http://www.tutorialspoint.com/design_pattern/mvc_pattern.htm) design pattern.

##### MVC components in angular

* Model — Models are data assingned to `scope` variable for the Document Object Model (DOM), and consumed by API `$watch`.

* View — The html template that rendered and bound to the value in DOM.

* Controller — In this case, the ngController directive specifies a Controller class that contains business logic behind the application's functions.

#### Data-Binding

```html
<body ng-controller="myCtrl">
    <input ng-model="name" />
    <p>Hello {{name}} !</p>
</body>
```
`{{name}}` serves as a variable to store the `$scope.name` value present in the ngController `myCtrl`.

#### ng-model
```html
<input ng-model="name" />
```

[ng-model](https://docs.angularjs.org/api/ng/directive/ngModel) provides a unique two-way-binding feature; it binds value to input.

#### Angular Expressions - {{Exp}}

Expressions are used to get or print the values of variables in the view. Direct directives appropriate the scope value for interpolation binding. However functionExpressions fail on this task. For Example:

```html
// work for
<p>{{scopeValue}}</p>

// fail for
<button ng-click="{{functionalExpressions()}}">Click me !</button>

// for directive used directly
<button ng-click="functionalExpressions()">Click me !</button>
<input ng-model="name" />
```
Anything written in `input tag` will be observed by scope API `$watch` and will display the value of the `$scope.name` object in paragraph tag.
```html
<p>Hello {{name}}!</p>
```

#### Source Code
See [plunker](https://plnkr.co/edit/5sxudDgiAOGe8hKYjRLK?p=preview) for live example.

#### ScreenShot
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/aeeb2d6b-58b1-4d24-97cc-9c42d0ef7954.png)


This is how we implement `angular.module`, register ngController to the Angular module, and use the scope variable with the controller and the view to bind with AngularJS's ngModel. 

I hope you found this article informative! See you soon with my next article on AngularJS. 