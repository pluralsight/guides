In this guide, I will walk through how to build a simple store front that offers on-demand delivery. Since UberRUSH is currently only available in San Francisco, Chicago and New York City, the store will be based in Manhattan.

This application will be built using Ruby on Rails on the backend, an embedded Stripe from to simulate store checkout as well as an API wrapper for UberRUSH. For simple store setups, UberRUSH's API only has a few necessary endpoints. [Checkout the documentation here](https://developer.uber.com/docs/rush), it is through and easy to read.



# App mockups----------


# ------

### Setting up the Rails App


First, let's initialize an empty Rails application and open it in your text editor.

    $ rails new instant_delivery
    $ cd instant_delivery
    $ atom .

---
#### Models

Create a model for our products.

    $ rails generate model Product title:string description:text price:float picture:string
    $ rake db:migrate
    
Create some seed data. I will be creating a shoe store and borrowing content from Zappos.

    # db/seeds.rb

    Product.create(title: 'Nike Flex 2016 RN', description: "Whether you're training for your first race or are just getting back into a gym routine, the Nike Flex 2016 RN will help keep you on stride.", price: 80.00, picture: "http://a1.zassets.com/images/z/3/5/8/8/3/9/3588391-p-MULTIVIEW.jpg")
    Product.create(title: 'PUMA Roma Texture', description: "The classically styled Roma gets a new skin and becomes the Roma Texture!", price: 65.00, picture: "http://a2.zassets.com/images/z/3/6/4/8/2/3/3648235-p-MULTIVIEW.jpg")
    Product.create(title: 'Supra Skytop', description: "Uppers of leather, suede, abrasion-resistant TUF canvas, or duct tape-like TUF materials.", price: 115.00, picture: "http://a3.zassets.com/images/z/3/5/1/8/7/9/3518799-p-MULTIVIEW.jpg")
    Product.create(title: 'Converse Chuck Taylor Mid', description: "As the seasons change, so do these shoes! Created with kurim, a durable material that changes colors depending on the angle it is viewed from, the Converse Chuck Taylor All Star High Street Mid will catch your eye every step you take. Look forward and live comfortably with Converse.", price: 70.00, picture: "http://a3.zassets.com/images/z/3/5/8/1/8/6/3581860-p-MULTIVIEW.jpg")
    Product.create(title: 'Reebok Crossfit Speed TR', description: "The multi-directional stability of the Reebok Crossfit Speed TR cross-training shoe creates a supportive base to help you exert maximum power and top speeds during even the toughest workout. Be ever ready for that next rep!", price: 99.99, picture: "http://a1.zassets.com/images/z/3/5/9/7/3/4/3597347-p-MULTIVIEW.jpg")
    Product.create(title: 'adidas Powerlift 3', description: "Dig deep and build strength from the ground up in the adidas Powerlift 3 weightlifting shoe.", price: 90.00, picture: "http://a1.zassets.com/images/z/3/5/8/9/9/0/3589900-p-MULTIVIEW.jpg")

Then seed the database.

    rake db:seed

---
#### Controllers

We will use a single Products controller to handle all the actions. There will be an index page for all the products, a show page for the individual product, a route to get an UberRUSH quote, a route to charge the credit card and create an UberRUSH delivery and final an order confirmation page.

I will fill out the other actions that deal with the API later.

    # controllers/products_controller.rb
    
    class ProductsController < ApplicationController
      def index
        @products = Product.all
      end
    
      def show
        @product = Product.find(params[:id])
      end
    
      def quote
        # code for UberRUSH shipping quote
      end
    
      def order
        # code for Stripe charge and UberRUSH delivery creation
      end
    
      def done
        # code for confirmation page
      end
    end

---
#### Routes
    
Here are the routes we will be using for the application. Based on the initial mockups, I will only have 3 views and the rest of the controller actions will be using AJAX.

    # config/routes.rb
    
    Rails.application.routes.draw do
      root to: 'products#index'
    
      resources :products, only: [:show]
    
      post 'quote', to: 'products#quote'
      post 'order', to: 'products#order'
      get 'done', to: 'products#done'
    end

---
#### Views

In the views folder, create a Products folder and 3 view files.

    touch index.html.erb
    touch show.html.erb
    touch done.html.erb

