Ruby on Rails is a popular framework used to create online applications. Below I've listed some tips for newcomers to the Ruby system. 

**Callbacks** - Use them wisely! 

- Callbacks are really useful. They can help you achieve really nice things within your Ruby on Rails application. However, using a lot of callbacks, or especially chained callbacks can be really problematic - i.e. what would happen if one fails? Do all of them fail? 

- They will all fail, a failed callback can leave your model in an invalid state, which will lead not only to other failing callbacks, but also other parts of the app not working correctly or failing as well. 

- So use callbacks wisely!

**Scopes** - Use more scopes!

- Scopes are really really great in rails. They help maintain clean code and things are really readable. They can be used to optimize stuff with eager loading, and they should be part of every model. 

**Concerns** 

- Concerns with Rails 4 are becoming part of my coding everyday. They have helped clean out code of a really big project that I'm working on. They help ensure things happen in your models, similarly to callbacks, but in a much more cleaner way. 

**Extract Complex Methods** - I cannot stress this enough!

- Complex methods are hard to read, they are hell for new developers joining to work on your app. They're also hard to maintain and even tougher when you want to add more stuff. Splitting complex methods will make your life much easier and will give you better, cleaner more robust code base. 


**Lean Controllers** 

- So I used to be one of those guys that put everything in the Controllers. We've all done it (and sometimes still do it). But lean controllers are really important, to maintain speed and stability within your app. Refactor everything you can to the model, I bet 40% of your controller code belongs in the model!

____________________________

Check out the guides posted below to learn more about specific Ruby on Rails functions and implementation tips to optimize web development.

- [Handling file upload using RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/handling-file-upload-using-ruby-on-rails-5-api)

- [Using token-based authentication with RoR5.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/token-based-authentication-with-ruby-on-rails-5-api)

- [Creating a chat using RoR5 Action Cable.](http://tutorials.pluralsight.com/ruby-ruby-on-rails/creating-a-chat-using-rails-action-cable)

_______

Did you find my tips useful? Feel free to post your feedback in the comments section below!