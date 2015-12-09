So, I'm working on a personal project to learn Ruby on Rails and the application structure that I desired required a complicated many-to-many relationship with a join model that itself contained data. This was a pretty complex model structure to setup and has numerous pitfall points that took a weekend of searching the Googles and reading a number of StackOverflow entries, GitHub gem documentation and RailsCasts to finally understand and get working the way I desired. Since all the documentation I found only dealt with small pieces of the whole and it took me all weekend to figure it out, I got to thinking there's no way I'm the only one out there trying to grok this crap. So, now that I got it working, I'm going to share how the heck to do it so you can learn from my guinea pigging.

## Application Summary
In this example, we'll be setting up a Recipe book. Recipes have multiple meta properties like their title, a brief description, instructions and the like. Each recipe of course needs ingredients and each ingredient needs quantities of that ingredient for the recipe. What we will be setting up is a relationship of Recipes to Ingredients through Quantities.

## Setting Up Your Models

`recipe.rb`

``` ruby
class Recipe < ActiveRecord::Base
  attr_accessible :title,
                  :description,
                  :instructions,
                  :quantities_attributes

  has_many :quantities
  has_many :ingredients,
           :through => :quantities

  accepts_nested_attributes_for :quantities,
           :reject_if => :all_blank,
           :allow_destroy => true
  accepts_nested_attributes_for :ingredients
end
```


### Adding the join model attributes to the mass assignment white-list

Our Recipe model contains the most configuration to set things up. As is expected, the `attr_accessible` property contains the properties of the Recipe model itself that can be modified, with the addition of a new property `:quantities_attributes`. This property will allow modification of the Recipe model (updating and creating) to also modify attributes of the associated Quantities records.

### Glueing the models to each other through a join model


``` ruby
has_many :quantities
has_many :ingredients, :through => :quantities
```


