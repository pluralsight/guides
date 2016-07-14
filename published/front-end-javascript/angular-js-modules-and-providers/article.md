AngularJS has many different components used to make an application. It is important in any project using JavaScript to be able to manage the code in a reasonable way. AngularJS uses modules, like many modern JavaScript libraries, to encapsulate functionality such as controllers, directives, filters and services. AngularJS is also highly focused on dependency injection and therefore has infrastructure in place to help instantiate the many objects necessary in your application. When you define a controller, for example, and list the dependencies in the constructor function, there are Providers which are responsible for instantiating the dependent objects used by the controller. This model allows AngularJS to correctly create and inject items into your code and provides a powerful means for you to control that object creation. In this article I will examine the module pattern and the different types of providers that can be used in an AngularJS application.


# Understanding Modules

Modules define a unit of code in your Angular application and contain the code you write such as controllers, directives, filters, and services. In simple examples, such as those used in previous articles, it is common to see an application defined as a single module.

```javascript
angular.module('mainModule',[])
 .controller('mainController', mainController);

function mainController(){
 var vm = this;
 vm.hello = "hello";
};
```
In these examples the controllers are all added to the same module. While that works with simple examples, as an application grows and has many controllers, sub sections, custom filters, directives, and services, it makes more sense to break those units of code up into their own modules. Consider that many AngularJS applications will also be worked on by a team of developers. Having the code spread out not only into individual modules, but also physical files, will make working on the application across the team much easier.

The module function can either create a module, or retrieve one. When creating a module, it is given a name and a list of dependencies. To retrieve a module, the function is called with just the name. Using this pattern allows for creating a module out of multiple files. For example, each controller can be defined in its own file and still added to the same module. 

```javascript
angular.module('mainModule',[]);
angular.module('secondModule',[]);
main.js

angular.module('mainModule')
 .controller('mainController', mainController);

function mainController(){
 var vm = this;
 vm.hello = "hello";
};
main-controller.js

angular.module('secondModule')
 .controller('secondaryController', secondaryController);
 
function secondaryController() {
 var vm = this;
 vm.hello = "multiple files"
}
secondary-controller.js
```

Notice that in the main.js file the two modules are created with an empty dependency list. In the controller files each module is then retrieved using the module function with the module name as the single parameter. The controllers can be attached to the module after it has been retrieved. Using this pattern enables more flexibility in where the code is written while maintaining the logical modules.
 
The use of dependencies provides additional power to this model as one module can simply declare that it depends on another module. AngularJS will then take care of loading that dependency making sure it is available for use when the depending module needs it. 

```javascript
angular.module('mainModule',['secondModule']);
angular.module('secondModule',[]); 
main.js (updated with dependencies)
```

In this updated main.js file, the main module declares a dependency on the second module. It is important to note that the second module is not declared until after the first module. Often in JavaScript the order of declaration is important as items need to be available before being referenced. When the main module is loaded the second one must be available, but not before then. Again this provides more flexibility in the declaration of the modules.

Combining the idea of multiple modules with the dependency support, an application can be built from linking those modules together. Often this is accomplished with a main module that has the singular purpose of linking in the other modules as dependencies.


In this example the “app” module serves to tie together the other modules used in the application. The core modules include things like services or user interface elements like filters and directives that might be used across modules or even across applications. As shown here, the core modules are listed as dependencies of the main app module and the individual modules where they are used. A different approach would be to only include those dependencies in the modules that use them. I’ve shown both here simply to point out the options.
 
I mentioned the idea of splitting application code into several different files. That may strike some as poor way to manage JavaScript thinking that all those files will have to be downloaded by the browser. However, it is common when working with JavaScript clients to use tools for bundling and minification of code. So having a module defined in 5, 10, or 15 different files will not matter at runtime as they will all be part of one compact file for download to the client. If you are not using these types of tools with your AngularJS application, you should spend some time investigating them and add them to your build process.

<img src="https://raw.githubusercontent.com/pluralsight/guides/master/images/fad9503d-c033-462c-83bf-b2e2d10a1d5c.png" width="800" height="600" />


## Bootstrapping

Once all the modules are defined, AngularJS needs to be started and pointed to those modules. The manual way to do this is using a function named “bootstrap” that connects AngularJS to both the DOM (document object model) and the module or modules containing your application code.

```javascript 
angular.bootstrap(document, ['mainModule'])

```

### Manual bootstrapping

With manual bootstrapping AngularJS needs a reference to the element in the DOM that will be host to the application and a list of dependencies. In the example above, the document is used as the parent to the application and the main module is declared as a dependency. Remember that the main module can, and does in our example, contain its own dependencies as well.

A shortcut used in place of the manual bootstrapping is to use the ng-app directive in the main HTML page hosting the application.

