In this article I am going to  start with the basics of angularJS implementation with step by step explaination and coding.

Lets start with the basic file structure :

* app.js
* controller.js
* index.html

#### Quick note
Here, `app.js` is a JavaScript file in which I created an instance of angular module. `controller.js` Javascript file has angular controller which is registered with angular module of `app.js` and contains business logic. `index.html` is the view page where I place html code and load `app.js` and `controller.js` script file.

## app.js

```
var app = angular.module('myApp',[]);
```
The [angular.module](https://docs.angularjs.org/api/ng/function/angular.module) is a global place for creating, registering and retrieving Angular modules.

#### Module

```
// Create a new module
var myModule = angular.module('myModule', []);

// register a new controller
myModule.controller('myController',['$scope',function($scope){}])

// configure existing services inside initialization blocks.
myModule.config(['$stateProvider', '$urlRouterProvider', function($stateProvider, $urlRouterProvider) {

}]);
```

`angular.module` is the entry point of angular application, one application have only one module which needs to placed on the rootElement  `<html>` or `<body>` tag.

We created `angular.module` named as `myApp` in app.js file.

## controller.js
```
app.controller('myCtrl',['$scope', function($scope) {
  $scope.name = 'Angular ';
}]);
```

In `controller.js` we register our Controller with the `angular.module` i.e `app` initialized in `app.js`

`app.controller` has two parameters, one is for name of `ngController` and second to load required `angular services/dependencies`, like we load `$scope` service and created instance in function.

#### $scope

[scope](https://docs.angularjs.org/guide/scope) is pre-defined object of angular which has api `$watch` for watching the status of model.

Using scope, we can bind the data between html and js and changes in html will reflect to js and vice versa by using api `$watch`.

scope has hierarchical structure, hence, scope inherit from it's parent scope and able to create child scope.

```
// parent scope
var parent = $rootScope();

// child scope
var child = parent.$new();
```

In our example, we created scope object as `$scope.name` and done binding in view(html page).

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

When `index.html` is loaded, it inject `angular.js`, `app.js` and `controller.js` in browser as we defined script in index.html. Hence `myApp` and `MyCtrl` are available for `index.html`
as `angular.module` is placed at rootElement.
```html
<html ng-app="myApp">
```

#### Directive

Angular has [Directive](https://docs.angularjs.org/api/ng/directive) components as `ng` like `ngBind, ngValue, nclass` and many more.
In this article I explained about `ngApp` and `ngController` directives and will come with other directives in coming article.


#### ng-app

[ngApp](https://docs.angularjs.org/api/ng/directive/ngApp) is angular's `pre-defined directive`.

> If `ngApp` is not placed in html `rootElement` then controller is fail to load, since `controller(myCtrl)` is registered with `ngApp("myApp")`.


#### ng-controller

```html
 <body ng-controller="myCtrl">
```

The [ngController](https://docs.angularjs.org/api/ng/directive/ngController) directive do binding with controller class to the view. This is a key aspect of how angular supports the principles behind the Model-View-Controller design pattern.

##### MVC components in angular

* Model — Models are data assingned to `scope` variable for DOM, and consumed by api `$watch`.

* View — The html template rendered and bind the value on DOM.

* Controller — The ngController directive specifies a Controller class that contains business logic behind the application.

#### Data-Binding

```html
<body ng-controller="myCtrl">
    <input ng-model="name" />
    <p>Hello {{name}} !</p>
</body>
```
`{{name}}` is like variable to store and pass data of `$scope.name` present in the ngController `myCtrl`.

#### ng-model
```html
<input ng-model="name" />
```

[ng-model](https://docs.angularjs.org/api/ng/directive/ngModel) provides two-way-binding feature, it binds value for input, select, textarea with object of `scope`.

#### Angular Expressions - {{Exp}}

Expressions used to get or print the values of variable in view.
Expressions are used to places the scope value for interpolation binding, but fail for functionExpressions,

for Example :
```html
// work for
<p>{{scopeValue}}</p>

// fail for
<button ng-click="{{functionalExpressions()}}">Click me !</button>

// for directive used directly
<button ng-click="functionalExpressions()">Click me !</button>
<input ng-model="name" />
```
 Anything written in `input tag` will observe by scope api `$watch` and will display value of `$scope.name` object in paragraph tag.
```html
<p>Hello {{name}}!</p>
```

#### Source Code
[plunker](https://plnkr.co/edit/5sxudDgiAOGe8hKYjRLK?p=preview) for live example.

#### ScreenShot
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/aeeb2d6b-58b1-4d24-97cc-9c42d0ef7954.png)


This is how we implement angular.module, register ngController to angular module and use of scope variable with controller and view to binding with ngModel in  AngularJS. see you soon with next article.