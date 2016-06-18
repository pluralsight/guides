Data analytics is an ever-growing trend in the software industry. An increasing number of industries rely on data in order to grow and gain competitive advantage. Almost all software products, from sleep cycle mobile apps to enterprise logistics software come coupled with in-depth analytics.  

However, making sense out of data tends to be a tedious, long and volatile process. Building a meaningful dashboard requires a lot of changes both to your back-end architecture and to your front-end visualizations. This is where [Keen.IO](https://keen.io/) can be very helpful. [Keen.IO](https://keen.io/) is a SaaS solution that makes analyzing data and building dashboards easy, fast and efficient. It comes with a developer-friendly [API](https://keen.io/docs/) that covers the whole process from collecting data, analyzing it and visualizing it. It also comes with [SDKs](https://keen.io/docs/sdks/) for a great variety of technologies.

In this guide, we are going to build a reactive dashboard from scratch by building a Ruby on Rails 5 web applciation using the newly-introduced [ActionCable](https://github.com/rails/rails/tree/master/actioncable) and [Keen.IO's Ruby SDK](https://github.com/keenlabs/keen-gem). Let's get started!


# Setting up

## Getting your API credentials 
  Go to [Keen.IO](https://keen.io/)'s website and register your free account. Once you register, log in and create your first project. Once you've created it, in its  overview tab, you will have the credentials of your project:
 
- Project ID: The unique identifier of your project
- API credentials:
  - Write key - used when you want to publish data to your collections
  - Read key - used for running queries and reading data from your collections
  - Master key - used for deleting records and collections 
  

### Connecting your Rails app with Keen.IO

 First, open your terminal and make a new Rails 5 application. You need to have [Ruby 2.2.4](https://www.ruby-lang.org/en/) and up in order to do that : 
 
 ```bash
rails _5.0.0_ new reactivedashboard
 ```
 Once the application is generated, add the following gems to your <code>Gemfile</code>
 
 
 
 Connecting to Keen.IO is done with the standard approach with using environment variables. [Environment variables](http://railsapps.github.io/rails-environment-variables.html) are used to configure an application's behaviour in different environments (development, staging, production, etc.). The environment variables must **always** kept secret, since some of them may give out sensitive information about your application.
 

  
  

1 rails _5.0.0_ new reactivedashboard

2.cd reactivedashboard

3. gemfile -
    gem "keen"
    gem "dotenv-rails"
    gem 'bootstrap-sass', '~> 3.3.6'
    gem 'sass-rails', '>= 3.2'

4. make .env file and add credentials
5. include js "//cdn.jsdelivr.net/keen.js/3.4.1/keen.min.js"
6. https://keen.io/docs/data-analysis/?s=gh-gem - overview, good to have
7. rails g scaffold product price:decimal name:string description:text favorites:integer
8. rails db:migrate
8.5 
```
class Product < ApplicationRecord
  after_create_commit { Keen.publish self }
  after_destroy { Keen.delete self} 
end
```

9. create action
  def dashboard
  end


  root to: 'products#dashboard'
  and a view
  

adding should work now, check console


11. channel analytics
 rails g channel analytics

12. rails g job UpdateAnalytics
13. 
```
 class UpdateAnalyticsJob < ApplicationJob
  queue_as :default
  def perform
    ActionCable.server.broadcast 'analytics_channel' 
  end
end
```

15. show dash and how to create queries. create 3 sample queres
16. client-side cable
17. bs
