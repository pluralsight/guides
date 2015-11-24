In a [previous article](http://blog.pluralsight.com/tutorial-angularjs), I discussed how AngularJS fits into web development as a framework for building rich client side web applications. One aspect of that framework is that it follows the Model-View-Controller (MVC) pattern for user interfaces. In this article I will examine how the MVC pattern is implemented in AngularJS and how that drives your application development.

### Model view controller
 
In the Model-View-Controller pattern, the different aspects of the application are broken into components to separate the responsibilities. The Model contains the data and logic, the View contains the visual layout and presentation, while the Controller connects the two together.

[![Screen-Shot-2015-11-24-at-8.45.08-AM](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.45.08-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.45.08-AM.png) 

Using this pattern, the various components have a relationship defined by their interactions. The model is only aware of itself. The view is generally aware of the model as it is responsible for rendering data contained in the model and invoking actions (methods) on the model. The job of the controller is to create and populate the model and hand it over to the view. The controller needs to know about the model and how to resolve the view and provide it the model. 

### Angular MVC
In AngularJS the MVC pattern is implemented in JavaScript and HTML of course. The view is defined in HTML while the model and controller are implemented in JavaScript. There are several ways that these components can be put together in AngularJS but the simplest form starts with the view. 

#### Views
The view in an AngularJS application is created with HTML. In a very simple example that HTML can all exist inside a single page. Almost all applications will quickly move past the single view model however, in which case the fragments of HTML that make up the view will be stored in individual files. For example, take this HTML page as a starting point:

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hack Hands Angular - Demos</title>
  <meta charset="utf-8">
</head>
<body>
  <div id="messageTitle"></div>
  <div id="message">Hello World</div>
</body>
</html>

A Simple HTML Page Without AngularJS
```
It would be easy to consider this entire HTML document to be the view at this point since there is only one. However, because this is a Single Page Application, only a portion of the page represents the current view. In this case the contents of the body represent the view while the HTML and HEAD elements make up the container for the individual views. 

In AngularJS it is also popular to mark a specific element in the page as the view giving you more control over which portions of the page make up the container and which make up the changing view. 

```html
<!DOCTYPE html>
<html>
<head>
  <title>Hack Hands Angular - Demos</title>
  <meta charset="utf-8">
</head>
<body>
<h1>Hack Hands Angular Demos</h1>
  <div ng-view>
    <div id="messageTitle"></div>
    <div id="message">Hello World</div>
  </div>
</body>
</html>

Explicit Declaration of the View

```

In this example a static heading has been added to the body of the document and a DIV has been added to act as the container for the view and is marked with the `ng-view` attribute, also known as a directive. As the application progresses to multiple views this will have significance. As it stands it does not really change anything with a single static view but it is good to understand how AngularJS will identify the location in the document for the view.

This HTML is not yet a functioning AngularJS view. There are a couple of steps required to include and bootstrap the AngularJS framework.

1. Reference the AngularJS framework.
2. Define the AngularJS application.

Referencing the AngularJS framework is a simple matter of adding a script tag referencing the framework script on a Content Delivery Network (CDN):

```html
<script>src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>
```

*Referencing the AngularJS Framework on the Google CDN*

When this script loads it will examine the page and look for directives such as the `ng-view` used previously. To bootstrap the application an `ng-app` directive is used, typically on the HTML element itself:

```html
<html ng-app="hackApp">

Bootstrapping the Angular Application
```

When AngularJS loads it will see this directive which will cause it to look for a module named “hackApp”. Modules are a key component of AngularJS that will be covered in more detail in a later article. For now it is enough to know this view needs to be able to reference a module representing the AngularJS application. This single line of code defines module named “hackApp” which will be used to connect controllers to the application as well.

```javascript
var hackApp = angular.module("hackApp", []);
scripts/ngHackApp.js
```
The script needs to then be referenced in the index.html page so the `ng-app` directive and the module can be connected. 

```html
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>
<script src="scripts/ngHackApp.js"></script>

Referencing the AngularJS Application

```

Viewing the page in a browser results in a simple view which shows the heading and the static message contained in the view. 

[![image_3](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.09-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.09-AM.png) 

Now that the application has a view, it needs a controller and model to actually make it more than just some HTML markup. The controller will be responsible for connecting the view and model for now, but will also handle resolving other dependencies later such as services to consume HTTP resources. 

The simplest way to provide a controller for the view is to use a directive in the markup as with the `ng-app` and `ng-view` directives earlier. Of course the directive needs the corresponding JavaScript implementation to function correctly.

```html
<!DOCTYPE html>
<html ng-app="hackApp">
<head>
  <title>Hack Hands Angular - Demos</title>
  <meta charset="utf-8">
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>
  <script src="scripts/ngHackApp.js"></script>
</head>
<body ng-controller="indexController">
  <h1>Hack Hands Angular Demos</h1>

  <div ng-view>
    <div id="messageTitle"></div>
    <div id="message">Hello World</div>
  </div>
</body>
</html>

Adding a Controller with a Directive
```

### Controllers

The controller for a view is responsible for pulling together the model used by the view and setting up any corresponding dependencies needed to render the view or handle input from the consumer of the view. In AngularJS the controller is resolved by name, from the `ng-controller` directive shown above or routing configuration shown shortly. The controller function must be registered with the AngularJS application in order to be resolvable by name. One way to register the controller is to manually add it to the JavaScript variable representing the AngularJS application. 

```javascript
var indexController = hackApp.controller("indexController", function ($scope) {
    // controller logic goes here
    }
);

Adding a Controller to the Application (scripts/ngHackApp.js)
```

Using the controller function on the application allows for registering a controller with a name and a constructor function. The name used here must match the name used in the `ng-controller` directive or in the routing configuration. The function parameter is the main entry point for the controller at runtime and will be called by the AngularJS runtime. In this function the controller sets up the model and readies it for the view.  

In order to do that work the controller may require various services. AngularJS uses Dependency Injection (HREF LINKTO:https://en.wikipedia.org/wiki/Dependency_injection) to provide controllers and other objects with the required dependencies. The use of this pattern both simplifies the development of the dependent objects and makes those particular objects or functions more easily testable. This is one of the primary design goals of AngularJS: testability. In the example above a parameter named `$scope` is declared in the constructor function for the controller. The naming convention using the dollar sign `$` at the start of the name indicates that this is a well-known dependency in AngularJS. You can create your own dependencies and refer to them using a string value for the registered name. 

In this simple example the controller only has one dependency and that is the `$scope` object. The `$scope` parameter refers to the current scope or context for the view. In other words, the `$scope` is the model used by the view. 

### Models

The model for a controller contains the data to be displayed as well as data to be collected in any input fields or forms. In addition, models may contain functions that are invoked based on user input or other activities such as clicking a button or making changes to the model data. 
The `$scope` object injected into the controller can be used as the model directly. For the simple example above the following code can be used to declare and set a value on the model.

```javascript
var indexController = hackApp.controller("indexController", function ($scope) {
    // controller logic goes here
    $scope.message = "Hello Hacking World";
    }
);

Adding information to the model (scripts/ngHackApp.js)
```

Now that the model `$scope` has data, it can be used in the view. AngularJS provides a rich declarative data binding experience. That means elements in the view can have their value bound to the model. Because the binding in AngularJS is two-way the value can be changed by the view (e.g. changing the value in a text box) or in the model (e.g. changing the model in a function when a button is clicked). 
To finish the example an updated view to show the value from the model looks like this. 

```html
<div ng-view>
  <div id="messageTitle"></div>
  <div id="message">{{message}}</div>
</div>

Updated view with data binding
```

The highlighted markup is a simple binding statement indicating that the value of the “message” property on the `$scope` should be rendered at this location; in this case, inside the DIV.
Combining the controller code that sets up the model with the declarative binding in the view gives us an updated, albeit not much different on the surface, output. 

[![image_1](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.29-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.29-AM.png) 
Updated view using databinding

Now the view is not much different from before, but it displays information from the model instead of hard coded text. In a more complete application there would be much more data in the model including perhaps lists of items that could be displayed in the view. This simple example does show the interaction between the model, view, and controller. This model will play out similarly in more complex applications. The differences will be in the complexity of the model, more advanced databinding and view elements, plus the use of additional services in the controller.

### Leveraging Binding, Models, and Dependency Injection

The example can be extended to show more usage of the two-way databinding, more advanced models, and using dependency injection. First, updating the view to show two-way databinding simply involves adding an input element bound to the same `$scope.message` property.

```html
<div ng-view>
  <div id="messageTitle"></div>
  <div id="message">Hello {{message}}!</div>
  <form>
    <input type="text" id="name" ng-model="message">
  </form>
</div>

Updating the view to show two-way databinding

```

Now viewing the page and changing the data in the input box will change the value of the message automatically. The input is tied to the model using the `ng-model` directive instead of the curly brace syntax used for the display. Using this approach allows for setting both the model and an initial value for the HTML element. This opens up choices that a developer will have to make about whether the initial value is defined in the HTML markup or set on the model in the controller code. For unit testing the controller code it will likely make more sense for those defaults to happen in code so they can be tested, but again, this approach gives the developer options.

[![image_1](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.41-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.41-AM.png)  
Binding causes display to update as model is edited in the input box

The model can also have functions defined which can be bound to events in the view. For example, a button click can be configured to invoke a particular function on the model. Using the `ng-click` directive on a button declares the binding allowing AngularJS to setup the appropriate event listener and connect that to the function on the model.

```html
<form>
  <input type="text" id="name" ng-model="message">
  <button ng-click="popupGreet()" value="popup">Greet</button>
</form>

Connecting events to functions on the model
```

This example expects the $scope to have a function defined with a name of popupGreet. Then clicking the button should invoke the function. 

```javascript
var indexController = hackApp.controller("indexController", function ($scope, $window) {
  // controller logic goes here
  $scope.message = "Hello Hacking World";

  $scope.popupGreet = function () {
    $window.alert("Hi there " + $scope.message);
  };
});

Updating the model with a function
```

The popupGreet function is added directly to the `$scope` object and can now be invoked by the click handler. In this function the goal is to show an alert with a message to the user. Rather than try to code directly to the browser window it is advised to use the $window service. This enables easier testing of the controller because any test written for the controller can provide a testable object for the $window parameter and validate its state after the test. 
One nice thing about this pattern is that the controller has no responsibility for creating the `$window` service. Rather it declares the `$window` service as a dependency, by including it in the constructor function in this case, and AngularJS takes care of injecting that dependency at runtime. 

[![image_1](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.54-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.46.54-AM.png) 
Using injected dependencies to interact with services

Routing
Defining a page with the controller and view identified in the markup is helpful to understand how all the pieces of the MVC model fit together, but it is not a realistic example for single page applications. Applications leveraging AngularJS will have several views and corresponding controllers and models with a need to navigate between those views. This navigation and the corresponding management of the controllers and views can be handled in AngularJS by taking advantage of the routing module. 
Routing is not part of the core AngularJS framework and must be added separately. This can be done by adding it as a depending in a variety of package managers such as Bower or NPM or by adding the JavaScript file from a CDN as shown here.

```html
<head>
  <title>Hack Hands Angular - Demos</title>
  <meta charset="utf-8">
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular.min.js"></script>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.4.5/angular-route.min.js"></script>
  <script src="scripts/ngHackApp.js"></script>
</head>
<body>
  <h1>Hack Hands Angular Demos</h1>

  <div ng-view>
        
  </div>
</body>

Index.html referencing the Angular Routing module
```

Because routing will take care of selecting and connecting the appropriate controller and view based on the URL, the `ng-controller` directive can be removed from the BODY element and the view placeholder is empty. The main view and any other views will now be defined in individual HTML files. 

```html
<div id="messageTitle"></div>
<div id="message">Hello {{message}}!</div>
<form>
  <input type="text" id="name" ng-model="message">
  <button ng-click="popupGreet()" value="popup">Greet</button>
</form>
<br>
<a href="#/techs">View technologies</a>

/views/main.html with a link to the technologies view
```

The template for the main content is identical to what was removed from the index page with the exception of a hyperlink to a new view that will show a list of technologies available for assistance on HackHands. The hyperlink starts with the hash or pound symbol which is defined in the HTML specifications as way to link to named anchors within a page. By forming links in this way the browser will not try to navigate to another page instead allowing AngularJS to react to the link request and load a different view. 
The technologies view is another simple view with a corresponding controller and model that shows a list of technologies. 

```html
<h3>Technologies</h3>
<ul ng-repeat="tech in techs">
  <li>{{tech}}</li>
</ul>
<br>
<a href="#">Home</a>

/views/techs.html
```

This views showcases another feature of databinding in the view: handling lists of items. The ng-repeat directive is used to repeat over a set of items in the model emitting view elements for each item and allowing for binding syntax that attaches to the individual item in the collection. In this case the model has a property named “techs” that is an array of technology names. In the view that list will be enumerated and each item will generate a list item `<li>` in the output. This view also has a link back to the main page by providing the same page link with no target. 

```javascript
var techController = hackApp.controller("techController", function ($scope) {
    $scope.techs = ["Java", "Go", "Node.JS", "JavaScript", "HTML", "Ruby on Rails"];
  }
);

/scripts/ngHackApp.js with the additional techController defined
```

The only dependency for this controller is the $scope which is populated with the list of technologies that can then be rendered in the view as a list. Here the model contains an array of strings but it could certainly contain an array of objects so that each item rendered in the template can have rich content and data displayed.

Note that both views are simply partial views, or bits of HTML that are loaded into the container or host page within the view element. This is most evident visually by the fact that the header content does not change when navigating pages and is only contained in the container markup. 

With two controllers and views the router is responsible for controlling the connections between them and handling the links. In order to manage the routes, the router must first be configured with the information that maps the URL used in links to a specific controller and view.

```javascript
var hackApp = angular.module("hackApp", ["ngRoute"]);

hackApp.config(['$routeProvider', function ($routeProvider) {
  $routeProvider.when('/techs', {
    templateUrl : "/views/techs.html",
    controller : "techController"
  }).otherwise({
    templateUrl : "/views/main.html",
    controller: "indexController"
  })
}]);

/ngHackApp.js with route configuration
```
The first thing to notice is that the creation of the application now includes a dependency on the ngRoute module. By declaring this dependency, the routing module is now loaded into AngularJS and initialized to be able to handle the links used in the templates.

Before an AngularJS application runs and processes the markup in the host page, executes controllers and displays a view, it goes through a configuration process. Like many other aspects of AngularJS, the configuration process supports dependency injection. In the example above the `$routeProvider` is injected into the config function. The implementation sets up the routes using the when function on the route provider to map each URL to the corresponding controller and template (view). The otherwise function provides a fallback or default controller and view if a match to the current URl cannot be located. 

[![image_1](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.47.36-AM.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/Screen-Shot-2015-11-24-at-8.47.36-AM.png) 
Navigating between views using hyperlinks

The routing module connects the various views together. Because each unique URL points to a specific view and controller, the user will be able to bookmark specific views and return to them.

#### Where are we?
In this article the MVC model as implemented by AngularJS was covered to provide the foundation for future articles. Those articles will go into depth on models, views and controllers to provide more detail on each item in isolation and the capabilities and requirements of each.

Learn more about the author of this tutorial: [Matt Milner](https://www.pluralsight.com/authors/matt-milner)