I will be using the Picnic CSS library, since every other tutorial uses Bootstrap. Add this to the head of your application layout.

    # views/layouts/application.html.erb
    
      <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/picnicss/6.1.1/picnic.min.css">

Add this to your application CSS file. This will be some extra styling for the index and show page.

    # assets/stylesheets/application.css
    
    .main {
      padding-top: 100px;
      width: 80%;
      margin: 0 auto;
    }
    
    #quote-info {
      display: none;
    }

Here is the code for the index file. 

    # views/products/index.html.erb
    
    <section class="main">
      <div class="flex three">
        <% @products.each do |product| %>
          <article class="card">
            <img src="<%= product.picture %>">
            <footer>
              <h3><%= product.title %></h3>
              <%= link_to "Deets", product_path(product), class: "button" %>
            </footer>
          </article>
        <% end %>
      </div>
    </section>

Create a navigation partial.

    # views/layouts/_navigation.html.erb
    
    <nav>
      <a href="/" class="brand">
        <span>Instant Kicks</span>
      </a>
    
      <!-- responsive-->
      <input id="bmenub" type="checkbox" class="show">
      <label for="bmenub" class="burger pseudo button">menu</label>
    
      <div class="menu">
        <a href="/" class="pseudo button icon-picture">Products</a>
        <a href="#" class="button icon-puzzle">Contact</a>
      </div>
    </nav>

Add it to the main layout file.
    
    # views/layouts/application.html.erb
    
    <body>
      <%= render 'layouts/navigation' %>
      <%= yield %>
    </body>


The root path should look something like this depending on your seed data.

![Index Page Screenshot](https://raw.githubusercontent.com/pluralsight/guides/master/images/52b756fb-cd27-468f-b4aa-804aae99550b.png)


Here is the code for the show file.

    <section class="main">
      <div class="flex two">
        <div class="product-image">
          <img src="<%= @product.picture %>" />
        </div>
        <div>
          <h1><%= @product.title %></h1>
          <p>
            <%= @product.description %>
          </p>
    
          <h2><%= number_to_currency(@product.price) %></h2>
          <hr />
          <h3>Instant Shipping</h3>
    
          <img src="http://www.ilos.com.br/web/wp-content/uploads/UberRUSH-logo.png" />
    
          <div id="shipping-quote">
            <%= form_tag "/quote", method: 'post', remote: true do %>
              <%= text_field_tag :address, nil, placeholder: "Street Address" %>
              <%= text_field_tag :city, nil, placeholder: "City" %>
              <%= text_field_tag :postal_code, nil, placeholder: "Postal Code" %>
              <%= submit_tag "Get a Delivery Quote" %>
            <% end %>
          </div>
          <br />
    
          <div id="quote-info">
            <hr />
            <h3>Quote Details</h3>
            <p>Cost Estimate: <span id="shipping-cost"></span></p>
            <p>Time Estimate: <span id="eta"></span></p>
          </div>
    
        </div>
      </div>
    </section>

The show path should look like this. The forms on this page will be using AJAX calls and jQuery for DOM manipulation.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f56a00de-c9dc-42f3-925a-25095ccb6b69.30)



This is it for our initial views. Let's get the API in the mix and start getting some quotes.

---

### UberRUSH API Wrapper 

I will show you the code shortly for the Ruby API wrapper that will communicate with UberRUSH. First, let's add some gems that we will need.

    # Gemfile
    
    gem 'httparty'
    gem 'stripe'
    gem 'figaro'
    
