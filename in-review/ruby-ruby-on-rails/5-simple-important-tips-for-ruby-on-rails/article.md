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

I include concerns every time I'm using Rails 4. They help ensure things happen in your models, similarly to callbacks, but in a much more cleaner way. 

**Extract Complex Methods**

I cannot stress this enough! Complex methods are hard to read, they are hell for new developers joining to work on your app. They're also hard to maintain and even tougher when you want to add more stuff. Splitting complex methods will make your life much easier and will give you better, cleaner more robust code base. 


**Lean Controllers** 

So I used to be one of those guys that put everything in the Controllers. We've all done it (and sometimes still do it). But lean controllers are really important, to maintain speed and stability within your app. Refactor everything you can to the model, I bet 40% of your controller code belongs in the model!

____________________________

Check out the guides posted below to learn more about specific Ruby on Rails 4 and Ruby on Rails 5 functions and implementation tips to optimize web development.

- [Handling file upload using RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/handling-file-upload-using-ruby-on-rails-5-api)

- [Using token-based authentication with RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api)

- [Creating a chat using Rails Action Cable.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/creating-a-chat-using-rails-action-cable)

_______

Did you find my tips useful? Feel free to post your feedback in the comments section below!