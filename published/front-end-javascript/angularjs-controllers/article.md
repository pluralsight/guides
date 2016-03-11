Controllers are one of the key components in the MVC architecture of AngularJS. And while it is a good goal to keep your controllers lean, preferring to locate code in the model and services, there is still quite a bit to controllers and how you use them. In this article I will discuss controllers and the various ways to use them in your AngularJS applications. This will include discussion of handling the model, using services, nested controllers and the various ways to connect the controller to a view.

## Controller Basics

In a [previous article](http://blog.pluralsight.com/tutorial-angularjs-mvc-implementation) on the [MVC pattern in AngularJS](https://app.pluralsight.com/library/courses/angularjs-dotnet-developers/table-of-contents), I discussed how the job of the controller is to connect the view with the model. The view handles the visual layout while the model handles much of the business logic. However, because controllers support [the dependency injection system in AngularJS](https://app.pluralsight.com/library/courses/angularjs-patterns-clean-code/table-of-contents), and simple model objects do not, controllers often leverage services to help build up the model, or to handle interactions from the view. 

## Connecting controllers to views

In order to prepare the model for the view, the controller needs to be aware of the view. The simplest way to connect a controller to the view is to statically link the two with a directive in the HTML markup as shown here.

```
<body>
    <h1>Hack Hands - Controllers</h1>
    <div ng-controller="technologyController">
      <label>{{heading}}</label>
  </div>
    <script src="./scripts/app.js"></script>
    <script src="./scripts/technologyController.js"></script>
</body>
```

This directive instructs AngularJS to instantiate the technologyController, defined in the JavaScript file of the same name, and use the model it defines for databinding. That means that binding statements for data or event handlers will try to resolve to the model object setup by the controller when it was created. Given the following controller definition, the “heading” binding will resolve correctly.

```
angular.module('hackApp')
        .controller('technologyController', technologyController);

    function technologyController($scope) {
        $scope.heading = "Choose your technology";
    }
```

The other common way that controllers are connected to a view is using routing. In that case the view and controller are both specified as parameters for a specific route as shown in this simple example.

```
angular.module('hackApp', ['ngRoute'])
    .config(function($routeProvider){
        $routeProvider.when('/mentoring', {
            templateUrl: '/views/technologies.html',
            controller: 'technologyController'
        })
        .otherwise({
            templateUrl: '/views/main.html',
            controller: 'technologyController'
         })
    });
```

With routes the controller still sets up the model for the view in the same way, but the controller itself is linked to the view through the route so there is no need to use the directive in the template markup. The template or view can focus on the binding statements and custom directives assuming the controller will provide the model.

One difference that comes into play with routing is that routes can have parameters and the controller will almost always need access to some or all of those parameters in order to properly load the model. For example, a page listing technologies might provide links to drill down into each technology to find a list of available mentors. The technology in question would be a parameter in the URL necessary for the controller of that view to load the appropriate mentors. Given a URL such as this: /mentoring/jquery, the parameter “jquery” is something that should be passed to the controller.

There are two steps to make this work: declaring the route parameter as part of the route configuration and setting up the controller to receive route parameters. For the first step, the addition of a placeholder in the location parameter when registering a route defines the name and position of the parameter.

```
$routeProvider.when('/mentoring/:tech', {
            templateUrl: '/views/technologies.html',
            controller: 'technologyController'
        })
```

As shown here, the route parameter is included in the URL at the appropriate location and the name is prefixed with a colon (:) to differentiate it from other parts of the path. Using the previous URL example (/mentoring/jquery), the “tech” route parameter now has the value of “jquery”.

The second part of the solution is for the controller to use the $routeParams service to collect the parameters. Each named parameter from the route location will have an accessible value hanging off this object. Continuing the example from before, the “tech” parameter can be accessed via the $routeParams object in the controller code and used to help build the model in whatever way is needed. 

```
function technologyController($scope, $routeParams) {
       if($routeParams.tech) {
           $scope.selectedTech = $routeParams.tech;
       }
 }
```

It is important to note that each navigation to a route results in the controller function being invoked to create the model. This makes sense as each invocation in this scenario is likely for a different parameter and a different set of data should be collected for the model. However, it is important to understand this as it means that attempts to store data locally in the controller would fail upon navigation to another instance of the controller. Additionally, it may make sense in your application design to include some form or caching if you expect users may revisit routes frequently.

## ControllerAs

In a future article, I will talk about scope and managing models, but one aspect of models is the ability to have the controller object itself be added to the scope by name. When you use this option it removes the need to use the $scope object directly inside the controller as you can add properties and methods directly to the controller and bind to those in the view.

```
angular.module('hackApp').controller('modelController', modelController);
 function modelController() {
        var vm = this;
        
        vm.techs = ['JavaScript', 'ASP.NET', 'C#'];
        vm.sortAsc = function(){
            vm.techs.sort();
        };
        
        vm.sortDesc = function(){
           vm.techs.sort();
           vm.techs.reverse();
        };
    }
```

The very first line of the controller function sets a scoped variable to the “this” variable which currently represents the controller itself. Once that scoped variable is created, it is treated as the view model and properties and functions are defined on that model. In this case an array of technologies is defined and two methods to sort the array in ascending or descending order. (Please do not pay much attention to the sort functions. There are many better ways to sort an array, but that is not the point of this example.)

In order to use this controller model, the view must be updated and the route definition or controller directive must be modified as well.

```
<div ng-controller="modelController as model">
        <button ng-click="model.sortAsc()">Asc</button>
<button ng-click="model.sortDesc()">Desc</button>
        <br />
        <ul>
            <li ng-repeat="tech in model.techs">{{tech}}</li>
        </ul>
    </div>
```

In the example above, the controller directive has used the pattern of declaring the controller to be used and providing the name by which it will be referred in the view. In this case the modelController from above will be referenced by the name “model”. Then in the ng-click event handler directives and the ng-repeat directive, the binding statements are based on the “model” reference to the controller. Behind the scenes the controller is simply added the $scope with a property named “model”. Considering that, the binding syntax should make complete sense.

When using routes, the controllerAs parameter is passed to the “when” or “otherwise” function to provide the name that will be used in the view to refer to the controller. 

```
$routeProvider.when('/model', {
            templateUrl: '/views/model.html',
            controller: 'modelController',
            controllerAs: 'model'
        })
```

The views used with the route would look identical to the previous example without the controller directive. This approach simplifies the code in the controller and makes the databinding in the view more explicit. In essence, this approach removes one of the dependencies of the controller, which leads us into a discussion of dependencies and how to manage them.

## Working with dependencies

When controllers do the work to prepare the model for the view, or handle interactions from the view, they most often will work with services to do the actual work. One of the main design goals of AngularJS is to use dependency injection to eliminate the need for the direct creation of these services. This not only makes unit testing controllers easier (as I will show shortly) but also simplifies the code and makes it easier to maintain. There are several different ways to manage both the declaration and resolution of the dependencies for a controller.

The primary model for working with dependencies is to pass an array to the constructor function of the controller that lists the dependencies by name and includes a function to be invoked that takes those dependencies as parameters such as in this example.

```
angular.module('hackApp')
        .controller('mentorController', ['$http','$scope',mentorController]);

function mentorController($http, $scope) {
        $scope.mentors = ['Matt', 'Bob', 'Alice', 'Raj', 'Priscilla'];
 }
```

Notice in this example that the controller function takes the name of the controller followed by an array which includes a list of dependencies and the controller function. Also note that the order of the dependencies is the same in the declaration and in the parameter list. Note that it is also possible to exclude the array listing the dependencies and have them inferred from the controller function parameter names. This assumes the parameter names match the service names and also makes your code unsafe for minification so is generally not a recommended approach.

Another approach applies the list of dependencies to the controller function via a property which provides another minification-safe option. This option is preferred by many developers as it makes the controller function itself cleaner and easier to read, especially when the list of dependencies grows large.

```
angular.module('hackApp')
        .controller('technologyController', technologyController);

    technologyController.$inject = ['$http', '$scope', '$routeParams'];
    function technologyController($http, $scope, $routeParams) {
     //implementation omitted
}
```

In this example the technologyController is declared as a controller function without listing the dependencies. Then the $inject property is set on the function listing those dependencies. As in the previous method, the dependency list must be in the same order as the parameter list so you will have to keep those in sync if you add, remove, or reorder the parameters.

Because both of these methods list the dependencies as strings, the values will not be minified when your code is processed for minification. It does mean more work in listing the dependencies twice and keeping the list in synch. Because of this, an open source project, ng-annotate, was created to simplify this process enabling you to simply define the parameters to the function and have a pre-processing tool create the $inject property for you on the controller. This tool does require that you add a comment or attribute to your controller functions but that is a consistent marker that is not tied to the parameter list. As you begin building applications it is worth taking a look at ng-annotate and working it into your tool chain to simplify the JavaScript code for your controllers.

When using routing, there are certain scenarios where you may want to provide dependencies to your controller as promises rather than having them resolved directly by the AngularJS injector. For example, if you have services that will asynchronously load data, or if you need to make decisions about whether to display the view, and therefore invoke the controller, based on some route parameters or data returned from a dependency. For those cases, the route provider has a resolve property you can set that allows you to resolve the dependencies and create promises that are all resolved before moving to the controller. 

```
.when('/model', {
            templateUrl: '/views/model.html',
            controller: 'modelController',
            controllerAs: 'model',
            resolve: {'modelService':modelServiceWrapper}
```

In this bit of code from the route provider configuration, the dependency called ‘modelService’ will be resolved with the function modelServiceWrapper shown here.

```
function modelServiceWrapper(){
        //replace this pseudocode with an asynchronous request to the server
        return {techs: ['JavaScript', 'ASP.NET', 'AngularJS']};
    
}
```

In this example I have used simple test code where you would normally put your service logic to load items from the server or resolve other required data for your controller. The controller function can then declare a dependency on the modelService and use the resulting object in its setup logic now with the promise resolved, meaning the data has been loaded already.

```
angular.module('hackApp')
        .controller('modelController', ['modelService', modelController]);

    
    function modelController(modelService) {
        var vm = this;
        
        vm.techs = modelService.techs;
    }
```

The benefit of this approach is that all the promises needed by the controller are resolved before the controller needs the result. This enables decision making in the routing to cancel routes or apply other logic based on the context and the promise results.

## Nested Controllers

It is a good practice to make controllers single purpose in their work. However, that means that sometimes controllers might need to work with another controller in partnership to generate and work with an entire view in the page. For example, with master-detail views one controller may be responsible for the master context while a separate controller handles the detail. In those case you can nest controllers.

In my example I have a view that shows a particular technology then lists the mentors available for that technology along with their details. In a given view I can nest the mentorController inside the technologyController to provide the full view model and handlers. In this way my controllers are usable on their own in different views but can coordinate with each other when used together.

What is really happening when controllers are nested in this way is that a new scope gets created for each controller and the child scope inherits from the parent. That topic will be covered in greater detail in the next article where I cover scopes and models in depth. For now, it is important only to know that you can use controllers in this way to keep them single purpose.

One temptation may be to build logic into your child controller that refers to the parent context but that tight coupling does not work well in almost all scenarios. In the next article I will show how to use services and other techniques to use nested controllers/scopes without taking those dependencies.

## Testing controllers

Several times in this series I have mentioned that AngularJS leverages dependency injection heavily and that testability was one of the key benefits. I would be remiss then to not talk about testing controllers in a solution. When testing, dependency injection allows for supplying services to controllers that are more suitable to unit testing scenarios and enable result verification and mocking or stubbing. AngularJS provides the ngMock module to help with unit testing AngularJS applications.

The ngMock module provides several things that aid in the testing process. The first are the “module” and “inject” methods on the “mock” object to enable loading modules in the testing framework and injecting services into the application. The second is a set of mock services that provided testable alternatives to the runtime services. These include an HTTP backend service, logging service, and exception handler just to name a few. These replacement services make testing easier by allowing for controlled inputs to controllers and expectation checking of the result of controller functions executing.

For example, the $httpBackend mock allows for responses to be predefined for certain requests enabling consistent inputs for a controller when requests are invoked. It also allows expectations to be defined so that errors are thrown if expected requests are not executed as part of the controller testing. The $log mock captures logging output and makes it available for expectation testing.

Most AngularJS developers use either Mocha or Jasmine as a unit testing framework while Karma is a test runner created by the AngularJS team to make running unit tests easier. Using these tools along with the ngMock module provides the core framework needed to write complete unit tests. There are other tools that are commonly used for more advanced mocking or stubbing capabilities based on the personal preferences of developers and their teams.

In the code below, several of the ngMock components are in use in a Jasmine test specification. The ngMock module is included in addition to the AngularJS core and routing modules and the application itself of course.

```
describe("technologyController", function() {
    var $httpBackend, techRequestHandler, $controller;
    
  beforeEach(function(){
      module('hackApp');
  });

  beforeEach(function() {
      inject(function($injector){
        $controller = $injector.get("$controller");
        $httpBackend = $injector.get('$httpBackend');
     
        techRequestHandler = $httpBackend.when('GET', '/techs')
                            .respond(['Angular', 'JavaScript', 'HTTP', 'HTML']);
      });
  });
```

The “module” and “inject” methods of ngMock are used to get a reference to the module and inject the $httpBackend mock and get a hold of the $controller object that will allow for creating a controller instance in the next step. Notice that the $httpBackend is setup using the when().respond() method chain to create an expected input to the controller when an HTTP GET request is invoked. Because this data is known, the results can be tested after the controller functions have been invoked.

```
  describe("Create technology based on route information", function(){
     it("should create the list of technologies when no route parameters are present", function() {
        var $scope = {};
        var $routeParams = {};
        var controller = $controller('technologyController',
              {$http:null, $scope: $scope, $routeParams:$routeParams });
        
        expect($scope.selectedTech).toBeUndefined();
        expect($scope.techs.length).toEqual(3);
    });
```

Once the mocks are setup, each test can create other inputs such as a $scope and $routeParams objects for inputs and use the $controller object to create the controller instance using dependency injection where needed. After the controller function is invoked the $scope can be inspected to test expectations. This particular example doesn’t make use of the $http service or mock, but the next example does.

```
describe("Load list from web", function(){
     it("should populate the webTechs property", function() {
        var $scope = {};
        var $routeParams = {};
        
        var controller = $controller('technologyController', 
          {$scope: $scope, $routeParams:$routeParams });
        
$httpBackend.flush();
        expect($scope.selectedTech).toBeUndefined();
        expect($scope.webTechs.length).toEqual(4);
    });
  });
```

In this case the $http parameter is not supplied to the controller thus it will be supplied using dependency injection and the $httpBackend mock will be supplied to the controller. Note that after the controller function has been called, the flush method is called on the $httpBackend mock to ensure that the controller gets the results and the test can finish. Again the results on the $scope can be tested to ensure that the proper data was returned and the controller has setup the model correctly.

This is just a simple example of how AngularJS make testing of JavaScript code easier with dependency injection and a solid starting point for mocking critical services during unit tests. As I mentioned, there are several other frameworks available to take testing further and enhance the running of tests, creating of stubs, and results verification.

## Where are we?

In this series we have covered what AngularJS is and how it implements the MVC pattern. In this article we have begun to dive deeper into the components of that MVC architecture with the controller. In the next two articles we will go deeper into models and views before finishing up with more advanced topics around filters and directives.

Author of this tutorial is [Matt Milner](http://blog.pluralsight.com/author/matt-milner).
