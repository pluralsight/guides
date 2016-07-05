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

Here is the code for the index file. There is a responsive navigation with all the products displayed as cards.

    # views/products/index.html.erb
    
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
    
Add this to your application CSS file to make the index page look nice.

    # assets/stylesheets/application.css
    
    .main {
      padding-top: 100px;
      width: 80%;
      margin: 0 auto;
    }

It should look like this:

![Index Page Screenshot](https://raw.githubusercontent.com/pluralsight/guides/master/images/52b756fb-cd27-468f-b4aa-804aae99550b.png)


Here is the code for the show file.

Here is the code for the done file.












##### API Wrapper / UberRUSH

##### Stripe

