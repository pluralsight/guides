Data analytics is an ever-growing trend in the software industry. An increasing number of industries rely on data in order to grow and gain competitive advantage. Almost all software products, from sleep cycle mobile apps to enterprise logistics software come coupled with in-depth analytics.  

However, making sense out of data tends to be a tedious, long and volatile process. Building a meaningful dashboard requires a lot of changes both to your back-end architecture and to your front-end visualizations. This is where [Keen.IO](https://keen.io/) can be very helpful. [Keen.IO](https://keen.io/) is a SaaS solution that makes analyzing data and building dashboards easy, fast and efficient. It comes with a developer-friendly [API](https://keen.io/docs/) that covers the whole process from collecting data, analyzing it and visualizing it. It also comes with [SDKs](https://keen.io/docs/sdks/) for a great variety of technologies.

In this guide, we are going to build a reactive dashboard from scratch by building a Ruby on Rails 5 web applciation using the newly-introduced [ActionCable](https://github.com/rails/rails/tree/master/actioncable) and [Keen.IO's Ruby SDK](https://github.com/keenlabs/keen-gem). Let's get started!


# Setting up

### Connecting your Rails app with Keen.IO

 First, open your terminal and make a new Rails 5 application. You need to have [Ruby 2.2.4](https://www.ruby-lang.org/en/) and up in order to do that : 
 
```bash
rails _5.0.0_ new reactivedashboard
```
Once the application is generated, move to its directory:
```bash
cd reactivedashboard
```
Add the following gems to your <code>Gemfile</code>:
```ruby
#Gemfile.rb

gem "keen"
gem "dotenv-rails"
gem 'bootstrap-sass', '~> 3.3.6'
gem 'sass-rails', '>= 3.2'
```
And install them:
```bash
bundle install
```

The [keen](https://github.com/keenlabs/keen-gem) gem will make Keen.IO's Ruby SDK available in the Rails app, [dotenv-rails](https://github.com/bkeepers/dotenv) will let you save your Keen.IO APi credentials securely and [bootstrap-sass](https://github.com/twbs/bootstrap-sass) will be used to make everything look nice and crisp.
  
 In order to set up Boostrap, go to <code>app/assets/stylesheets/application.scss</code>:
and insert the following lines:

```scss
// "bootstrap-sprockets" must be imported before "bootstrap" and "bootstrap/variables"
@import "bootstrap-sprockets";
@import "bootstrap";
``` 
 
 Now, onto the more important things: 
 
### Getting your credentials
Connecting to Keen.IO is done with the standard approach with using environment variables. [Environment variables](http://railsapps.github.io/rails-environment-variables.html) are used to configure an application's behaviour in different environments (development, staging, production, etc.). The environment variables must **always** kept secret, since some of them may give out sensitive information about your application. One way to do this is using the [dotenv-rails]((https://github.com/bkeepers/dotenv) that you just installed in your Rails applicaiton.

  Go to [Keen.IO](https://keen.io/)'s website and register your free account. Once you register, log in and create your first project. Once you've created it, in its  overview tab, you will have the credentials of your project:
 
- Project ID: The unique identifier of your project
- API credentials:
  - Write key - used when you want to publish data to your collections
  - Read key - used for running queries and reading data from your collections
  - Master key - used for deleting records and collections 
 

 In the root of your Rails application, create a <code>.env </code> file. 
``` 
KEEN_PROJECT_ID=YOURKEENPROJECTID
KEEN_MASTER_KEY=YOURKEENMASTERKEY
KEEN_WRITE_KEY=YOURKEENWRITEKEY 
KEEN_READ_KEY=YOURKEENREADKEY
```
Then, open up your first Keen.IO project's dashboard and paste your credentials in the corresponding fields.
 
  With the keen gem installed and your API credentials inserted in your <code>.env</code> file, you are ready to publish, delete and update information through your Rails app.
  
# Publishing your data

Keen.IO uses [collections](https://keen.io/guides/data-modeling-guide/) that represent different types of events that happen in your application. An collection can contain all sorts of data you would like to record - clicks of an user, purchases made through your application, searches and so on. 


In this guide, Keen.IO will keep track of the information of a particular model collection.We we will call it <code>products</code>. A product will have a name, a description, number of times it is favorited and a price. 

Let's scaffold the product model:

```
rails g scaffold product price:decimal name:string description:text favorites:integer
```
This will generate all the views, controller actions, model and database migrations you need to make [CRUD](https://en.wikipedia.org/wiki/Create,_read,_update_and_delete) operations on the products.
Add the model to your database:
```
rails db:migrate
```

Let's start populating data into Keen.IO . Add tollowing snippet to your model:
```ruby
#app/models/product.rb

class Product < ApplicationRecord
  after_save { Keen.publish 'products' , self }
end
```
<code>after_save</code> is a [callback hook](http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html) which is executed every time a product  in our application's database. The <code>Keen.publish</code>  method provided by the Keen.IO's Ruby SDK accepts two parameters. The first parameter is the collection that is going to be interacted with and the second parameter is the data. In this case, <code>self</code> refers to the particular instance of the model itself, which means that we are going to send all the parameters of the Product model (name, price, favorites, etc) every time a particular object is created.

> **Recording events vs recording entities**
> When you are doing analytics, you must always think about data in terms of event. In this tutorial, we are not recording the products themselves, we are only the event of their creation. The product entity should stay in the database of the application. What's important is the data each event generates. Read more on the topic [here](https://keen.io/blog/53958349217/analytics-for-hackers-how-to-think-about-event-data).



Start your Rails server
```bash
 rails s
```

Go to [http://localhost:3000/products/new](http://localhost:3000/products/new)  and create some products.
After you have added a few products, go to your Keen.IO's project dashboard. Under the overview tab you will be able to see your newly created collection and all the times in which you just  created in the Rails application.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6392586c-2c7f-4325-8ca6-6cfcf27a717c.001)


You are now ready to start analyzing the information you just entered.
# Analyzing your data
The next step is to start  making sense out of the whole data. At this point, a team would nornally try to devise the numerous ways to represent the data in a meaningful format. Thankfully, [Keen.IO's data analysis API does all this for you](https://keen.io/docs/data-analysis/?s=gh-gem) . It gives you options to select the core analysis types:
 - **Sum** - calculate the sum of numeric values in a collection
 - **Average** - calculate the average of numeric values in a collection
 - **Minimum** - return the mininu, of all numeric values in a property
 - **Maximum** - return the maximum of all numeric values in a property
 - **Percentile** - calculate a percent of a given property in a colleciton
 - **Median** - get the median value of all numeric values of a given property
 - **Count and count_unique** - count either all or the unique occurences of values of a given property
 - **Select and select_unique** - select a list of values found for a given property
 
 
You can also input query parameters to group your data orfilter it data in terms of:
- **Timeframe** - Calculate values for a given timerframe
- **Interval** - Calculate values from the currenty minute up to the current year.
- **Filter** - Refine your serach by filtering out values for given properties

You can also combine multiple queries and collections to make more complex analytics using [funnels](https://keen.io/docs/api/#funnels).


## Query explorer
 You can start playing with the events collected in Keen.IO using its explorer. To access it, go to your project overview and click on the explorer tab on the top right. [You can also download the explorer and use it locally ](https://github.com/keen/explorer).
 

 Using the explorer's UI , you can leverage all the analysis types and apply  the filters easily. For this tutorial, we are going to create three different queries:
 
 ### Creating and saving queries
 
In your explorer, select **count** as analysis time, collection type as **products** and and a time frame as **this month**.
 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a6da99de-4a13-4f19-b36b-c9c383c013ea.002)


 Voila! You just created your first query. You can save your query and use it any time you want. You can also visualize the data differently if applicable by chosing from the different types of visualizations in the top right , next to the field for the query name.
 

 The next step is to click on the **embed** button on the right and get the code for displaying the query for the next step in the guide:
 

```javascript
var client = new Keen({
  projectId: "YOURKEENPROJECTID",
  readKey: "YOURKEENREADKEY"

});

Keen.ready(function(){
  
  var query = new Keen.Query("count", {
    eventCollection: "product",
    timeframe: "this_14_days",
    timezone: "UTC"
  });
  
  client.draw(query, document.getElementById("my_chart"), {
    // Custom configuration here
  });
  
});
```
You can continue playing with the explorer and create one or two more queries. Make sure you save them so because you are going to need the code for embedding in the next step of the guide.

# Visualising your data reactively

After collecting data and analyzing it, it is time to build the dashboard.
First, include the [Keen.IO JavaScript SDK ](https://github.com/keen/keen-js) in the Ruby on Rails application. There are many ways to do this, but for the sake of simplicity, you can do this by simply adding it to your applicaiton layout:

```html
<!-- app/views/layouts/applicaiton.html.erb -->
<html>
  <head>
    <!-- Paste this line in the head section of the layout -->
    <%= javascript_include_tag "//cdn.jsdelivr.net/keen.js/3.4.1/keen.min.js" ,'data-turbolinks-track': 'reload'%>
  </head>
  <!-- content -->
</html>
```

Next, let's create an action and a view for the dashboard:

```ruby
#app/controllers/products_controller.rb
class ProductsController < ApplicationController
  #...
  def dashboard
  end
  #...
end
```

Add the following line to <code>routes.rb </code> in order to make the dashboard the root action of the application:
```
#app/routes.rb
Rails.application.routes.draw do
  root to: 'products#dashboard' #set the dashboard as your root
  resources :products
end
```
Lastly, create a view for your <code> dashboard </code> action:

```html
<!-- app/views/products/dashboard.html.erb -->
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">
        Keen Dashboard
      </a>
    </div>
    <ul class="nav navbar-nav navbar-right">
      <li><%= link_to 'All products', products_path%></li>
    </ul>
  </div>

</nav>
<div class="container-fluid">
  <div id="alert-placeholder"></div>
  <div class="row">
    <div class="col-md-6" id="chart-wrapper">

    </div>

    <div class="col-md-6" id="pie-wrapper">

    </div>
  </div>
  <div class="row">
    <div class="col-md-6" >
      <div class="col-md-6" id="total-wrapper">

      </div>
    </div>
  </div>

</div>
```
The view contains a simple bootstrap header and a few columns with ids. The <code>chart-wrapper</code> , <code>pie-wrapper</code> and <code>total-wrapper</code> divs are going to be used as a reference for putting the analytics elements into the document.

### Implementing reactivity
 Reactive analytics means that the visualization of the data is going to change automatically as you change the data in Keen.IO without the need to restart the page. Ruby on Rails 5 make this easy by introducing  [ActionCable](https://github.com/rails/rails/tree/master/actioncable). ActionCable is a Rails 5 module that introduces an API for working with [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) in Ruby on Rails.
 
 The first step is to generate a channel. In your terminal, execute:
 ```bash
  rails g channel analytics
 ```
The command will create a server side representation of the channel in <code>app/channels</code>  directory and a coffeescript file in <code>app/assets/channels</code> for handling the client side of the channel.

First, you need to explicitly define the name of the channel through which the analytics information is going to be streamed.
 ```
 #app/channels/analytics_channel.rb
class AnalyticsChannel < ApplicationCable::Channel
  def subscribed
    stream_from "analytics_channel"
  end
  #...
end
 ```
<code>steam_from</code> is used to further specify the channel. It is used because a channel has multiple instances. For example, one channel can broadcast to multiple users. In that case, the parameters entered in <code>stream_from</code> would include an unique identifier. In this case, we are doing the simple way, by simply hardcoding a string.

Second, you need to make a [job](http://edgeguides.rubyonrails.org/active_job_basics.html) which is going to broadcast to the analytics channel every time the application publishes an event. The idea of putting a broadcast in a job is because jobs can run in parallel with other processes which happening in the application. This ensures that your application will be able to queue multiple requests for broadcasts that occur simultaneously.

Open your terminal and generate the job.
```bash
 rails g job UpdateAnalytics
```
In order to maintain reactivity, the job has to be queued every time an event is published to Keen.IO. In this guide, we are doing it only once - when creating a new product:
```ruby
#app/models/product.rb
class Product < ApplicationRecord
  after_create_commit do
    #publish the product creation event to Keen.IO
    Keen.publish 'products' , self
    #broadcast the job to update the analytics
    UpdateAnalyticsJob.perform_later self 
   end

    #...
end
```
In the job for the job itself, add a call to broadcast to the <code>analytics_channel</code>:
```ruby
#app/jobs/update_analytics_job.rb
class UpdateAnalyticsJob < ApplicationJob
  queue_as :default

  def perform(product)
    ActionCable.server.broadcast 'analytics_channel', product: product
  end
end
```

That's it! The back-end is set up. Let's move to the front-end and put the Keen.IO JavaScript SDK library that was previously added in the appication to use.
### Visualizing queries

 Go to <code>app/assets/javascripts/channels/analytics.coffee</code>
 
 There, you can see two functions - <code>connected() </code> , <code> disconnected() </code> and <code> received() </code> . <code> connected() </code> is called when the client connects to the server-side websocket, <code> disconnected() </code> is called in the opposite case, and <code> received()</code> is called when data is broadast from the server.

 Outside of these functions, declare a **global** function that is going to be called to reload all the queries and visualize them on the dashboard:
 
 App.analytics = App.cable.subscriptions.create "AnalyticsChannel",

```javascript
  connected: ->
      loadAnalytics();

  disconnected: ->
    # Called when the subscription has been terminated by the server

  received: (data) ->
      loadAnalytics();
 

@loadAnalytics = () =>
  #paste the embed code from Keen.IO here

 ```
 
 <code> @loadAnalytics </code> (**@** denotes a global function in CoffeeScript) is going to be called in three places - when the client connects to the server-side channel, when there is data received from the channel and in the view itself. The only thing that is missing is the call from the view itself, so let's add it:
 
```html
<!-- app/views/products/dashboard.html.erb -->

<!-- put this in the bottom of the document -->
    <script>
        loadAnalytics()
    </script>
```

Now, go to your Keen.IO explorer and start getting the embed code from your favorite queries. Use [js2.coffee](http://js2.coffee) to transpile the javascript to coffeesscript.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f631fa39-6494-470c-af0c-e30241b021e2.003)

Here is an example with three queries displayed in the different div wrappers:
```js
@loadAnalytics = () =>
  client = new Keen(
    projectId: 'YOURKEENPROJECTID'
    readKey: 'YOURKEENREADKEY')
  Keen.ready ->

    allProducts = new Keen.Query("count", {
      eventCollection: "product",
      timeframe: "this_1_months",
      timezone: "UTC"
    });


    favoritesDist = new Keen.Query("count_unique", {
      eventCollection: "product",
      groupBy: [
        "name"
      ],
      targetProperty: "favorites",
      timeframe: "this_14_days",
      timezone: "UTC"
    });

    productsCreatedToday =  new Keen.Query("sum", {
      eventCollection: "product",
      interval: "daily",
      targetProperty: "id",
      timeframe: "this_14_days",
      timezone: "UTC"
    });

    client.draw productsCreatedToday , document.getElementById('chart-wrapper'), {}
    client.draw favoritesDist, document.getElementById('pie-wrapper'), {}
    client.draw allProducts, document.getElementById('total-wrapper'), {}
```