Bundle install these. [httparty](https://github.com/jnunemaker/httparty) is for the API wrapper, [Stripe](https://github.com/stripe/stripe-ruby) is for credit card payments and [Figaro](https://github.com/laserlemon/figaro) is for environment variables.

Here is the API wrapper code. The major points are as follows:
1. I set the pickup location to a single address.
2. The base uri is the UberRUSH sandbox API. You will have to change that in production.
3. To get the Uber Client ID / Client Secret, you need to [create an Uber Developer Account](https://developer.uber.com/).
4. The API require OAuth tokens on each call.
5. I have included all the basic endpoints, though we will only need to use a few of them.
6. Deliveries need a pickup location object, a dropoff location object and an items array. Read more on the [official documentation](https://developer.uber.com/docs/rush).


    module Uber
    
      # Default pickup location, for single-store implementations
      PICKUP =
        {
          location: {
            address: "40 W 57th Street",
            city: "New York",
            state: "New York",
            postal_code: "10019",
            country: "US"
          },
          contact: {
            company_name: "Instant Kicks",
            email: "admin@instantkicks.com",
            phone: {
              number: "+12129541234",
              sms_enabled: true
            }
          }
        }
    
      class UberRush
        include HTTParty
        attr_reader :base_path
    
        # Sandbox URI, Change to Production when Ready
        base_uri 'https://sandbox-api.uber.com/v1/deliveries'
    
        # Gets OAuth Token on new Instance
        def initialize
          url = "https://login.uber.com/oauth/v2/token"
    
          token = self.class.post(url,
            body: {
              client_id: ENV['uber_rush_client_id'],
              client_secret: ENV['uber_rush_client_secret'],
              grant_type: 'client_credentials',
              scope: 'delivery'
            }
          )["access_token"]
    
          @base_path = "?access_token=#{token}"
        end
    
        # Gets all deliveries on your account
        def all_deliveries
          url = "#{base_path}"
          self.class.get(url)
        end
    
        # Gets one specified delivery on your account
        def one_delivery(id)
          url = "/#{id}#{base_path}"
          self.class.get(url)
        end
    
        # Returns a delivery quote
        def delivery_quote(pickup_obj = Uber::PICKUP, dropoff_obj)
          url = "/quote#{base_path}"
    
          self.class.post(url,
            body: { pickup: pickup_obj, dropoff: dropoff_obj }.to_json,
            headers: { 'Content-Type' => 'application/json' }
          )
        end
    
        # Creates a delivery and returns a delivery object
        def create_delivery(items_arr, pickup_obj = Uber::PICKUP, dropoff_obj)
          url = "#{base_path}"
    
          self.class.post(url,
            body: { items: items_arr, pickup: pickup_obj, dropoff: dropoff_obj }.to_json,
            headers: { 'Content-Type' => 'application/json' }
          )
        end
    
        # Changes delivery status to client_canceled
        def cancel_delivery(id)
          url = "/#{id}/cancel#{base_path}"
          self.class.post(url)
        end
      end
    end
    
We need to update our controller code so the form can call the API wrapper. Here I am creating a new instance of the UberRUSH class and generating an OAuth token. We then set the pickup location to our default store address, and populate the dropoff object with the user inputs. UberRUSH is highly location-dependent, so these examples are coded to work in Manhattan. Tweak the code if you want to try SF / Chicago.

We take the first quote from the UberRUSH API response, since it is the first one is for on-demand delivery. THe other quotes are for each available delivery window for the day. There is more detail in developer documation if desired.

    # controller/product_controller.rb
    
      def quote
        ur = Uber::UberRush.new
    
        pickup_obj = Uber::PICKUP
    
        dropoff_obj = {
          location: {
            address: params[:address],
            city: params[:city],
            state: "New York",
            postal_code: params[:postal_code],
            country: "US",
          }
        }
    
        response = ur.delivery_quote(pickup_obj, dropoff_obj)
    
        respond_to do |format|
          format.html
          format.js do
            @quote = response["quotes"][0]
          end
        end
      end

Let's add some JavaScript and see how the API works. We will add the pickup and dropoff ETA to come up with an estimate.

    # views/products/quote.js.erb
    
    $('#shipping-cost').html("<%=  number_to_currency)@quote["fee"]) %>");
    $('#eta').html("<%=  @quote["pickup_eta"] + @quote["dropoff_eta"] %> minutes");
    
    $('#shipping-quote').hide();
    $('#quote-info').show();


Go to the product page and enter an address in Manhattan.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/abec67dc-a2d8-42f2-8e0e-a61035c09402.31)

Click 'Get a Delivery Quote' to see the results.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b5d66215-1293-4db2-af71-0007db065abe.24)

Awesome, we have got a quote! 

In a production application, you should calculate a target time and round-up to account for traffic and other potential time delays. [UberRUSH's design guidelines](https://d1a3f4spazzrp4.cloudfront.net/uberex/UberRUSH_API_guidelines_v2.pdf) have a much more detailed work-flow. This tutorial is just to cover the basics.

In the next section, we are going to add Stripe, create a sandbox UberRUSH delivery (instead of just a quote) and simulate the checkout process.

---

### Stripe and Checkout




##### API Wrapper / UberRUSH

##### Stripe

