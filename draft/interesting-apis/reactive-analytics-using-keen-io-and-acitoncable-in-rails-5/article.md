1 rails _5.0.0_ new reactivedashboard

2.cd reactivedashboard

3. gemfile -
 gem "keen"
 gem "dotenv-rails"
4. make .env file
5. include js "//cdn.jsdelivr.net/keen.js/3.4.1/keen.min.js"
6. https://keen.io/docs/data-analysis/?s=gh-gem - overview, good to have
7. rails g resource product price:decimal name:string description:text favorites:integer

```
 rails g resource product price:decimal name:string description:text favorites:number
Running via Spring preloader in process 32569
      invoke  active_record
      create    db/migrate/20160612144534_create_products.rb
      create    app/models/product.rb
      invoke    test_unit
      create      test/models/product_test.rb
      create      test/fixtures/products.yml
      invoke  controller
      create    app/controllers/products_controller.rb
      invoke    erb
      create      app/views/products
      invoke    test_unit
      create      test/controllers/products_controller_test.rb
      invoke    helper
      create      app/helpers/products_helper.rb
      invoke      test_unit
      invoke    assets
      invoke      coffee
      create        app/assets/javascripts/products.coffee
      invoke      scss
      create        app/assets/stylesheets/products.scss
      invoke  resource_route
       route    resources :products
```
8. rails db:migrate
9. create action
  def dashboard
    @products = Product.all
  end


  root to: 'products#dashboard'
  and a view
  
10. create a form new product and insert it
11. channel
```
rails g channel products
Running via Spring preloader in process 32900
      create  app/channels/products_channel.rb
   identical  app/assets/javascripts/cable.js
      create  app/assets/javascripts/channels/products.c
```
12. rails g job RecordEvent
13. 