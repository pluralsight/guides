In this guide, I will walk through how to build a simple store front that offers on-demand delivery. Since UberRUSH is currently only available in San Francisco, Chicago and New York City, the store will be based in Manhattan.

This application will be built using Ruby on Rails on the backend, an embedded Stripe from to simulate store checkout as well as an API wrapper for UberRUSH. For simple store setups, UberRUSH's API only has a few necessary endpoints. [Checkout the documentation here](https://developer.uber.com/docs/rush), it is through and easy to read.



### Setting up the Rails App
____

First, let's initialize an empty Rails application and open it in your text editor.

    $ rails new instant_delivery

##### Models

Create a model for our products.

    $ rails generate model Product title:string description:text price:float picture:string
    $ rake db:migrate
    
Create some seed data. I will be creating a shoe store and borrowing content from Zappos.

    # db/seeds.rb

    Product.create(title: 'Nike Flex 2016 RN', description: "Whether you're training for your first race or are just getting back into a gym routine, the Nike Flex 2016 RN will help keep you on stride.", price: 80.00, picture: "http://a1.zassets.com/images/z/3/5/8/8/3/9/3588391-p-MULTIVIEW.jpg")
    Product.create(title: 'PUMA Roma Texture', description: "The classically styled Roma gets a new skin and becomes the Roma Texture!", price: 65.00, picture: "http://a2.zassets.com/images/z/3/6/4/8/2/3/3648235-p-MULTIVIEW.jpg")
    Product.create(title: 'Supra Skytop', description: "Uppers of leather, suede, abrasion-resistant TUF canvas, or duct tape-like TUF materials.", price: 115.00, picture: "http://a3.zassets.com/images/z/3/5/1/8/7/9/3518799-p-MULTIVIEW.jpg")
    Product.create(title: 'Converse Chuck Taylor All Star High Street Mid', description: "As the seasons change, so do these shoes! Created with kurim, a durable material that changes colors depending on the angle it is viewed from, the Converse Chuck Taylor All Star High Street Mid will catch your eye every step you take. Look forward and live comfortably with Converse.", price: 70.00, picture: "http://a3.zassets.com/images/z/3/5/8/1/8/6/3581860-p-MULTIVIEW.jpg")
    Product.create(title: 'Reebok Crossfit Speed TR', description: "The multi-directional stability of the Reebok Crossfit Speed TR cross-training shoe creates a supportive base to help you exert maximum power and top speeds during even the toughest workout. Be ever ready for that next rep!", price: 99.99, picture: "http://a1.zassets.com/images/z/3/5/9/7/3/4/3597347-p-MULTIVIEW.jpg")
    Product.create(title: 'adidas Powerlift 3', description: "Dig deep and build strength from the ground up in the adidas Powerlift 3 weightlifting shoe.", price: 90.00, picture: "http://a1.zassets.com/images/z/3/5/8/9/9/0/3589900-p-MULTIVIEW.jpg")

Then seed the database.

    rake db:seed

##### Controllers



##### Routes

##### Views


##### API Wrapper / UberRUSH

##### Stripe

