Data analytics is an ever-growing trend in the software industry. An increasing number of industries rely on data in order to grow and gain competitive advantage. Almost all software products, from sleep cycle mobile apps to enterprise logistics software come coupled with in-depth analytics.  

However, making sense out of data tends to be a tedious, long and volatile process. Building


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
