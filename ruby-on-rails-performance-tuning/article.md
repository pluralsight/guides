For something that is touted as being an "easy" to use framework, Rails is a pretty complex beast. Over the years it has progressed from a seemingly simple framework that "anyone" can learn to the intricate collection of add-ons, gems and extensions that make it the power house it is now.

Optimization is an overlooked aspect of Rails development that often isn't covered in the tutorials you find online.  With the sheer simplicity of snapping in a few gems, throwing together some quick Active Record queries, pushing to Heroku and forgetting about the rest, it's all too easy to find your application running slow and sluggish, before long you need to backtrack and try to figure out what exactly you did that caused the slow down.

Writing better, leaner code is often the answer, although this article isn't a coding guide, instead I'm going to list a few simple things you can do to get better performance from your application.  These might not be applicable for everyone, but merely some tricks that iv learned that have helped me squeeze a bit more performance out of some recent projects of mine.

### Use Unicorn As Your Default Rails Server

This one is a pretty common one, but something I see new Heroku users missing out on a lot.  Rails by default uses the "Webrick" server to run Rails, common practice was to often change this it use "Thin" instead.

![Unicorn Ruby on Rails](https://s3.amazonaws.com/static.written.com/angry_unicorn-150x1501415641218.png)

The problem here is that both these servers don't handle concurrent connections very well, not ideal for a production web server. There are ways to optimize this, but it's just better and easier to switch to Unicorn.

Unicorn is a multi threaded web server and allows for multiple "workers" which effectively double and triple the amount of connections you can accept at a time by forking processes.

As an example, setting 5 worker processes allows for 5 concurrent connections. In a Heroku app, every time you scale a dyno, your increasing your maximum concurrent connections by 5.

`4 x Dynos with Unicorn configured for 5 workers = 20 concurrent connections`

You'll read over the web that there's configuration involved with unicorn, but really, it's not much for a basic setup.

Example on my Heroku / Rails app, I used the default Unicorn config and followed instructions found here [Adding Concurrency to Rails Apps with Unicorn](https://blog.heroku.com/archives/2013/2/27/unicorn_rails), changing only 2 or 3 parameters, then I was up and running with Unicorn. About 10 minutes in total.  Obviously do some quick Googling to see what works and is appropriate for your app.

With regards to how many workers you can run, that will depend on how heavy your application is and how much free memory you have remaining.

Heroku have made some suggestions about tuning your database connection limit in relation to your Unicorn workers here: [Heroku Database | Calculating Required Connections](https://devcenter.heroku.com/articles/concurrency-and-database-connections#calculating-required-connections)

### Host Your Assets Externally

By now you've switched to Unicorn, and you're feeling great that you can service more concurrent requests then before, except that when we think about page loads,  one end user doesn't translate to one request.

The problem is your page is loaded with resources such as  JavaScript, CSS, images, favicon, fonts etc. When a user hits your page and retrieves all these assets, they come down as multiple requests.

Those 5 unicorn workers are starting to lose their shine when you have 30 or 40 assets coming down the line per page load.

But wait, there's more ...

All browsers will have a HTTP request limit, which effectively queues HTTP requests to prevent people from opening too many connections per request.  The amount of concurrent connections allowed per browser, per domain is typically 2 ( [The Two HTTP Connection Limit Issue](http://www.openajax.org/runtime/wiki/The_Two_HTTP_Connection_Limit_Issue) ).

When you consider a page loaded with images, CSS, JavaScript etc. It's not hard to do the math here.

So, what's the solution?

You've heard about compressing JavaScript and CSS then combining into a single file to reduce the HTTP requests per page right?  Well what if you could offload some of those HTTP requests to another domain?

By simply introducing another domain into your page request, you're offloading some of the request queue to another resource.

That browser HTTP request limit poses less of a problem and your request queue is split up.

> The idea is to keep your Rails app serving dynamic content only, and offload the static resources to something else to free up your server for more dynamic content requests.

A common choice is to use Amazon Web Services S3\. AWS S3 provides a simple online file storage facility that you can use to store all the static assets from your web application.  Simply dump your assets in an S3 bucket and make it public then swap out your links in your web app for the S3 links.

![Amazon Web Services](https://s3.amazonaws.com/static.written.com/aws-300x1291415641218.jpg)

You can begin to see some immediate results by simply looking at the rails logs, either development.log or production.log. You'll notice there's is a much lower number of requests to the server, only only those requesting dynamic content are visible.

The page should feel snappier right away and if you pull up the network panel on your Chrome developer tools you can see the requests are being split between your rails app and AWS S3.

I find this especially useful for hosting my 2X res retina ready web graphics which can double in size from regular web graphics.  It's not terribly useful for hosting tiny under 5Kb icons, but with things like diagrams, retina logos, PDF's etc, it can make a big difference.

### Automatic Remote Asset Hosting

Alright, so you want to use S3, but you don't really like the thought of having to manually move files to S3 every time you add new images to your site.  No problems, there's a gem for that called:

**Asset Sync**: [Learn More](https://github.com/rumblelabs/asset_sync)

Asset Sync works by pushing your static files to Amazon S3 automatically on asset pre-compilation when deploying your app.  This happens automatically after a push if you're using Heroku, otherwise you  just need to run


``` ruby
rake assets:precompile
```


Simply provide asset sync with the details of your S3 bucket and let it do the rest.

It does have a requirement that you have used the Rails in built image_tag helpers for the URL's to be swapped out though, so you may want to check your code for rogue tags before using this gem.

### Preload Active Record Associations

Active Record makes database querying so simple. Chain together a few methods and bam, you've saved your self lines and lines of T-SQL. The problem is this simplicity masks the underlying operation and it's very easy to not realize how inefficient your database calls are.

Take for example:

``` ruby
class User < ActiveRecord::Base
  has_many :posts
end

class Post < ActiveRecord::Base
  has_many :comments
  belongs_to :user
end

class Comment < ActiveRecord::Base
  belongs_to :post
end
```

Let's say on a users profile page we would like to show a listing of comments from this user.

``` ruby
@user = User.find(id)

@user.posts.each do |post|
   post.comments.each do |comment|
       <%= comment.message %>
   end
end
```

What you end up with is something like:

``` ruby
User Load (1.1ms) SELECT `users`.* FROM `users` WHERE 'id' = 1 LIMIT 1
 Post Load (0.7ms) SELECT `posts`.* FROM `blogs` WHERE `blogs`.`user_id` = 2
 Comment Load (0.7ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 43
 Comment Load (1.7ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 55
 Comment Load (2.2ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 32
 Comment Load (0.9ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 66
 Comment Load (2.2ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 56
 Comment Load (4.8ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 65
 Comment Load (1.8ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 68
 Comment Load (0.8ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` = 71
```

The user has commented in 9 different posts, which results in 9 separate queries to the DB. This is a small scale example, but you can see how this can compound into something nasty.

The solution is to use includes()

``` ruby
@user = User.find(id).includes(:posts => :comments)
```

When using includes, Active Record ensures that all of the specified associations are loaded using the minimum possible number of queries. The actual query executed is converted into a more optimal singular query and it is that result set that is iterated over instead of the replicated find(id) style queries.

Our query stack list now looks like

``` ruby
User Load (1.0ms) SELECT `users`.* FROM `users` WHERE 'id' = 1 LIMIT 1
Post Load (0.4ms) SELECT `posts`.* FROM `posts` WHERE `posts`.`user_id` IN (1)
Comment Load (0.5ms) SELECT `comments`.* FROM `comments` WHERE `comments`.`post_id` IN (43, 55, 32, 66, 56, 65, 68, 71)
```

Much more efficient

### Use Caching

Caching common pages or actions can give a pretty dramatic performance increase.  Even if your pages don't contain calls to the database or heavy dynamic content,  the simple fact of avoiding rendering overheads for partials or content blocks can increase performance enough to make it worth while.

#### Page Caching

When a page is cached, it's stored in the /public folder as plain old static HTML. This gets served up by the web server much faster than normal as the HTML page can be served directly without needing to go through Rails at all.

Take for example, a standard home page for a business's website.  The home page may consist of several partials, a bunch of Rails helper methods for images, links and some precompiled CSS and JS.

A normal request will filter through the router first then to the controller, further onto the action then serve up the view.

The view of course then needs to render the various partials (which may also contain more Ruby/Rails items to process) and then switch out all the Rails helper tags for regular HTML.

This all happens pretty fast, but you need to think about how the performance of all this compounds when you add more functionality.

A cached page is served up as a single HTML file without having to bounce through the Rails stack, so none of this pre-processing needs to happen.  Your web server dishes up the page as fast as it can retrieve it from the file system (which is pretty damn fast).

A cached page is also resilient to server errors as there is no back end processing required to serve it up. This is why by default, your 500 and 404 error pages are plain HTML pages in the /public folder.

Try dropping a HTML copy of your page into the /public directory and then goto  http://yourapp.com/yourpage.html and see how fast it loads.

A lot of the time, pages on your site may only update content once or twice a year, and you're essentially serving up a static page whilst taking the performance hit of a dynamic page rendering time.

Some examples of pages that could benefit from page caching:

* FAQ pages
* Terms and conditions pages
* Home page (for product items or company etc)
* Help pages
* About pages
* Product overview pages

Ideally, it's better to keep your servers resources for crunching data, making DB requests or handling dynamic content requests rather than just figuring out what to put on the page.   Page caching is a nice way to reduce time spent rendering pages.

It's quite easy to setup, simply set

``` ruby
config.action_controller.perform_caching = true
```

in your production.rb then

``` ruby
set caches_page :your_page_action
```

at the top of the controller you wish to use it on.

If you want to expire the cached page so that it will be re-cached you can use the following anywhere in your code

``` ruby
ActionController::Base::expire_page("/yourpage")
```

There are several different ways you can cache your Rails application. Some more notable ones are **Action Caching **and **Fragment Caching**.

These are more suitable when it's not feasible to cache the entire page, or when you need to manage authentication for a cached page and require the use of before and after filters.

Action Caching allows us to cache a page whilst still running through the full Rails stack. This means it triggers the before and after filters like a normal Rails request, how ever this also means we lose the performance benefit of going directly to the HTML file.

Fragment Caching allows a fragment of view logic to be wrapped in a cache block and served out of the cache store when the next request comes in. This is useful on pages with lots of different fragments of which only some you would like to cache

Read more on it here: [Caching With Rails](http://guides.rubyonrails.org/caching_with_rails.html)

### Use CloudFlare

If you have gone ahead and followed some of the advice above about hosting you're assets externally and leveraging some of the Rails caching features, then using CloudFlare is less important as it effectively duplicates some of these features.
![CloudFlare](https://s3.amazonaws.com/static.written.com/cloudflare1415641219.jpg)

CloudFlare caches and compresses the content from your website at their end and serves it to your users in the new trimmed down format, acting as a "man in the middle" between your servers and your users.

CloudFlare runs their own content delivery network around the world and can deliver the content from your website to your users in the shortest possible time by serving the assets from a location thats closest to the user.

Other options such as CSS and JSS compression are standard features, how ever for Rails users that have Sprockets enabled, this is already happening on our app by default.

CloudFlare works by taking over your DNS and routing all your website traffic through their servers whilst applying all the internal CloudFlare optimizing magic along the way, effectively acting like a proxy cache and holding static copies of your CSS and JS in cache ready for immediate delivery when a page request is made.

Best of all, it's free for a basic account. The basic account is still quite powerful and I run many of my own sites through the basic account, most of which I see visible improvements in performance.

The Pro accounts offer greater performance enhancements such as image optimization, resource preloading and a feature called SPDY that apparently alters the HTTP request to reduce latency and increase security, but it costs $20 a month.  It's also worth mentioning that if you want to use SSL, and you want to use your own certificate you will need to use a Pro account as the free accounts can only offer certificates that are signed by CloudFlare.

Something else worth mentioning is that CloudFlare offers some great security features and has the ability to block spammers and suspected malicious users before they even get to your server.   You can chose to display a CAPTCHA style authentication box to suspect users before they're allowed access.

This feature alone has been brilliant for a few forums I run, stopping around 8% of total users for suspected spam accounts.

 [Check out: CloudFlare here](https://www.cloudflare.com)