```html
<html ng-app="mainModule">
```

### Declarative bootstrapping

When using this declarative model, the directive is placed on the element that will host the application (the first parameter in the manual approach) and is provided the name of the module to use as the starting point (the second parameter in the manual approach). 
Using either the manual or declarative bootstrapping method will initialize the application and load the modules. Once the modules are loaded and begin being utilized, they will start to require the dependencies that have been declared. That is where providers come into play.

## Providers

AngularJS provides a strong dependency injection model to simplify development and testing. One key aspect of that model is the notion of providers. Providers are the bits of code that are responsible for instantiating or providing instances of the various dependencies used by your application. For example, to use the $http service or $cookies service in your application, some code has to provide an instance of those services. That is the responsibility of the $httpProvider and $cookiesProvider respectively which are part of the AngularJS framework.

Providers can create many types of objects including native JavaScript types like strings and dates as well as custom object types or services. Adding controllers to a module is an example of a provider model as a function must be provided to define the controller. For other types of objects, function and values used in an application, custom providers can be written.

There is a basic pattern for creating a provider that provides the most flexibility but also requires the most work. Using this model you define a function and include a $get property that will be invoked by AngularJS to create your object. It is important to keep in mind that all providers create singletons so they will only be asked to provide the object once. After that, the object is stored in the AngularJS injector service where it can be looked up when needed to fulfill a dependency declaration. 

```javascript
function calendarProvider() {
 var cal = "en.usa";
 this.setCalendar = function(calendar){
  cal = calendar;
 }; 
 this.$get = ['calendarToken', function(calendarToken){
  return new LocalCalendarService(cal);
 }];
}
angular.module("mainModule")
 .provider("calendar", calendarProvider)
```

In this provider implementation the object has a method, setCalendar, that can be used to configure the provider. This type of configuration enables the provider to be more flexible and useful in different applications or environment. The $get property is the core of the provider implementation. Notice that it supports dependency injection of other providers with a list of dependencies and then has a function responsible for the creation of the object being provided. This function can use the dependencies as shown, as well as local variables in the provider, such as the “cal” variable shown, that enables configuration.

In this example the function returns a custom object but it could also return a string, date, function, or other simple type. In the final two lines of code the provider is added to a module. Remember that modules are the containers for all aspects of code in an AngularJS application and contain not only the controllers, directives and filters, but also the providers themselves.

In order to use the object provided, a controller or other dependency enabled type can declare a dependency on the registered name of the provider as this example shows with a controller.

```javascript
angular.module('mainModule')
 .controller('mainController',['calendarService', 'calendar', mainController]);
```

As mentioned, this is the basic pattern for a provider and comes with certain benefits including support for dependencies and the ability to be configured. I will talk more about configuration shortly which will help determine if that benefit is required in an application. For many situations though, this is more work that should be required to create an object. For that reason, AngularJS provides several methods on a module which provide shortcuts to create certain types of objects. Each of these methods is simple wrapper over an actual provider created behind the scenes, and each have slightly different characteristics and features.

### Service provider

The Service provider allows for the creation of a custom object using the constructor pattern. In other words, if you have an object that can be created using the “new” keyword, with or without parameters, then you can use a shortcut to create that service. For example, the following service object retrieves public holidays from a google calendar and invokes a callback with the results. The service has dependencies as indicated by the parameters in the constructor function.

```javascript
function CalendarService(calendarUrlFactory, $http) {
 this.getHolidays = function(cb) {
  $http.get(calendarUrlFactory)
   .then(function(results){
    cb(results, null);
   },
   function(error){
    cb(null, error);
   }
  );
 }
}
```

Rather than create a full provider implementation for this object, the shortcut service method can be invoked on the module.

```javascript
angular.module("mainModule")
 .service('calendarService',['calendarUrlFactory', '$http', function(calendarUrlFactory, $http) {
  return new CalendarService(calendarUrlFactory, $http);
 }] );
```

The service method takes a name for the service, a list of dependencies and a simple function to create the object. This code replaces the provider code shown earlier to create the object. Using this pattern still provides support for dependency injection into the service as shown in the example where the calendarUrlFactory and $http service dependencies are listed. However, one drawback to this simpler model is that it does not allow for configuration of the provider before the service is created.

Once this service provider has been registered with the module it can be declared as a dependency in other modules and their components.

### Factory Provider

In some cases the object returned by a provider is not a custom object but instead a native type such as a string, number, or function. In those cases a factory provider can be used to create the object. This provider is much like the service provider but does not return a custom object. 

```javascript
angular.module("mainModule")
 .factory("calendarUrlFactory", ['calendarName', 'calendarToken', function(calendarName, calendarToken){
  return calendarName + "/events? maxResults=10&key=" + calendarToken;
 }]);
```

