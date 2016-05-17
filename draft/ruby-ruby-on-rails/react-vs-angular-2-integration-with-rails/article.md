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
 
 Because Angular 2 uses TypeScript, which is a superset of JavaScript, Angular 2 will also need a TypeScript transpiler configured in the directory of the Rails application. Transpilers (short for [trascompilers](http://www.computerhope.com/jargon/t/transcompiler.htm) in JavaScript are tools that read to read the TypeScript code (or CoffeScript or similar)  and transpile it to pure JavaScript that can be interpreted by the browser. 
 
 

React | Angular 2 | Rails   
------------------- | -------------------- | ------------
Root/Parent component | Service | View layer


React | Angular 2 | Rails   
------------------- | -------------------- | ------------
app/assets/javascripts directory | public or a separate directory| app/views diectory