It used to be a complete pain in the butt to setup many to many relationships with Ruby on Rails and `has_and_belongs_to_many` configurations. Now, with Rails 3 though its super easy via the [has_many, through](http://guides.rubyonrails.org/association_basics.html#the-types-of-associations) property. This allows you to easily setup a relationship from one model to another with a join model between them to easily provide access to the models from each other. In this case for example `@recipe.ingredients` and with the `through` command on the Ingredient model end `@ingredient.recipes`.

### Setting up the model's ability to modify other model attributes


``` ruby
accepts_nested_attributes_for :quantities,
    :reject_if => :all_blank,
    :allow_destroy => true
accepts_nested_attributes_for :ingredients
```

With the `accepts_nested_attributes_for` model property you can specify that a model will be able to also accept attributes for some of its relational models as setup through the has_many associations. You can also define rejection and deletion controls for the relational model through this method. See the official [Active Record Nested Attributes](http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html) documentation for more information.

`quantity.rb`

``` ruby
class Quantity < ActiveRecord::Base
  attr_accessible :amount,
                  :ingredient,
                  :ingredient_attributes

  belongs_to :recipe
  belongs_to :ingredient

  accepts_nested_attributes_for :ingredient,
                                :reject_if => :all_blank
end
```


### Adding the join model attributes to the ...join model attribute white-list


``` ruby
attr_accessible :amount,
    :ingredient,
    :ingredient_attributes
```


Here we add the `:ingredient` and `:ingredient_attributes` to the mass-assignment white-list of the join model. This allows modification of the join model (Quantity) through the parent model (Recipe) to also modify and create entries in the final relational model (Ingredient). Note that the mass-assignment white-list on the join model also includes its own attribute of an `:amount`, something that using a `has_many, through` relationship gives us the ability to do.

### Creating the join relationship


``` ruby
belongs_to :recipe
belongs_to :ingredient
```

As this is a many to many relationship, there will be many recipes and many ingredients, but there will only be _one_ quantity of _an_ ingredient (or multiple ingredients) for each recipe. The belongs_to specification will explain to Ruby on Rails that each association of an Ingredient to a Recipe will be joined together by a record in the Quantity join table.

### Allow the join model to modify its relational model


``` ruby
accepts_nested_attributes_for :ingredient,
    :reject_if => :all_blank
```


Setting up a similar accepts_nested_atrributes_for specification on the join model allows the join model (Quantity) to modify properties of the final relational model (Ingredient) so when a Recipe is saved with a nested form, the individual Quantity entries can have Ingredient properties and modify those Ingredient properties.

`ingredient.rb`

``` ruby
class Ingredient < ActiveRecord::Base
  attr_accessible :name

  has_many :quantities
  has_many :recipes, through: :quantities
end
```

The final relational model doesn't have a whole lot unique to it with the exception of the specification of the `has_many :recipes, through: :quantities` definition. This allows easy access to the recipes that an ingredient is associated to via simple `@ingredient.recipes` reference.

## Installing Cocoon to make nested forms easy-peasy

To make setting up a nested form for a Recipe that can have 1 or more Ingredients each with a Quantity easy, we use a little gem called [Cocoon](https://github.com/nathanvda/cocoon). Cocoon provides drop-in JavaScript functionality to easily add and remove multiple entries of Ingredients to your nested form. As an added bonus, Cocoon can be combined with the [Formtastic](https://github.com/justinfrench/formtastic) or [Simple Form](https://github.com/plataformatec/simple_form) gems if you want to make your forms building even easier. To install Cocoon, just add it to your Rails project's Gemfile:

``` ruby
gem 'cocoon'
```

After adding this line to your Gemfile, just run `bundle install` to get the gem installed in your application. With Cocoon installed now, just add this line to your application.js file to include Cocoon's JavaScript to your asset pipeline:

``` ruby
//= require cocoon
```

This will allow all of Cocoon's jQuery magic to do its thing.

## Setting up the form code

The tricky part of getting all this model magic to work of course is getting your nested form setup. Nested forms are a pain in the butt, but hopefully this will help get you rolling on your project. You can build your form with the helper of your choice, but I'm going to just use Rails' built in form helpers for this example to focus on the core of the setup instead of muddying the waters with a third-party helper. The only third party structure you may notice here is the Twitter Bootstrap structure (which, if you _do_ want to use Formtastic, there is a handy-dandy [Formtastic Bootstrap](https://github.com/mjbellantoni/formtastic-bootstrap) gem to modify its output to match Bootstrap's structure).

``` ruby
<%= form_for @recipe, html: {class: "form-horizontal"} do |f| %>
  <fieldset id="recipe-meta">
    <ol>
      <li class="control-group">
        <%= f.label :title, "Recipe Name", class: "control-label" %>
        <div class="controls"><%= f.text_field :title %></div>
      </li>
      <li class="control-group">
        <%= f.label :description, "A brief description of this recipe", class: "control-label" %>
        <div class="controls"><%= f.text_area :description, rows: 5 %></div>
      </li>
      <li class="control-group">
        <%= f.label :instructions, "Instructions for this recipe", class: "control-label" %>
        <div class="controls"><%= f.text_area :instructions, rows: 10 %></div>
      </li>
    </ol>
  </fieldset>

  <fieldset id="recipe-ingredients">
    <ol>
      <%= f.fields_for :quantities do |quantity| %>
        <%= render 'quantity_fields', f: quantity %>
      <% end %>
    </ol>
    <%= link_to_add_association 'add ingredient', f, :quantities, 'data-association-insertion-node' => "#recipe-ingredients ol", 'data-association-insertion-method' => "append", :wrap_object => Proc.new {|quantity| quantity.build_ingredient; quantity } %>
  </fieldset>

  <%= f.submit %>
<% end %>
```


### The nested form

All of this form is pretty standard until you get to the nested portion, then it gets a bit tricky, so I'll walk through each piece.


``` ruby
<%= f.fields_for :quantities do |quantity| %>
  <%= render 'quantity_fields', f: quantity %>
<% end %>
```


This loop creates all the form entry fields for your ingredients and their quantity amounts through a partial named after the Quantities themselves. The partial name here is important for Cocoon to work properly: `_[model]_fields.html.erb`.


``` ruby
<%= link_to_add_association 'add ingredient', f, :quantities,
    'data-association-insertion-node' => "#recipe-ingredients ol",
    'data-association-insertion-method' => "append",
    :wrap_object => Proc.new {|quantity| quantity.build_ingredient; quantity } %>
```


This a new helper method introduced by Cocoon to help create additional Quantity/Ingredient fields in your form. Not all of the properties shown here on the `link_to_add_association` method are required, but some are necessary for our form interaction to work properly, notably the `:wrap_object` property:


``` ruby
:wrap_object => Proc.new {|quantity| quantity.build_ingredient; quantity }
```

This property will allow Cocoon to create a new, empty Ingredient object associated with the added Quantity entry on the form. Basically, without it you can't add new ingredients, just a Quantity amount associated with no ingredient.

The `data-association-insertion-node` and `data-association-insertion-method` properties allow the "add" button to properly append new Quantity/Ingredient fields to our form structure. The nice thing is, once you have all this configured here, Cocoon does all the rest of the work - not a single line of JavaScript required.

### The nested form partial

`_quantity_fields.html.erb`


``` ruby
<li class="control-group nested-fields">
  <div class="controls">
    <%= f.label :amount, "Amount:" %>
    <%= f.text_field :amount %>

    <%= f.fields_for :ingredient do |quantity_ingredient| %>
      <%= quantity_ingredient.text_field :name %>
    <% end %>

    <%= link_to_remove_association "remove", f %>
  </div>
</li>
```


The nested partial here will be used with the output form and the template for any added Quantity/Ingredient fields to your form interface. The unique piece of this template added for Cocoon is the link_to_remove_association method. This method automatically looks for the [closest()](http://api.jquery.com/closest/) element to it with the `nested-fields` class and will remove the fields from the form when clicked and therefor from the database record upon form submission. These properties cannot be modified, so make sure that the `link_to_remove_association` method is contained within an element in the partial file with the `nested-fields` class.

## Voila!

That's it! Now that we have sprinkled the appropriate Ruby pixie dust in the correct areas, the Rails framework will take over the rest of the work. Ain't Rails beautiful?

