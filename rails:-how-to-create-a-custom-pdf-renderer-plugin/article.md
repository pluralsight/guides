Before diving into our series on building great Web apps with Rails, check out our [tutorial on how to get started](http://blog.pluralsight.com/tutorial-rails) with the popular framework.

Let’s begin with a quick refresher on Rails. Rails is a Web framework within the Ruby programming language, and it’s also a Ruby gem. Ruby gems are comprised of code, documentation and a gemspec. The gemspec contains information on the gem including what it does and who created it. And since the <code>gemspec</code> is also Ruby code, you can wrap scripts to generate file names. Gems make it easy for Ruby programmers to share code that they want to install in their applications, and it’s this process which makes Ruby on Rails one of the most popular ways to build great Web apps. 

For Ruby Web apps, the Rails framework uses the MVC architecture for arranging your code. Typically, the controller is responsible for taking important information from the models and then sending that data to the views for rendering. However, this isn’t always the case, as you can use Rails to generate static HTML views, and you can also have a controller action generate a view with no corresponding model.

[![RailsRuby_01](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_01.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_01.png) 

[![RailsRuby_02](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_02.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_02.png) 


You can also use Bundler to create your own gems and to handle gem dependency management. 

**MVC and the Rails render() method** 

Rails can render templates and files, making it possible to also render images, raw text and various file formats like html, xml, json, and txt. Additionally, we can add more options like CSV or PDF to the Rails built-in render() method, which comes packaged as part of the Rails framework. The render() method is the common interface for rendering a given model or template. In this example we’ll look at how you can modify the render() method and return a PDF created with the Prawn gem—essentially, creating a Rails plugin that's a PDF renderer.
 
(Note: A renderer is a hook exposed by the render() method to customize its behavior. For further documentation about the Prawn gem, check out the Github repository for [Prawn](https://github.com/prawnpdf/prawn).)

Since Rails generators include a generator for writing plugins, creating the plugin that will be the PDF renderer is easy. You can create plugins for extending the Rails framework. For our PDF renderer example, hop into your terminal and run the command **rails plugin new pdf_renderer**. 

After running that command, locate your newly generated Rails plugin and run the command **ls** to see a listing of all the files in your Rails plugin project directory. You’ll see that a basic plugin structure containing a **pdf_renderer.gemspec** file, a **Rakefile**, a **Gemfile**, and **lib** and **test** directories were created. This is similar to what happens if you were to run the **rails new myapp** command to create a new Rails app titled “myapp,” which generates the skeletal structure of a new Rails application. One difference you’ll notice is when generating the plugin project, there’s a **test/dummy** directory that allows you to run your tests inside a Rails app environment. 

The **pdf_renderer.gemspec** file that was generated provides basic Ruby gem specification. This lists the gem’s author(s), version, gem dependencies, and source files. It also allows you to bundle the plugin into a Ruby gem, making it easy to share your code across different Rails apps. 

Just like with a regular Rails app, the little plugin app example (for writing a renderer) includes a Gemfile in which your gem management can be handled with Bundler. This locks our environment to use only the gems listed in both the **pdf_renderer.gemspec** file and the **Gemfile**. It’s no different than using only the gems listed in the **Gemfile.lock** file and the **Gemfile** of a regular Rails app. New or updated gem dependencies are bundled by running the **bundle install** and **bundle update** commands, respectively, from the terminal (while in the plugin app’s root). 

Booting the dummy Rails application created inside of the test directory is similar to the way we boot any other Rails application. But in this case, you boot it by running the **rails s** command in your terminal from the plugin app’s **test/dummy** directory inside the plugin project file. The boot file, located in **test/dummy/config/boot.rb**, is similar to the boot file in any other Rails app, except that this one needs to point to the **Gemfile** at the root of the plugin project, like this: [![RailsRuby_03](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_03-1024x554.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_03.png) 

You’ll also notice that the **test/dummy/config/application.rb** file is a stripped-down version of the **/config/application.rb** file in a regular Rails app:

[![RailsRuby_04](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_04-1024x556.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_04.png) 

Now, in this example of making a PDF renderer plugin using Prawn, we add the Prawn gem to our Gemfile just like we would in any other Rails app. And since the Prawn gem will be a dependency for this Rails plugin example, we also need to add it to the **pdf_renderer.gemspec** file by adding this line of code, s.add_dependency “prawn”, “0.12.0”: 

[![RailsRuby_05](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_05-1024x552.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_05.png)   

[![RailsRuby_005](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_005-1024x550.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_005.png) 

After adding the Prawn gem you’ll run the command **bundle install**, just as you would with any other Rails app, then hop into interactive Ruby by running the irb command and create a sample PDF file in irb, like this: 

[![RailsRuby_06](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_06.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_06.png) [![RailsRuby_006](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_006.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_006.png) 

In your text editor (in your project file directory that was made for writing this plugin) you’ll see a sample PDF file. In this screencap of my text editor (below), you’ll see two sample PDF files. That’s because when I created the first one, sample.pdf forgot to screencap the irb session. So, I made a second sample PDF file for illustrative purposes, this shows you what the terminal output should look like for the irb session. 

Getting back to Prawn, if you read the documentation, you'll see that Prawn has its own syntax for creating PDF files. It gives us an easy-to-use API. However, it cannot convert HTML files into PDF files. If you want the ability to do that -- and you can’t find a Ruby gem that can handle that task -- you may consider building your own Ruby gem using Bundler. Still, you can always check out available gems by going to Rubygems.org and looking for one that already has what you need. There are tons of new Ruby gems being built and uploaded to Rubygems.org daily, so there’s a high probability of finding one that will meet your app’s needs. 

Check out the terminal output of the test results for testing the code for the plugin (pictured below). As with any other Rails project, you’ll want to write at least two sanity tests. The tests and the code for this PDF renderer plugin are rather straightforward. Make a test file for your integration tests, **/test/integration/pdf_delivery_test.rb**, and write a couple of simple integration tests like this: 

[![RailsRuby_07](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_07-1024x551.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_07.png) 

Now we write the first test to verify that a PDF file is being returned when booting up the app. We do this by going to localhost: 3000/home.pdf with the expectation that it should fail until we write the code for telling Rails how to handle the pdf option in the render() method inside of the /**lib/pdf_renderer.rb** file, like this: 

[![RailsRuby_08](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_08-1024x533.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_08.png) 

You’ll also need to make a view file, **index.pdf.erb**, that corresponds to the index action (that you should have written as part of your basic CRUD methods in your dummy app’s controller). This is similar to what we do when we create the **index.html.erb** file in other Rails applications. Of course, you’ll want to add the appropriate routes, too: 

[![RailsRuby_09](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_09-1024x551.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_09.png)

Now, if you run your rake tests, you can have fun watching them pass. 

[![RailsRuby_10](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_10.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_10.png)

To see this in your browser, go to the **test/dummy** directory and start up the rails server. Then, go to localhost: 3000/home.pdf. You should see a PDF form pop up that you can edit. Here’s a screencap of mine: 

[![RailsRuby_11](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_11.png)](http://blog.pluralsight.com/wp-content/uploads/2015/11/RailsRuby_11.png) 

You’re probably wondering how Rails knew what content type to set in the return or response to our request. The answer is simple. Rails comes with a pre-packaged set of registered formats and MIME types. You may verify this by examining the heart of the Rails framework in **rails/actionpack/lib/action_dispatch/http/mime_types.rb**. This is where you’ll find a listing of all the pre-packaged content types in Rails, which can handle this for us: png, jpg, gif, css, html, text, js, bmp, tiff, ics, csv, mpeg, xml, yaml, atom, json, zip and pdf. You may verify this by checking out the [Github repository for Rails](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch/http/mime_type.rb). 

Take a moment to go over that repository’s code, and you’ll see that the PDF format is defined with its specific content type. When the user (in this case, us) made a request in the browser to see /home.pdf URL in localhost, Rails retrieved the PDf format from the URL and then verified it with the format.pdf code block that was defined with the index method (or action) in the HomeController. It then proceeded to set up the correct content type before invoking the block that called the render() method. 

**The Rails rendering stack** 

The key word here to understanding the Rails rendering stack is the word render itself. Whenever you see the word render, you should associate it with everything pertaining to what a user sees when making a browser request; everything from interactive forms to autoresponders to views. 

Rails also relies on its rendering stack for its ability to extend ActionController and ActionMailer within the Rails framework, (more on that later in this series). Both ActionController and ActionMailer share several common features and, in accordance with the DRY principle, the underlying code governing those shared features was abstracted out to the AbstractController class; a convenient public superclass which ActionController and ActionMailer both use and inherit from. 

AbstractController lets you pick and choose your preferred functionalities. One of its included modules is ActionView. One of the instances of ActionView::Base is view_context. It’s an instance of a View class and, according to the API documentation, that View class must have the following methods: View.new[lookup_context, assigns, controller]. It’s the context in which templates in your project are evaluated. When instantiated, it receives the view_assigns() method as one of its parameters, which should return a hash. To break it down a little better, think of it this way: The reason the link_to method works in a template is because it’s a method that’s available inside ActionView::Base. 

As for assigns, it references all of the instance variables in the controller that will be accessible in the view. This is why whenever you declare an instance variable in a controller, like @products = Product.all (where @product is designated as an assign), it will also be available in your views. Now, if you don’t want a controller sending any assigns to the view, you simply override the view_assigns() method to return an empty hash. In returning an empty hash, you will not have any of the controller methods (also known as actions) passing to the view. 

The set up of the Rails rendering stack allows you to hook into the rendering process and customize it with your own features. This is an awesome amount of power to have at your disposal. Use that power wisely and always be mindful that you’re not exposing yourself or, worse, exposing your clients or your employer to potential security holes. The first question you should ask when architecting your app is, _How much do I really need to extend the rendering stack?_ 

**Takeaway** 

Now that you're finished with Part 1 of our Rails framework series,  you should have a handle on how to create your own custom PDF renderer plugin. Take some time to familiarize yourself with the process and stay tuned for the next Rails tutorial.
