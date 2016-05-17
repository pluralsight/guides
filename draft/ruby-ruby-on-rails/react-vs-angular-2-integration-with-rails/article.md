In this tutorial, the React and Angular 2 way of setting up an application and streaming data back and forth will be reviewed and the differences will be analyzed.

### Setting up a sample Rails application
 Because the focus on the tutorial is on getting React or Angular integrated into a Rails application, the Rails application itself will be as simple as possible - it will contain one action which will return arbitrary JSON that is going to be rendered on the page using React or Angular. Open up your terminal or command prompt and type:
 
```bash
rails new sampleapp
```
 This will scaffold a default Rails application and install all the required dependencies to make it run. Go to the folder of the application and locat the Gemfile. Fo this application, you will need only the following gems:
 
```ruby
#Gemfile.rb
gem 'sqlite3'
gem 'sprockets'
gem 'rack-cors'
```

<code> SQLite3 </code> is the gem for the Ruby-based database, <code> Sprockets </code> is for the asset pipeline and <code> rack-cors </code> is used to enable cross-origin HTTP requests (CORS) so that the React/Angular client-side can communicate with the server. In your terminal/command promt, run:

```bash
bundle install
```
This will install the gems in the Gemfile. To configure <code> rack-cors </code>, add this code snippet to the configuration file of the Rails applicaiton:

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
This configuration will permit all types of requests from any source (<code> * </code> denotes any address) to be accepted. For development, this setup is acceptable. However, for production, you need to replace the asterisk with the URL of your client-side application.

The next step is to make a controller and an action that will return JSON data and make a route for them. For the sake of simplicity and for the tutorial, let's put the action in the application controller:

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
 An action called  <code> index </code> is going to respond with the  <code> { some: 'data' } </code> JSON object when called. Let's put a route for the action:
```ruby
#config/routes.rb
Rails.application.routes.draw do
  get '/api' => 'application#index', defaults: { format: :json }
end
```

### Integrating Angular 2 

Angular 2 has two specificts  - it is a framework and it uses TypesScript. These specifics come with certain requirements when it comes to integration:

Because Angular 2 is a framework and not a library, it would be best if is  put in a separate directory where all its files are going to reside. This means that, instead of putting it into the Rails asset pipeline (app/assets), it will reside in the Rails application's <code>public</code> folder, separated from the compilation and the logic of the Rails application. This will allow a clearer separation of concerncs between the Rails and the Angular 2 frameworks and their dependencies.

Angular 2 also TypeScript, which is a superset of JavaScript, Angular 2 will also need a TypeScript transpiler configured in the directory of the Rails application. Transpilers (short for [trascompilers](http://www.computerhope.com/jargon/t/transcompiler.htm) in JavaScript are tools that read to read the TypeScript code (or CoffeScript or similar)  and transpile it to pure JavaScript that can be interpreted by the browser. 

There are three files that need to be created in order to fulfill the requirements:
 - **package.json**
 - **typings.json**
 - **tsconfig.json**
 

To install these requiurements, you need to have the [Node Package Manager (npm)](https://www.npmjs.com/) installed on your computer.
 
##### package.json
 The <b>package.json</b> file that contains a list of all the packages required to integrate Angular 2 with Rails:
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
Paste the contents in a file in the root directory of the Rails application and name it <b> package.json </b>.
The file contains [typings](https://www.npmjs.com/package/typings), a package that is used for configuring the behaviour of TypeScript and the [typescript](https://www.npmjs.com/package/typescript)  package itself as *devDependencies* (dependencies of the dependencies). Obviously, [angular2]([https://www.npmjs.com/package/angular2) is also one of the packages as well as libraries such as [systemjs](https://github.com/systemjs/systemjs) and [es6-shim](https://github.com/paulmillr/es6-shim) that add EcmaScript 6 functionality, which is requried for Node 5.


##### typings.json
Also in your root directory, create a file named **typings.json** . This file will be used to configure the dependencies of TypeScript after the packages are installed. You can also do it manually by running <code> typings install </code> in your console.
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
The file contains standard configuration for the behavior of TypeScript in the Angular 2 application. One thing that requires a paricular attention is the <code>rootDir</code> property which defines that the Angular 2 application will reside in the <code> public </code> folder.

After the three files are added, write the following command in your console:
```bash
 npm install
```

And wait as the packages are installed. Once the command completes, there will be an extra folder named <code> node_modules </code> in the root directory that will contain the installations of all the packages.

The configuration is almost finished, but there is a **gotcha** - the <code> node_modules </code> directory will not load in the application's assets since it is not in the <code>app/assets</code> directory. Thus, it must be added explicitly:
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


React | Angular 2 | Rails   
------------------- | -------------------- | ------------
Root/Parent component | Service | View layer


React | Angular 2 | Rails   
------------------- | -------------------- | ------------
app/assets/javascripts directory | public or a separate directory| app/views diectory

