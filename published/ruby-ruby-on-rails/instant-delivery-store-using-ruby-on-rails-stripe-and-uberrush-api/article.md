In this guide, I will walk through how to build a simple store front that offers on-demand delivery. Since UberRUSH is currently only available in San Francisco, Chicago and New York City, our store will be based in Manhattan.

This application will be built using Ruby on Rails on the backend, an embedded Stripe to simulate store checkout, as well as an API wrapper for UberRUSH. In this tutorial, I assume that readers are proficient with Rails, Stripe, and configuration.

For simple store setups, UberRUSH's API has just a few necessary endpoints. [Checkout the documentation here](https://developer.uber.com/docs/rush). It is thorough and easy to read.


-----
### App Mockups

I will be creating a simple shoe store with a home page and a product page.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/32c5a47b-b47f-4d33-9b01-aa2911d1f7a8.45)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/489e5305-e125-486f-a369-e60ecbc479b3.44)



------

### Setting up the Rails App


First, let's initialize a new Rails application and open it in a text editor.

    $ rails new instant_delivery

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

We will use a single Products controller to handle all the actions. For example, there will be an index action for all the products, a show action for the individual product, and an action to get an UberRUSH quote. There will also be actions to charge a credit card and create an UberRUSH delivery and finalize/confirm an order.

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
    
Here are the routes we will be using for the application. Based on the initial mockups, I will only have 3 views (index, show, and done). The rest of the controller's actions will use AJAX.

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

**I will be using the Picnic CSS library, since every other tutorial uses Bootstrap.** Add this to the head of your application layout.

    # views/layouts/application.html.erb
    
      <link rel="stylesheet" type="text/css" href="https://cdn.jsdelivr.net/picnicss/6.1.1/picnic.min.css">

Add the following to your application CSS file. This will be some extra styling for the index and show page.

    # assets/stylesheets/application.css
    
    .main {
      padding-top: 100px;
      width: 90%;
      margin: 0 auto;
    }
    
    .hidden {
      display: none;
    }
    
    .logo {
      width: 100%;
    }



Here is the code for the index file. 

    # views/products/index.html.erb
    
    <div class="flex one three-1000">
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
      <section class="main">
          <%= yield %>
      </section>
    </body>


The root path should look something like this depending on your seed data:

![Index Page Screenshot](https://raw.githubusercontent.com/pluralsight/guides/master/images/52b756fb-cd27-468f-b4aa-804aae99550b.png)


Here is the code for the show file. We will only be using the bare minimum data for UberRUSH deliveries. I will hide the "quote-info" div for now. It will show and populate on an AJAX call later.

    <div class="flex one two-1000">
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
        <img src="https://s31.postimg.org/cjyo11eqj/uber_rush.png" class="logo"/>
        <p>
            <small>UberRUSH is an on-demand delivery network powered by Uber. Once your order is picked up, you’ll receive a text message with a link to track your delivery in real time on the map. So you always know exactly when it will arrive. </small>
        </p>
        
        <div id="shipping-quote">
          <%= form_tag "/quote", method: 'post', remote: true do %>
            <%= text_field_tag :address, nil, placeholder: "Street Address" %>
            <%= text_field_tag :city, nil, placeholder: "City" %>
            <%= text_field_tag :postal_code, nil, placeholder: "Postal Code" %>
            <%= submit_tag "Get a Delivery Quote" %>
          <% end %>
        </div>
        <br />
    
        <div id="quote-info" class="hidden">
          <h3>Quote Details</h3>
          <p>Cost Estimate: <span id="shipping-cost"></span></p>
          <p>Time Estimate: <span id="eta"></span></p>
        </div>
    
      </div>
    </div>


The show path should look like this:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2fce40ae-bce8-420e-89a5-eb7dafbbc2bb.26)


The forms on this page will be using AJAX calls and jQuery for DOM (Document Object Model) manipulation.


Our initial views are now complete. Let's get the API in the mix and start getting some quotes.

---

### UberRUSH API Wrapper 

I will show you the code shortly for the Ruby API wrapper that will communicate with UberRUSH. First, let's add some gems that we will need.

    # Gemfile
    
    gem 'httparty'
    gem 'stripe'
    gem 'figaro'
    
