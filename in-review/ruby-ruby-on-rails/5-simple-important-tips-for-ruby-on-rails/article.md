Ruby on Rails is a popular framework used to create online applications. Below I've listed some tips for newcomers to the Ruby system. 

**Callbacks** - Use them wisely! 

What are callbacks? Callbacks are methods that play a [specific role](http://guides.rubyonrails.org/active_record_callbacks.html) in an object's life cycle. They can be really useful. For instance, using callbacks could help satisfy initial conditions placed on a certain object class, as seen in [this example.](https://www.bignerdranch.com/blog/the-only-acceptable-use-for-callbacks-in-rails-ever/)

However, using a lot of callbacks, or especially chained callbacks can be problematic. A failed callback can leave your model in an invalid state, which could compound with other callbacks failing and lead to pesky app malfunctions. 

To sum, using callbacks wisely can lend to cleaner, less buggy, and more readable code.

**Scopes** - Use more scopes!

[Scopes](http://api.rubyonrails.org/classes/ActiveRecord/Scoping/Named/ClassMethods.html) are excellent in Rails. Scoping allows us to find and access various RoR objects based on particular tags. 
- An example of effective scoping:

```Rails
class Train < ActiveRecord::Base

    scope :freight,-> { (where(type: freight)) } 
    //find all freight trains
    
    scope :coal_powered,-> { joins(:fueling_type).where('fueling_type.coal_powered = ?', true) } 
    //find all coal-powered freight trains

end 
```

Scopes keep code readable, and they should be part of every model. 

**Concerns** 

I include concerns every time I'm using Rails 4. They help ensure that designated functions actually work in your models. In fact, they operate like callbacks do but are often cleaner, since callbacks are commonly overused.

# Add example here 

**Extract Complex Methods**

I cannot stress this enough! Complex methods are hard to read, and they are hell for developers new to your app. They're also hard to debug and even tougher to expand. Splitting complex methods into helper functions will make your life much easier and give you a cleaner, more robust code base. 

# Add example here


**Lean Controllers** 

So I used to be one of those guys who put everything into [Controllers](http://www.tutorialspoint.com/ruby-on-rails/rails-controllers.htm). Pretty much every Rails user has done this at some point in time (and sometimes still do it). But **lean controllers** are really important in maintaining speed and stability within your app. Refactor everything you can to the model; from my experience, 30-40% of controller code actually belongs in the model!

____________________________

Check out the guides posted below to learn more about specific Ruby on Rails 4 and Ruby on Rails 5 functions and implementation tips to optimize web development.

- [Handling file upload using RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/handling-file-upload-using-ruby-on-rails-5-api)

- [Using token-based authentication with RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api)

- [Creating a chat using Rails Action Cable.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/creating-a-chat-using-rails-action-cable)

_______

Did you find my tips useful? Feel free to post your feedback in the comments section below!