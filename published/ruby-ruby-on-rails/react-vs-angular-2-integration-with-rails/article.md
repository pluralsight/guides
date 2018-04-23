![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8ee194bc-0c61-4176-ae80-e4a2d2ae39d6.png)




 One of the most recent debates in the web development world has been over the choice of constructing the view layer of modern applications. In this guide, we will compare the [React](https://facebook.github.io/react/) and the [Angular 2](https://angular.io/) integrations using Ruby on Rails. The two approaches will be analyzed in terms of ease of integration and use with Rails.

### Setting up a sample Rails application
 Because the focus on the tutorial is on getting React or Angular integrated into a Rails application, the Rails application itself will be as simple as possible - it will contain one action that will return an arbitrary JavaScript Object Notation (JSON) that is going to be rendered on the page using React or Angular. Open up your terminal or command prompt and type:
 
```bash
rails new sampleapp
```
 This will scaffold a default Rails application and install all the required dependencies to make it run. Go to the directory of the application and locate the Gemfile. For this application, you will need only the following gems:
 
```ruby
#Gemfile.rb
gem 'rails'
gem 'sqlite3'
gem 'sprockets'
gem 'rack-cors'
```

[SQLite3](https://rubygems.org/gems/sqlite3/versions/1.3.11)  is the gem for the Ruby-based database, [Sprockets](https://github.com/rails/sprockets-rails) is for the asset pipeline, and [rack-cors](https://github.com/cyu/rack-cors) is used to enable cross-origin HTTP requests (CORS) so that the React/Angular client-side can communicate with the server. 

In your console, run:

```bash
bundle install
```
This will install the gems in the Gemfile. To configure <code>rack-cors</code>, add this code snippet to the configuration file of the Rails application:

```ruby
 #config/application.rb
module Starterapp
  class Application < Rails::Application
  #...
    config.middleware.insert_before 0, 'Rack::Cors' do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end

```
This configuration will permit all types of requests from any source (<code>*</code> denotes any address) to be accepted. For development, this setup is acceptable. However, for production, you need to replace the asterisk with the URL of your client-side application.

The next step is to make a controller and an action that will return JSON data and make a route for these data. For the sake of simplicity and for the tutorial, let's put the action in the application controller:

```ruby
 #app/controllers/application_controller.rb
class ApplicationController < ActionController::Base
  #...
  def index
    respond_to do |format|
      format.json { render json: {some: 'data'} }
    end
  end
  #...
end
```
 An action called  <code>index</code> is going to respond with the  <code>{some: 'data'}</code> JSON when requested through [http://localhost:3000](http://localhost:3000). Let's put a route for the action:
```ruby
#config/routes.rb
Rails.application.routes.draw do
  get '/api' => 'application#index', defaults: { format: :json }
end
```

## Integrating Angular 2 

 Angular 2 has two specifics  - it is a framework, meaning that it only provides general functionality and can be modified to fit users' needs, and it uses [TypeScript](https://www.typescriptlang.org/). These specifics come with certain requirements when it comes to integration with the  Rails application:

Because Angular 2 is a framework and not a library, it would be best if is  put in a separate directory where all its files are going to reside. This means that, instead of putting it into the Rails asset pipeline (<code>app/assets</code>), the Angular 2 app will reside in the Rails application's <code>public</code> directory, separated from the compilation and the logic of the Rails application. This will allow us to distinguish the concerns of using the Rails and the Angular 2 applications and their dependencies.

Angular 2 uses TypeScript, a superset of JavaScript. As of today, there isn't a way for TypeScript to be implemented into Rails' asset pipeline, which means that a transpiler has to be configured in the root directory of the Rails application. Transpilers (short for [transcompilers](http://www.computerhope.com/jargon/t/transcompiler.htm)) in JavaScript are tools that read   code (or CoffeScript or similar)  and transpile it to pure JavaScript that can be interpreted by the browser. 

#### Setting up the environment for Angular 2
Because of TypeScript's requirements, there are three files that need to be created in the root directory in order for the environment to be set up for an Angular 2 application. 

 - *package.json*
 - *typings.json*
 - *tsconfig.json*
 
**To install these requirements, you need to have the [Node Package Manager (npm)](https://www.npmjs.com/) installed on your computer.**  Let's go through each of the files:

 
##### package.json
 The *package.json* file that contains a list of all the packages required to integrate Angular 2 with Rails:
```json
{
  "name": "rails-ng2",
  "version": "0.0.1",
  "scripts": {
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "typings": "typings",
    "postinstall": "typings install"
  },
  "license": "ISC",
  "dependencies": {
    "angular2": "2.0.0-beta.15",
    "systemjs": "0.19.26",
    "es6-shim": "^0.35.0",
    "reflect-metadata": "0.1.2",
    "rxjs": "5.0.0-beta.2",
    "zone.js": "0.6.10",
    "intl": "^1.0.1"
  },
  "engines": {
    "node": "5.3.0"
  },
  "devDependencies": {
    "typescript": "^1.8.10",
    "typings":"^0.7.12"
  }
}
 
```
Paste the contents in a file in the root directory of the Rails application and name it *package.json*.
The file contains [typings](https://www.npmjs.com/package/typings), a package that is used for configuring the behaviour of TypeScript and the [typescript](https://www.npmjs.com/package/typescript)  package itself as *devDependencies* (dependencies of the dependencies). Obviously, [angular2]([https://www.npmjs.com/package/angular2) is also one of the packages as well as libraries such as [systemjs](https://github.com/systemjs/systemjs) and [es6-shim](https://github.com/paulmillr/es6-shim) that add EcmaScript 6 functionality, which is requried for Node 5.


##### typings.json
Also in your root directory, create a file named *typings.json*. This file will be used to configure the dependencies of TypeScript after the packages are installed. You can also configure TypeScript dependencies manually by running <code> typings install </code> in your console.
```json
{
  "ambientDependencies": {
    "es6-shim": "github:DefinitelyTyped/DefinitelyTyped/es6-shim/es6-shim.d.ts#7de6c3dd94feaeb21f20054b9f30d5dabc5efabd",
    "jasmine": "github:DefinitelyTyped/DefinitelyTyped/jasmine/jasmine.d.ts#5c182b9af717f73146399c2485f70f1e2ac0ff2b"
  }
}
```
The last file you need to create in the root directory is **tsconfig.json**:
```json
{
  "compilerOptions": {
    "target": "es5",
    "module": "system",
    "moduleResolution": "node",
    "sourceMap": true,
    "emitDecoratorMetadata": true,
    "experimentalDecorators": true,
    "removeComments": false,
    "noImplicitAny": false,
    "rootDir":"public"
  },
  "exclude": [
    "node_modules",
    "typings/main",
    "typings/main.d.ts"
  ]
}
```
The file contains standard configuration for the behavior of TypeScript in the Angular 2 application. One thing that requires paricular attention is the <code>rootDir</code> property which defines that the Angular 2 application will reside in the <code>public</code> directory.

After these three files are added to the root directory of the Rails application, write the following command in your console:
```bash
 npm install
```

Wait as the packages are installed. Once the command completes, there will be an extra directory named <code> node_modules </code> in the root directory that will contain the installations of all the packages.

In order to run the configuration of the typescript, run:
```bash
 npm run tsc
```


The configuration is almost finished, but there is a **gotcha** - the <code>node_modules</code> directory will not load in the application's assets since it is not in the <code>app/assets</code> directory. Thus, the directory must be added explicitly in the configuration of the Rails application:
```ruby
#config/application.rb
module Starterapp
  class Application < Rails::Application
      #.. CORS configuration
      
     # Explicitly add the 'node_modules' directory
     config.assets.paths << Rails.root.join('node_modules')
  end
end
```

#### Bootstrapping Angular 2
The Rails application is now ready to load an Angular 2 application that resides in its <code>public</code> directory. To start off, let's create a root html document that is going to load all the JavaScript files:
```html
<!-- public/index.html -->
<html>
<head>
    <!-- 1. Load libraries -->
    <script src="/assets/intl/dist/Intl.js"></script>
    <script src="/assets/intl/locale-data/jsonp/en.js"></script>

    <script src="/assets/es6-shim/es6-shim.min.js"></script>
    <script src="/assets/systemjs/dist/system-polyfills.js"></script>

    <script src="/assets/angular2/bundles/angular2-polyfills.js"></script>
    <script src="/assets/systemjs/dist/system.src.js"></script>
    <script src="/assets/rxjs/bundles/Rx.js"></script>
    <script src="/assets/angular2/bundles/angular2.dev.js"></script>
    <script src="/assets/angular2/bundles/http.dev.js"></script>
    <script src="/assets/angular2/bundles/router.dev.js"></script>
</head>
<body>
<script>
    System.config({
        map: {
            'app': '/app',
        },
        packages: {
            'app': {
                format: 'register',
                defaultExtension: 'js'
            }
        }
    });
    // and to finish, let's boot the app!
    System.import('app/boot');
</script>

<base href="/">
<app-router></app-router>
</body>
</html>
```

Between the *<script></script>* tags, the [systemJS](https://github.com/systemjs/systemjs) library will configure the modules and import the <code>app/boot</code> file, which is going to be included later in the guide. Another interesting snippet in the file is the<code> app-router</code> tag, where the built-in Angular 2 router component is going to be mounted.

##### The first Angular 2 component

In the <code>public</code> directory, add an <code>app</code> directory. This is where all the Angular 2 files will be placed. Let's start with the first component - <code>home.component</code>
```javascript
// public/app/home.component.ts
import {Component, OnInit} from 'angular2/core'
import {RouteParams}  from 'angular2/router'
import {Http, HTTP_PROVIDERS} from 'angular2/http';
@Component({
    selector: 'home',
    templateUrl: '/app/home.component.html'
})
export class HomeComponent {
    message: string;
    constructor( public http: Http){
        this.http.get('http://localhost:3000/api')
            .subscribe(
                data => this.message = data.json().some,
                err => console.log(err)
            );
    }
}
```
 One of the most notable features of TypeScript is that it provides classes and ability to <code>import</code> and <code>export</code> snippets of code. Such features make the code more maintainable and more familiar to programmers who are not used to JavaScript's idiosyncracies. 
 
 Here, you can see how the <code>Http</code> provider is imported into the file and then made available in the constructor of the <code>HomeComponent</code> class. Doing http requests is done similarly to Angular 1's <code>$http.get()</code> service, but the syntax of the promise is handled a little bit differently - <code>subscribe</code> acts similarly to <code>$promise.then()</code>, and the functions for resolving the promise use [EcmaScript 6's arrow syntax](http://exploringjs.com/es6/ch_arrow-functions.html). 
 
 To make it more familiar, the Angular 2 <code>data => this.message = data.json().some</code> translates to <code>function(data) { $scope.message = data.some }</code> in Angular 1. Another interesting feature is that  variables can have a strong type, as it is demonstrated with the <code> message: string; </code> component variable.
 
 Now let's add the template of the component:
```html
<!-- public/app/home.component.html-->
<h2>Rails Angular 2</h2>
{{message}}
```
The template will simply print the message that was fetched from the Rails application.

##### Routing the component

 The first component is ready, but there must be a way through which it can be reached withing the Angular 2 application. Because Angular 2 is a framework, it comes with its own built-in router that can be imported and configured to your liking:
```javascript
//public/app/app_router.component.ts
import {Component} from 'angular2/core'
import {RouteConfig, ROUTER_DIRECTIVES} from 'angular2/router'


@Component({
        selector: 'app-router',
        template: '<router-outlet></router-outlet>',
        directives: [ROUTER_DIRECTIVES],
        styles:[]
    })
@RouteConfig([

])
export class AppRouterComponent {}
```
In this file, the router component is imported and configured. You can see that the component is bound to the  <code>app-router</code> tag. The built-in router directives are put under the <code>directives</code> property.  <code>@RouteConfig</code> contains an array of route objects. 

A route object can contain a *path* , *name* and a *component* that it uses. Let's add <code>home.component</code> in there:

1. Import the component

    ```javascript
    import {HomeComponent} from './home.component'
    ```
2. Put it in <code>@RouteConfig</code>

    ```javascript
     @RouteConfig([
        { path: '/', name: 'Home', component: HomeComponent }
    ])
    ```

The last thing that needs to be done is to add the <code>boot</code> file that is going to bootstrap, or initiate, the application:
```javascript
//public/app/boot.ts
import {provide} from 'angular2/core'
import {bootstrap} from 'angular2/platform/browser'
import {AppRouterComponent} from './app_router.component'
import {HTTP_PROVIDERS} from 'angular2/http'
import {ROUTER_PROVIDERS} from 'angular2/router'

bootstrap(
    AppRouterComponent,
    [
        HTTP_PROVIDERS,
        ROUTER_PROVIDERS
    ]
);
```
You can regard <code>boot.ts</code> as the initialization file of the Angular 2 application. It imports the <code>AppRouterComponent</code> that was defined earlier and bootstraps it, making all its routes and components reachable through their paths.

This step completes the Angular 2 integration into your Rails application. Start the server using:
```bash
rails s
```
Open your browser and go to [http://localhost:3000](http://localhost:3000). You should see the <cvode>home.component</code> with the message displayed on the server. if you encounter any errors, you can check the [GitHub repository](https://github.com/hggeorgiev/rails-angular2starter) of what we just implemented.

## Integrating React
 Unlike Angular 2, which is a full-fledged framework, React is simply a library that provides bare-bones features for constructing user interfaces. React uses [JSX](https://jsx.github.io/), or Java Serialization to XML, to render its components. JSX is much simpler and lighter than TypeScript, and, unlike TypeScript, JSX can be integrated into Rails' asset pipeline. As a result, React integration becomes sufficiently faster. 
 
 
#### Setting up the environment for React
 Because React's JSX easily integrates with the asset pipeline, you can set up the environment by simply installing the [react-rails](https://github.com/reactjs/react-rails) gem. It comes with a set of useful features - server-side rendering, component generators and more. Add the gem to your Gemfile:

```ruby
# Gemfile.rb
gem 'react-rails'
```
Install the gem:
```bash
bundle install
```
Run the generator provided by the gem to make the Rails application work with React:
```bash
rails g react:install
```
This will create an initialization file named <code>component.js</code> in the <code>app/assets</code> directory and a new directory, <code>app/assets/javascripts/components/</code> for the rest of the components. It is also going to add the React dependencies to the asset pipeline:
```javascript
//app/assets/javascripts/application.js

//= require react
//= require react_ujs
//= require components
```

React is configured, now all we need is to put a controller, a view, and a route for our Ruby on Rails application to render the components. 

First, let's create a simple controller. Go to <code>app/controllers</code> and create a file named <code>site_controller.rb</code>:
```ruby
#app/controllers/site_controller.rb

class SiteController < ApplicationController
  def index
  end
end
```
Put the controller's <code>index</code> action as a root for your Rails application:

```ruby
#app/config/routes.rb

Rails.application.routes.draw do
   get '/api' => 'application#index', defaults: { format: :json }
  root to: 'site#index' #add the route for the React component
end
```


This is everything that is required to setup the environment React for the Rails application. Next, we are going to render the data from the server.
 
#### The first React component
  
 To make a new component, you can simply run the generator and it will be created for you:
```javascript 
 rails generate react:component Item  --es6
```
The generator will create an component in <code>app/assets/javascripts/components</code> in EcmaScript 6 syntax. Open the file and put the following code snippet:


```javascript 
//app/assets/javascripts/components/item.es6.jsx
class Item extends React.Component {
    constructor(props, context) {
        super(props, context);
        this.state = {item: ''};
    };

    render () {
    return (
      <div>
        <h2> Rails React starter </h2>
        <div> {this.state.item}</div>
      </div>
    );
  }
}




```
<code> constructor() </code> is used to initialize the component's state variables. In the constructor, an empty <code>Item</code> object will be initialized in the component's state. <code>super</code> is a common Object-Oriented pattern, here it's used to inherit the props and context from the <code>React.Component</code> class.
The <code>render()</code> function is used to render html into the component. In this case, it will render the <code>item</code> state of the component. <code>this.state</code> contains the state of the component, which is usually contained in private or component-specific variables. <code>item</code> is one of these variables. When there are several nested components however, the data between them is passed through <code> this.props </code>.

What we want is to render the data from the server into <code>item</code>:

```javascript 
//app/assets/javascripts/components/item.es6.jsx
class Item extends React.Component {
   //constructor
    componentDidMount() {
        $.getJSON('/api', (response) => { this.setState({ item: response.some }) });
    }
  //render
 }

```
<code> componentDidMount() </code> is a method that will be called when the component becomes mounted into the DOM. Simply said, it is called when the component is rendered on the page. In it, the <code>$.getJSON</code> function makes a request to <code>localhost:3000/api</code> and uses EcmaScript 6's arrow function to get the callback when the request succeeds. [this.setState()](https://facebook.github.io/react/docs/component-api.html#setstate) will set the <code>item</code> property with the property of the <code>response</code> object, which will contain <code>{ some: 'data' }</code>.

Last, make a view to render the component. Go to <code>app/views</code>, create a directory named <code>site</code> and create a file named <code>index.html.erb</code>. Put the following snippet in it:

```html
<!-- app/views/site/index.html.erb -->
<%= react_component 'Item' %>
```
<code> react_component </code> is a built-in helper method provided by react-rails that is going to render the <code>Item</code> component into the Rails view.

 With this step, the Rails application integrates React. Go to http://localhost:3000 and see the results. If you encounter any problems, check the [GitHub repository](https://github.com/hggeorgiev/rails-reactstarter) .

## Summary

Here is a overview of the three different ways the view layer in Rails can be hanlded:

#### Handling server-side data

React | Angular 2 | Rails   
------------------- | -------------------- | ------------
Root/Parent component | Service | View layer

#### Location of the view layer

React | Angular 2 | Rails   
------------------- | -------------------- | ------------
app/assets/javascripts directory | public or a separate directory| app/views directory
#### Language


React | Angular 2 | Rails   
------------------- | -------------------- | ------------
JSX| TypeScript| ERB


________________________

Both React and Angular 2 come with their pros and cons. However, in terms of integration and configuration with Rails, React is a clear winner. The integration with the asset pipeline and server-side rendering make React far simpler to set up and more configurable with Ruby on Rails. Angular 2, on the other hand, tends to be bulkier and trickier to implement. Part of the issue is TypeScript's complexity when it comes to integrating with the asset pipeline. Furthermore, Angular 2 is quite new to the Rails ecosystem, so implementations have not yet been adequately generated. 

