In response to a recent client request for a richer browser side UI experience, I took a fresh look at all the recent advances in JavaScript rich client frameworks. The [React library](http://facebook.github.io/react/) stood out as unique, innovative, and impressive.

The main reasons that I like React are:

1. It’s a better abstraction than MVC!
2. React keeps track of what needs to change in the DOM with its virtual DOM model.
3. All the view rendering code can assume that nothing changes during the rendering process as components recursively call `render()`. This makes reasoning about the rendering code much simpler.
4. The simpler conceptual model of always rendering the entire UI from a given state is akin to the server side rendering of HTML pages, that Rails programmers are more familiar with.
5. The documentation is very good, and it’s got significant traction.

Given that React is just about the **View** part of the client UI, or more specifically, view **components**, it seems especially suitable for integration into the Rails ecosystem to help build better rich JavaScript UIs. The [React website](http://facebook.github.io/react/) contains a [simple tutorial](http://facebook.github.io/react/docs/tutorial.html) utilizing Node for the backend. Suppose you want to use Rails for the backend?

This following instructions walk you through the steps to build the original simple tutorial with a Rails 4.2 backend utilizing the [react-rails gem](https://github.com/reactjs/react-rails). With the Rails scaffold generator, very little Rails coding is required. You can try the end result of the completed tutorial [on Heroku](http://react-rails-tutorial.herokuapp.com/), and the code [on Github](https://github.com/justin808/react-rails-tutorial).

Since the original React [tutorial](http://facebook.github.io/react/docs/tutorial.html) is excellent, I will not be rehashing any of it’s explanations of how React works. This tutorial purely focusing on converting that tutorial to utilize Rails.

Besides carefully studying the original tutorial, I recommend:

1. Watching these 2 videos for an introduction to React’s virtual DOM model. a. This video explains [design philosophy of React and why MVC is not the right model for building UIs](https://www.youtube.com/watch?v=x7cQ3mrcKaY). b. This video [compares ReactJs vs. Key Value Observation(EmberJs) and Dirty Checking (AngularJS)](https://www.youtube.com/watch?v=-DX3vJiqxm4).
2. Play with the examples on the [React overview page](http://facebook.github.io/react/). Don’t just read the examples. You can play with the code right on that page!
3. Read the [docs](http://facebook.github.io/react/docs/getting-started.html), which I found fairly interesting.


## Useful React Links


1. [Completed React-Rails tutorial Live on Heroku](http://react-rails-tutorial.herokuapp.com/): Tutorial Live on Heroku.
2. [Rails 4.2, React, completed tutorial](https://github.com/justin808/react-rails-tutorial): Github repo for completed tutorial.
3. [React: A Javascript Library For Building User Interfaces](http://facebook.github.io/react/): Main website for React.
4. [React Tutorial](http://facebook.github.io/react/docs/tutorial.html): The Node basis for this tutorial.
5. [reactjs/react-tutorial](https://github.com/reactjs/react-tutorial): Github repo for official Node based tutorial.


## Tutorial Step by Step


### Create a brand new Rails 4.2 Application

1. Install Ruby 2.1.2 or whichever recent Ruby you prefer. I use [rvm](http://rvm.io/rvm/install).
2. Install Rails gem `gem install rails --pre` NOTE: There is a bug if you RubyGems versions newer than 2.2.2\. This detailed in this [question on Stack Overflow](http://stackoverflow.com/a/25438597/1009332).
3. Create the Rails app `rails new react-rails-tutorial`
4. `cd react-rails-tutorial`
5. Create `.ruby-version` and `.ruby-gemset` per your preferences inside `react-rails-tutorial` directory.
6. Run bundler `bundle install`
7. Create new git repository `git init .`
8. Add and commit all files: `git add . && git commit -m "rails new react-rails-tutorial"`

### Create Base Rails App Scaffolding for Comment model

1. Run generator. Be sure to use the exact names below to match the React tutorial. `rails generate scaffold Comment author:string text:text`
2. Migrate the database `rake db:migrate`
3. Commit `git add . && git commit -m "Ran rails generate scaffold Comment author:string text:text and rake db:migrate"`

### Create Page for App


1. Run the controller generator `rails generate controller Pages index`
2. Fix your `config/routes.rb` to go to the home page, by changing `get 'pages/index'` to `root 'pages#index'`


### Try Out the New Rails App

1. Start the server `rails server`
2. Open your browser to [http://0.0.0.0:3000](http://0.0.0.0:3000/) and see the your blank home page.![brand-new-root-page](assets/brand-new-root-page-790x275.jpg)
3. Open your browser to [http://0.0.0.0:3000/comments](http://0.0.0.0:3000/comments) and see the comments display. ![rails-comments-index-action](assets/rails-comments-index-action-790x385.jpg)
4. Add a comment. Click around. Neat! ![rails-comments-new-action](assets/rails-comments-new-action-790x512.jpg) ![rails-comments-show-action](assets/rails-comments-show-action-790x297.jpg)
5. Test out the json API, automatically created by Rails: `curl 0.0.0.0:3000/comments.json ` and see `[{"id":1,"author":"Justin","text":"My first comment.","url":"http://0.0.0.0:3000/comments/1.json"}]%`
6. View your routes
``` bash
> rake routes
             Prefix Verb URI Pattern Controller#Action
               root GET / pages#index
           comments GET /comments(.:format) comments#index
                    POST /comments(.:format) comments#create
        new_comment GET /comments/new(.:format) comments#new
       edit_comment GET /comments/:id/edit(.:format) comments#edit
            comment GET /comments/:id(.:format) comments#show
                    PATCH /comments/:id(.:format) comments#update
                    PUT /comments/:id(.:format) comments#update
                    DELETE /comments/:id(.:format) comments#destroy

```
7. If all that worked, then commit your changes `git add . && git commit -m "Ran rails generate scaffold Comment author:string text:text and rake db:migrate"`

### React Tutorial Using Node

This is what we’ll be converting to Rails 4.2.

1. Create a new branch, in case we want to test the same design with AngularJS or EmberJS: `git checkout -b "react"`
2. Take a look at the [React Tutorial](http://facebook.github.io/react/docs/tutorial.html) and the github repo: [reactjs/react-tutorial](https://github.com/reactjs/react-tutorial).
3. Open up a new shell window. Pick a directory and then do `git clone git@github.com:reactjs/react-tutorial.git`
4. cd to the `react-tutorial.git` directory and open up the source code.
5. Optionally run the tutorial example per the instructions on the `README.md`

### Adding React to Rails

1. We’ll be using the [reactjs/react-rails gem](https://github.com/reactjs/react-rails). Plus we’ll need to include the `showdown` markdown parser, using the [showdown-rails gem](https://github.com/joshmcarthur/showdown-rails). Add these lines to your Gemfile and run `bundle`
``` ruby
gem 'react-rails', github: 'reactjs/react-rails', branch: 'master'
    gem 'showdown-rails'
```
Note, I’m using the tip of react-rails. Depending on when you try this tutorial, you may not wish to be using the tip, and don’t do that for a production application!
2. Per the gem instructions, let’s add the js assets below the `turbolinks` reference in `app/assets/javascripts/application.js`
``` javascript
//= require jquery
    //= require jquery_ujs
    //= require turbolinks
    //= require showdown
    //= require react
    //= require_tree .
```
3. Once you verify that you can load `0.0.0.0:3000` in your browser, then commit the files to git:
``` bash
git commit -am "Added react-rails and showdown-rails gems"
```

### Move Tutorial Parts to Rails Application

Now the fun starts. Let’s take the parts out of the node tutorial and put them into the Rails app.

1. Copy the necessary line from `react-tutorial/index.html` to replace the contents of `app/views/pages/index.html.erb`. You’ll just have one line there:
``` html
<div id="content"></div>
```
2. Now, the meat of the tutorial, the JS code. Copy the entire contents of `react-tutorial/scripts/example.js` into `app/assets/javascripts/comments.js.jsx` (Renamed from comments.js.coffee).
3. Commit the added files, so we can see what we change from the original versions.
``` bash
git commit -am "index.html.erb and comments.js.jsx added"
```
4. Start the rails server (`rails s`). Visit `0.0.0.0:3000`. Nothing shows up!

### Tweak the Tutorial

In the example, the call to load `example.js` comes after the declaration of the DOM element with id “content”. So let’s run the `renderComponent` after the DOM loads. Wrap the `React.renderComponent` call at the bottom of `comments.js.jsx` like this:

``` javascript
$(function() {
  React.renderComponent(
    <CommentBox url="comments.json" pollInterval={2000} />,
    document.getElementById('content')
  );
})
```

Let’s commit that diff:

``` bash
git commit -am “React component loads”
```

Then copy the css from `react-tutorial/css/base.css` over to `app/assets/stylesheets/comments.css.scsss`

The styling in is not quite right. ![copying-tutorial-styling](assets/copying-tutorial-styling-790x282.jpg)

### Add bootstrap-sass Gem

1. Add the gems

``` ruby
gem 'bootstrap-sass'
    gem 'autoprefixer-rails'
```
2. Run `bundle install`

3. Rename `app/assets/stylesheets/application.css` to `application.css.scss` and change it to the following:

``` ruby
@import "bootstrap-sprockets";
    @import "bootstrap";
```

4. Optionally, add this line to `app/assets/javascripts/application.js`

``` ruby
//= require bootstrap-sprockets
```

5. Restart the application. Notice that there is no padding to the left edge of the browser window. That’s an easy fix. Let’s put the content div inside a container, by changing `app/views/pages/index.html.erb` to this:

``` xml
<div class="container">
      <div id="content"></div>
    </div>

```

6. Let’s spruce up the data entry part. Take a look at the [Boostrap docs for CSS Forms](http://getbootstrap.com/css/#forms). You’ll have to refer to the diffs on github for this change. Or you can take creative license here! ![with-bootstrap-sass](assets/with-bootstrap-sass-790x596.jpg)


### Adding Records Fails

The first issue is that we’re not submitting the JSON correctly to add new records.

```
Started POST "/comments.json" for 127.0.0.1 at 2014-08-22 21:48:55 -1000
Processing by CommentsController#create as JSON
  Parameters: {"author"=>"JG", "text"=>"Another **comment**"}
Completed 400 Bad Request in 1ms

ActionController::ParameterMissing (param is missing or the value is empty: comment):
  app/controllers/comments_controller.rb:72:in `comment_params'
  app/controllers/comments_controller.rb:27:in `create'
```

If you look at this method in `comments_controller.rb`, you can see the issue:

``` ruby
def comment_params
  params.require(:comment).permit(:author, :text)
end
```

The fix to this is to wrap the params in “comment”, by changing this line in `comments.jsx.js`, in function `handleCommentSubmit`.

``` ruby
data: comment,
```

to


``` ruby
data: { comment: comment },
```

Here’s a enlarged view of that diff from RubyMine. ![wrap-comment-json](assets/wrap-comment-json-790x202.jpg)

After that change, we can observe this in the console when adding a new record:

```
Started POST "/comments.json" for 127.0.0.1 at 2014-08-22 21:55:18 -1000
Processing by CommentsController#create as JSON
  Parameters: {"comment"=>{"author"=>"JG", "text"=>"Another **comment**"}}
   (0.1ms) begin transaction
  SQL (0.7ms) INSERT INTO "comments" ("author", "created_at", "text", "updated_at") VALUES (?, ?, ?, ?) [["author", "JG"], ["created_at", "2014-08-23 07:55:18.234473"], ["text", "Another **comment**"], ["updated_at", "2014-08-23 07:55:18.234473"]]
   (3.0ms) commit transaction
  Rendered comments/show.json.jbuilder (0.7ms)
Completed 201 Created in 22ms (Views: 5.0ms | ActiveRecord: 3.9ms)
```

### When Visiting Other Pages in the App

If you go to the url `0.0.0.0:3000/comments` and look at browser console, you’ll see an error due the page load script looking for a component of id `content` that doesn’t exist. Let’s fix that by checking that the DIV with id `content` exists before calling `React.renderComponent`.


``` javascript
$(function() {
  var $content = $("#content");
  if ($content.length > 0) {
    React.renderComponent(
      <CommentBox url="comments.json" pollInterval={2000} />,
      document.getElementById('content')
    );
  }
})
```

### Deploying to Heroku

It’s necessary to make a couple changes to the Gemfile. Use `pg` in production and add the `rails_12factor` gem.

``` ruby
gem 'sqlite3', group: :development
gem 'pg', group: :production

gem 'rails_12factor'
```

### Turbolinks

If you’re going to have other pages in the application, it’s necessary to change when `React.renderComponent` is called, switching from document “ready” event to to the document “page:change” event. You can find more details at the [Turbolinks Gem repo](https://github.com/rails/turbolinks).


``` javascript
$(document).on("page:change", function() {
  var $content = $("#content");
  if ($content.length > 0) {
    React.renderComponent(
      <CommentBox url="comments.json" pollInterval={2000} />,
      document.getElementById('content')
    );
  }
})
```

