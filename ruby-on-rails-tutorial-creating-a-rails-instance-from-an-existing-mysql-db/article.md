In this Ruby on Rails tutorial, I will show how to create a Rails instance from an existing MySQL database.

I have an existing site that used Codeigniter and a MySQL DB, and I want to more or less convert that site into a Ruby on Rails site. I would like to make things as automatic as possible so I don't have to manually port any of the existing data or create the model properties.

I found [this thread on ruby-forum.com](http://www.ruby-forum.com/topic/134848 "Generate a migration from an existing MySQL Database (like Symfony)") that explains the basics of how to migrate an existing MySQL DB so it works with Rails. Let's follow the steps provided:

** 1. Write config/database.yml to reference your database. **

I'm going to follow [the guidelines in the RailsGuides for configuring a MySQL DB](http://guides.rubyonrails.org/getting_started.html#configuring-a-mysql-database "Configuring a MySQL Database"). The guidelines say that I should change my config/database.yml file to look something like this:


``` ruby
development:
  adapter: mysql2
  encoding: utf8
  database: blog_development
  pool: 5
  username: root
  password:
  socket: /tmp/mysql.sock
```


With some small tweaks to match my local environment, I'm ready for step 2:

** 2. Run "rake db:schema:dump" to generate db/schema.rb **

First stumbling block. I get the following error upon trying "rake db:schema:dump":

> _Please install the mysql2 adapter: `gem install activerecord-mysql2-adapter` (mysql2 is not part of the bundle. Add it to Gemfile.)_

After following the suggestions I've been given (1\. gem install activerecord-mysql2-adapter and 2\. Adding "gem 'mysql2'" to my Gemfile) I try "rake db:schema:dump" again. Second stumbling block. I get this error:

> _Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)_

 [This StackOverflow thread](http://stackoverflow.com/questions/5499035/ruby-on-rails-3-cant-connect-to-local-mysql-server-through-socket-tmp-mysql-s "Ruby on Rails 3 Can't connect to local MySQL server through socket '/tmp/mysql.sock' on OSX") tells me that I need to figure out the correct socket file and specify it in my database.yml file. I run 'mysqladmin variables | grep socket' from the command line and find out that my socket file is located at '/Applications/XAMPP/xamppfiles/var/mysql/mysql.sock' because, of course, I'm running XAMPP for my MySQL server.

Success! Well, I think so anyway. At least I didn't get any error messages this time.

** 3. Convert _schema.rb_ into db/migrate/001_create_database.rb **

My schema.rb file reflects a successful connection to my MySQL DB. It seemingly faithfully describes the tables and fields of my DB. So I create a '/db/migrate' folder, copy my 'schema.rb' file into it, rename it '001_create_database.rb' and wrap the schema as suggested by the ruby-forums.com thread:


``` ruby
class CreateDatabase < ActiveRecord::Migration
  def self.up
    # insert schema.rb here
  end

  def self.down
    # drop all the tables if you really need
    # to support migration back to version 0
  end
end
```

One note about this: The instructions in the ruby-forums.com thread say the first line of the migration file should be:


``` ruby
class CreateMigration < ActiveRecord::Migration
```
But it didn't work for me until I changed it to:

``` ruby
class CreateDatabase < ActiveRecord::Migration
```

(Note the updated class name.) All good now.

So now I think I have what I need to create the DB if I need to.

Now for the cool part. If I create a simple model whose class name matches one of the tables in my DB, I can immediately start accessing the data in the DB. Simple model:


``` ruby
class Tweet < ActiveRecord::Base
  # I have a table called 'tweets' in the DB
end
```

Now in the Rails console I can type:

``` ruby
> i = Tweet.find(296) # 296 is an ID of an item in the DB
```

And I get the data related to that record!

One last thing to really show that a connection has actually been made to the old DB. I want to display some records in the browser. There are really only three steps to accomplish this, but I'll add a fourth step to allow for pagination between all the records.

 **Step 1: Add routes to set up basic CRUD functions**

This is pretty simple, just add the following line to the beginning of the routes.rb file:


``` ruby
resources :tweets # replace 'tweet' with the name of your model
```


Now if I go to http://0.0.0.0:3000/tweets I get an error "uninitialized constant TweetsController." That's because, well, I haven't created my Tweets controller.

 **Step 2: Add a controller and define an index function**

In the '/app/controllers' directory, I simply created a file called 'tweets_controller.rb' and added the following text (replacing the names of the classes and models as necessary):


``` ruby
class TweetsController

end
```


This won't work until we get through step 3.

 **Step 3: Add the 'will_paginate' gem**

In the Gemfile, add the following line:

``` ruby
gem 'will_paginate', '3.0.3'
```

Then run 'bundle install'

 **Step 4: Create a view**

In the '/app/views' directory I created a 'tweets' directory and within that directory I created a file called 'index.html.erb'. The 'index' part of that file name matches the index function of my TweetsController for a reason - this view gets called automatically when the user goes to '/tweets/index.html'.

In the same '/app/views/tweets/' directory I create a very simple _partial _file that will be called whenever an individual tweet needs to be displayed. That file is called "_tweet.html.erb" and the contents look like this:

The contents of the '/app/views/tweets/index.html.erb' file I mentioned above look like this:


``` ruby
 <!-- Knows to repeatedly render the _tweets partial -->
```

And that's it! I get all my DB records listed, 10 at a time, with pagination links so I can jump around between them. There's no formatting and a lot more work will go into displaying and organizing the data, but this is a really good start.

I create these rough tutorials as I go, so I fear some of the info may be disjointed or inconsistent. Please let me know if you have any questions or need anything clarified. I'm happy to help.