In this example the factory method is invoked on the module to register a function to create a computed calendar URL. The function has two dependencies that are used to calculate the value at runtime. This provider type enables the same shortcut over the more verbose provider model and includes support for dependencies. The main difference between this provider and the service provider are in the types that are returned. You can see an example of this factory being used a dependency in the previous example where the service uses it internally to perform some work.


### Value Provider
In some cases there may be a need to simply provide a value throughout an application from a centralized code location. For those cases the value provider makes is simple to register a value with the injector which can then be used throughout the application. 

```javascript
angular.module("mainModule")
 .value("calendarName", "https://www.googleapis.com/calendar/v3/calendars/. . .");
```

Here the value method on the module registers a simple string with the injector. Elsewhere in the application a dependency can be declared on the “calendarName” provider. The object provided for that name will be the value that was registered using this method. The previous example of a factory provider used a dependency on this provider to get the calendar name value in order to create the full URL. Using this simple provider type enables putting the important values for the application in one location in the code. No more magic strings spread in that code or need for a service that wraps the configuration values.

One important distinction between this provider type and others is that the value provider does not support dependencies. A value provider is expected to return an object on its own without the support of other services so it is most useful for primitive types and functions.
 
### Constant Provider

The final way to create a provider is the constant method on the module to define a constant value as shown here.

```javascript
angular.module('mainModule')
 .constant("calendarToken", "YOUR API TOKEN ");
```

Like the value provider this provider registers a simple value with the injector to make it available throughout the code. The natural question is why this provider is needed at all and what features is supports that the value provider does not. The short answer is that, unlike the value provider, the constant provider is available at configuration time. The longer answer requires a discussion of provider configuration and the runtime process of loading providers and services.

### Provider and Service Lifecycle

When the bootstrap process runs, an AngularJS application goes through two stages: configuration and run. During the configuration process each provider is instantiated, then each module is configured by calling the config method if it exists. It is in the config method that providers following the full provider model can have functions invoked to configure them.

```javascript
angular.module('mainModule',['secondModule'])
 .config(['calendarProvider', function(calendarProvider){
  calendarProvider.setCalendar("en.usa");
 }]);
```

The config method on the module accepts a list of dependencies like services do but in this case the list is of providers which are then passed to the function. This all happens before the modules are run and therefore services are not available during this phase because they have not been created yet. Notice that none of the other provider types are able to be configured at this time.

However, during the configuration phase the constant providers are available for use. This means that the functions or values registered through constant providers can be used to configure the other providers as well as be used at runtime by services, controllers, etc. For example, assuming the calendarProvider was updated to include a setToken method, the following configuration code can be used.

```javascript
angular.module('mainModule',['secondModule'])
 .config(['calendarProvider','calendarToken', function(calendarProvider, calendarToken){
  calendarProvider.setCalendar("en.usa");
  calendarProvider.setToken(calendarToken);
 }]);
```

The config function now has a dependency on the calendarToken constant provider which allows it to be used to pass the token to the calendarProvider upon configuration. This is the big benefit of the constant provider: being accessible at configuration time and run time.

#### Choosing a Provider Model

To choose between providers models there are a couple of key decision points that will help including the type of object being returned and the requirements around the configuration cycle of the AngularJS application. The flowchart below is one example of a decision process to pick a provider type.


<img src="https://raw.githubusercontent.com/pluralsight/guides/master/images/e2941837-41f3-449f-8038-6b2749ceb0b3.png" width="500" height="600" />

#### Module and Providers Together

To pull this all back together, modules are the containers for the code in an AngularJS application. As such, they contain controllers, services, and other code. Modules and services can declare dependencies which are resolved by Angular’s injector service. To populate the injector service with singleton instances of services, functions, and values; providers are created and registered with the injector through methods on the module. Each pattern other than the core provider pattern is a simplification for certain cases to make writing and testing code easier.

Another thing to note is that the pattern for creating controllers is the same as the factory provider model. However, there is a difference in how objects created using the two different patterns are instantiated which is why there are separate methods for registering them. A controller is not a singleton like other services and may be instantiated multiple times when a route is invoked or the controller is otherwise activated by AngularJS. It is important to keep in mind these differences in the lifecycle of these special objects and other custom objects or values created in user code. 

### Where are we?
In this article we covered the module and provider patterns used by AngularJS to manage application code and dependency creation. With a good understanding of the patterns and underlying techniques used to manage an application, it is time to move on to understanding the core coding concepts that make up the bulk of each AngularJS application.

Author of this tutorial is [Matt Milner](https://www.pluralsight.com/authors/matt-milner).