Bundle-install these. [httparty](https://github.com/jnunemaker/httparty) is for the API wrapper, [Stripe](https://github.com/stripe/stripe-ruby) is for credit card payments, and [Figaro](https://github.com/laserlemon/figaro) is for environment variables.

Here is the API wrapper code. The major points are as follows:
1. I set the pickup location to a single address.
2. The base uri is the UberRUSH sandbox API. You will have to change that in production.
3. To get the Uber Client ID / Client Secret, you need to [create an Uber Developer Account](https://developer.uber.com/).
4. The API requires an OAuth token on each call.
5. I have included all the basic endpoints, though we will only need to use 2 of them for this tutorial.
6. Deliveries need a pickup location object, a dropoff location object and an items array. Read more on the [official documentation](https://developer.uber.com/docs/rush).


    # config/initializers/uber_rush.rb
    
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
    
      class RUSH
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
        def get_deliveries
          url = "#{base_path}"
          self.class.get(url)
        end
    
        # Gets one specified delivery on your account
        def get_delivery(id)
          url = "/#{id}#{base_path}"
          self.class.get(url)
        end
    
        # Returns a delivery quote
        def create_quote(pickup_obj = Uber::PICKUP, dropoff_obj)
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
    
We need to update our controller code so that the form can call the API wrapper. To accomplish this, I am creating a new instance of the UberRUSH class and generating an OAuth token. We then set the pickup location to our default store address, and populate the dropoff object with the user inputs. UberRUSH is highly location-dependent, so these examples are coded to work in Manhattan. Tweak the code if you want to try SF / Chicago.

We take the first quote from the UberRUSH API response, since the first one is for on-demand delivery. The other quotes are for each available delivery window later in the day. Check out the developer documentation if you are interested in learning more about future scheduling deliveries.

    # controller/product_controller.rb
    
      def quote
        ur = Uber::RUSH.new
    
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
            @dropoff = dropoff_obj
          end
        end
      end

Let's add some JavaScript and see how the API works. We will extract the fee amount, and add the pickup and dropoff ETA to come up with an estimate.

    # views/products/quote.js.erb
    
    $('#shipping-cost').html("<%= number_to_currency(@quote["fee"]) %>");
    $('#eta').html("<%=  @quote["pickup_eta"] + @quote["dropoff_eta"] %> minutes");
    
    $('#shipping-quote').hide();
    $('#quote-info').show();


Go to the product page and enter an address in Manhattan.





![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ef3cf37b-4f27-48d1-979f-df4b1645bb98.28)


Click 'Get a Delivery Quote' to see the results.




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6eaef89e-5beb-4999-808c-c1da75b6ecba.28)




Awesome, we've got a quote! 

In a production application, you should calculate a target time and round-up to account for traffic and other potential time delays. [UberRUSH's design guidelines](https://d1a3f4spazzrp4.cloudfront.net/uberex/UberRUSH_API_guidelines_v2.pdf) have a much more detailed work-flow for professional-grade applications. This tutorial is just to cover the basics.

In the next section, we are going to add Stripe, create a sandbox UberRUSH delivery (instead of just a quote), and simulate the checkout process.

---

### Stripe and Checkout

Create a Stripe account, if you do not already have one, and put the **test publishable key** and **test secret key** into your environment variables. Add a config file to the project.

    # config/initializers/stripe.rb

    Rails.configuration.stripe = {
      publishable_key: ENV['stripe_publishable_key'],
      secret_key: ENV['stripe_secret_key']
    }
    
    Stripe.api_key = Rails.configuration.stripe[:secret_key]

Now we can update our views. Add this form below the "quote-info" div. The hidden field tags will pass along some info that is necessary for the UberRUSH delivery.

    # views/products/show.html.erb
    
      <div id="checkout" class="hidden">
        <h3>Order Product</h3>
        <%= form_tag "/order", method: 'post' do %>
          <%= text_field_tag :first_name, nil, placeholder: "First Name" %>
          <%= text_field_tag :last_name, nil, placeholder: "Last Name" %>
          <%= text_field_tag :email, nil, placeholder: "Email" %>
          <%= text_field_tag :number, nil, placeholder: "Cell Phone" %>
          <%= text_field_tag :address, nil, placeholder: "Street Address" %>
          <%= text_field_tag :city, nil, placeholder: "City" %>
          <%= text_field_tag :postal_code, nil, placeholder: "Postal Code" %>

          <%= hidden_field_tag :title , @product.title %>
          <%= hidden_field_tag :price , @product.price %>

          <script src="https://checkout.stripe.com/checkout.js" class="stripe-button"
              data-key="<%= Rails.configuration.stripe[:publishable_key] %>"
              data-description="Shoes"
              data-amount="">
          </script>
        <% end %>
      </div>

Let's update our JavaScript to return the first form's parameters to the second form. This minimizes the number of inputs that the user or customer has to add.

    # views/products/quote.js.erb

    $('#shipping-cost').html("<%= number_to_currency(@quote["fee"]) %>");
    $('#eta').html("<%=  @quote["pickup_eta"] + @quote["dropoff_eta"] %> minutes");
    
    $('#shipping-quote').hide();
    $('#quote-info').show();
    $('#checkout').show();
    
    $('input[name=address]').val('<%= @dropoff[:location][:address]  %>');
    $('input[name=city]').val('<%= @dropoff[:location][:city]  %>');
    $('input[name=postal_code]').val('<%= @dropoff[:location][:postal_code]  %>');

The page should now look like this once a quote is entered.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b2da1557-12fe-4ccb-91e2-a7bc9d420206.28)

Now let's add the controller logic to combine the UberRUSH delivery with the Stripe charge. The logic is as follows:

1. Stripe will create a customer and validate the card. If that fails, the page will refresh.
2. An UberRUSH delivery is created with the store pickup, the user inputted dropoff location and the shoe information. Our call will work for a single item at a time.
3. The total is calculated using the price of the object plus the fee from the UberRUSH delivery response object.
4. Stripe will charge the customer and redirect them to the order confirmation page.


        # controllers/products_controller.rb
        
        def order
          customer = Stripe::Customer.create(
            :email => params[:email],
            :card  => params[:stripeToken]
          )
        
          ur = Uber::RUSH.new
          pickup_obj = Uber::PICKUP
        
          dropoff_obj = {
            location: {
              address: params[:address],
              city: params[:city],
              state: "New York",
              postal_code: params[:postal_code],
              country: "US",
            },
            contact: {
              first_name: params[:first_name],
              last_name: params[:last_name],
              email: params[:email],
              phone: {
                number: "+1" + params[:number],
                sms_enabled: true
              }
            }
          }
        
          items = [
            {
              title: params[:title],
              quantity: 1,
              price: params[:price]
            }
          ]
        
          response = ur.create_delivery(items, pickup_obj, dropoff_obj)
          @amount = ((params[:price].to_f + response["fee"]) * 100).to_i
        
          charge = Stripe::Charge.create(
            :customer    => customer.id,
            :amount      => @amount,
            :description => params[:title],
            :currency    => 'usd'
          )
        
          redirect_to done_path(amount: @amount)
        
        rescue Stripe::CardError => e
          flash[:error] = e.message
          redirect_to :back
        
        end
        

Add the order completed page.

    # view/products/done.html.erb
    
    <h1>Thank you for the order!</h1>
    
    <p>Total Cost with Shipping: <strong><%= number_to_currency(params[:amount].to_i / 100.00) %></strong></p>
    
    <p>Your Shoes will Arrive: <strong><%= params[:time] %> minutes</strong></p>
    
    <p>A copy of the invoice has been sent to your email, and you will recieve SMS updates from the UberRUSH.</p>



Let's test it out with some data. The CC info will validate on Stripe for testing purposes.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/383cc27c-9e66-44e3-8216-afa804165228.41)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b4ee0009-315a-408d-80c6-2a0aae97d515.41)


Stripe will send an invoice via email and Uber will provide SMS updates to the customer. However, all the APIs are currently in test/sandbox mode, so neither the Stripe message nor the Uber message will actually be created just yet.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/676e9462-d474-42f1-9d2c-c6d6ec194326.41)


That's it! Our project is complete.

This was my first coding tutorial, so there is a lot more that could be added in regards to validation and error-checking (and of course TDD). I wanted to create an API wrapper and show a basic use case for it. Feel free to leave feedback in the comments below or fork this guide. I skipped over some minor details, since I assumed all the readers would have some experience with Rails, Stripe and configuration.

[The Github code is here.](https://github.com/ty-shaikh/uber-rush-tutorial)

Thank you for reading!

