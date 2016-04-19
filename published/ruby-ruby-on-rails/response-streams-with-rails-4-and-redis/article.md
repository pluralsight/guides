If you're on the fence about updating an older application to use Rails 4, the addition of `ActionController::Live` might be helpful in making your decision a little easier. It enables keeping a connection open to your server, which can then respond with partial updates with ease. This bridges one of the bigger gaps that causes people to choose [node.js](http://nodejs.org/) over Rails for projects.

## A Basic Redis Connection

Aaron Patterson wrote a great post about [Live Streaming in Rails](http://tenderlovemaking.com/2012/07/30/is-it-live.html), over a year ago, but the interface is mostly the same today. That post is still a good starting point for `ActionController::Live`.

I first ran into the subject when working on Code School's [Rails 4: Zombie Outlaws](http://rails4.codeschool.com/) course. The last level is all about streaming, with a mention towards the end about using streaming with Redis. If you connected to this endpoint in a browser, the page would load forever and occasionally send back responses to the browser

```ruby
class ActivitiesController < ApplicationController
  include ActionController::Live

  def index
    response.headers["Content-Type"] = "text/event-stream"
    redis = Redis.new

    redis.psubscribe("user-#{current_user.id}:*") do |on|
      on.pmessage do |subscription, event, data|
        response.stream.write "data: #{data}

"
      end
    end
  rescue IOError
    # Client disconnected
  ensure
    redis.quit
    response.stream.close
  end
end
```

Here's a quick recap of what's going on:

* We're using [Puma](https://github.com/puma/puma) which allows for concurrent connections to the server.
* We create a new connection to Redis. This is important because when we call `psubscribe`, that connection is locked, and can't do anything else.
* Use `psubscribe` to subscribe to all messages for this user by using an expression. Elsewhere in the application, we are `publish`ing messages to this same channel.
* When a message is received, it's passed down to the client. In this case we're passing down JSON.
* `ensure` that the `redis` connection is `quit` and the response is ended.

## The Problem

If you wrote the above code and opened up that action in a browser, it would actually work fine -- until you tried to load the page again. At that point there would be two connections open from the servers standpoint, but only one active. This is due to the fact that the server doesn't know that the client disconnected.

That `IOError` error isn't triggered when the client disconnects as you might expect, but instead when the server attempts to write to the `response.stream` only to find that it is no longer active. Turns out this is a [well discussed problem](https://github.com/rails/rails/issues/10989) That leaves us with a few options on how to test if the client has disconnected:

* Have the server connection timeout every minute or so. (If you're on Heroku, my guess is this will automatically happen)
* Ping the client every few seconds to see if they are still there.


## A Working Solution

I ran into a [StackOverflow](http://stackoverflow.com/questions/14268690/actioncontrollerlive-is-it-possible-to-check-if-connection-is-still-alive) post on this exact topic, which led to _a_ working solution for this. This solution follows the "ping" method.

```ruby
class ActivitiesController < ApplicationController
  include ActionController::Live

  def index
    response.headers["Content-Type"] = "text/event-stream"
    redis = Redis.new

    ticker = Thread.new { loop { sse.write 0; sleep 5 } }
    sender = Thread.new do  
      redis.psubscribe("user-#{current_user.id}:*") do |on|
        on.pmessage do |subscription, event, data|
          response.stream.write "data: #{data}

"
        end
      end
    end
    ticker.join
    sender.join
  rescue IOError
    # Client disconnected
  ensure
    Thread.kill(ticker) if ticker
    Thread.kill(sender) if sender
    redis.quit
    response.stream.close
  end
end
```

This solution is based on the idea that the server will know the client has disconnected when it attempts to write to it only to find it's now there. In this case we open up two threads -- one that does our Redis subscription, and another that handles making sure the client is still there.

If you know of a better way of doing this, I'd love to hear it. Please post it in the comment section below. Short of using Sinatra, Goliath or another middleware this is the only way I've found to handle this.

## Closing the Database

One downside of keeping the connection open is that if you're using ActiveRecord, that connection will not be released until the request is complete. During the Redis subscribe phase, if you don't need to keep that connection open, you can return the current connection to the `connection_pool`.

```ruby
ActiveRecord::Base.connection_pool.release_connection
```

If you set this up to run in a `before` filter, and do any database communication before that, you shouldn't run into database connection limits.

## Update

For an example of how this technique is used, read the post on [Teaching iOS 7 at Code School](http://adamfortuna.com/articles/teaching-ios7-at-codeschool). This post details the user experience that can be achieved using response streams. Tutorial was [originally posted here](http://adamfortuna.com/articles/response-streams-with-rails-4).
