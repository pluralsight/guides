HTTP and Websockets
--
In HTTP, the connection between the server and the client has a short lifespan: The client requests a resource from the server, a connection with a server is established and the requested resource (be it JSON, HTML, XML or some kind of a file) is streamed to the client as a response. Then, the connection is closed. But how the client would know if the server has new or updated data? Usually, HTTP would use long polling, in which the client would 'ask' the server if there is something new in a given interval of time.

 Unlike HTTP, WebSockets is protocol that enables the client and the server to keep a open connection, enabling them to directly stream data between each other. The client 'subscribes' to an open websocket connection in the server and when there is new information, the server 'broadcasts' the data and the subscribed clients receive it. In that way, both the server and the client know about the state of the data and can easily synchronize changes occur.

How does ActionCable function?
--

 Since the Rails controllers are purpose-built to handle HTTP requests, Rails has devised a different way to handle its integration of WebSockets. Rails 5 apps have a new directory in their `app` directory - `channels`. Channels act as controllers for WebSocket requests - they encapsulate the logic about particular work of unit such as chat messages or notifications. The channels can be subscribed to client-side in order to transmit data from and to the channel or multiple channels.


Installing ActionCable
--

 Action Cable is a new feature that requires you to have the latest version of Rails 5. One of the prerequisites of Rails 5 is that you have Ruby 2.2.4 and up installed alongside with its development kit. As of March 11, 2016, the latest stable version is Rails 5 beta3. In order to get it, you need to install the `rails` gem with the following options:

```bash
gem install rails --pre --no-ri --no-rdoc
```

Action Cable requires the [PostgreSQL](http://www.postgresql.org/download/) as its adapter. You need to install it on your Operating System and configure it before continuing with the tutorial.
When creating the a new Rails app, you need to specify it as an adapter so that the new app won't have the default SQLite.

```bash
rails _5.0.0.beta3_ new chatapp --database=postgresql
```
After creating your app and configuring the `database.yml` file to match your PostgreSQL settings, we are ready to move on and create an empty database:

```bash
cd chatapp
rake db:create
```

Laying the foundation
--

As you have guessed from the name of the new application, we are going to build a very simple chat that leverages the power of Action Cable. Here is how it is going to work in a few steps of pseudocode:

-   Create a server-side websocket channel called `room_channel` that will have methods for when a client subscribes and unsubscribes and a method for sending data to the client.
-   Create a client-side representation of the channel using JavaScript (CoffeeScript in this case) that will be called `RoomChannel` (note the naming convention). It will subscribe to the server-side channel and will have methods which will be fired on connecting/disconnecting from the server and methods for handling sending and receiving of data 
-   Make a [Rails Job](http://edgeguides.rubyonrails.org/active_job_basics.html) that will queue the new message to be created in the database and broadcast it back through the websocket channel after it is created.


 
First, let's start  offby scaffolding a simple controller for the chat room. It will contain the view for the chat itself.


```bash

rails g controller rooms show
```

 The controller and the action are done, let's put them as the root for the application:

```ruby

#config/routes.rb

Rails.application.routes.draw do
root to: 'rooms#show'
```
Next, we'll create the model which, as you'd probably guess, is called `message` and it will have a content attribute where the actual message will be stored.

```bash
rails g model message content:text
```
And migrate it into the database. Note that in Rails 5, you can use `rails` for rake tasks.

```bash
rails db:migrate
```

With the model ready, let's add all the messages to the  `show` action.
```ruby
#app/controllers/rooms_controller.rb

class RoomsController < ApplicationController
  def show
    @messages = Message.all
  end
end

```
The controllers and the model are done, time to move on to the views.

First, it is time to remove the scaffolded view for `rooms#show` and replace it with this:

```erb
<h1>Chat room</h1>
  
<div id="messages">
  <%= render @messages %>
</div>
  
<form>
  <label>Say something:</label><br>
  <input type="text" data-behavior="room_speaker">
</form>

```

Second, let's create a folder for messages in the views folder and create the partial that we are going to use for rendering a single message

```erb
# app/view/messages/_message.html.erb

<div class=“message”>
  <p><%= message.content %></p>
</div>
```

 `render @messages` will automatically look for a `_message.html.erb` partial in the `views/messages` directory to display each single message, so that we don't have to write anything extra - it just cannot get DRYer than that!

Creating a channel
--

 So far, it's all been the standard Rails stuff. Time to move to the interesting part - creating your first ActionCable channel. 

 The first we need to do is enable the usage of ActionCable in the application. It is done in two simple steps:
  1. Mount it in `routes.rb` .
```ruby
  #config/routes.rb
     # Serve websocket cable requests in-process
   mount ActionCable.server => '/cable'
```
  2. Initialize it in `cable.coffee`

```coffee
#= require action_cable
#= require_self
#= require_tree ./channels
#
@App ||= {}
App.cable = ActionCable.createConsumer()

```

It is also good to check if you have  `<%= action_cable_meta_tag %>` in the head section of `app/views/layouts/application.html.erb`.


Rails has introduced a new generator for generating channels, so let's run it and see what it does:

```bash
rails g channel room speak
```
 Running it will generate two files - a javascript file located in `app/assets/javascripts/channels/room.coffee` and a ruby file located in `app/channels/room_channel.rb`. 

Let's analyze the ruby file first:

```ruby
 #app/channels/room_channel.rb

class RoomChannel < ApplicationCable::Channel
  def subscribed
    # stream_from "some_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak
  end
end
```

 The `subscribed` method is a default method that is called when a client connects to the channel and it is usually used to 'subscribe' the client to listen to changes. The `speak` action is a custom action that we created when we ran the generator. It will be used to receive data from its client-side representation.

```coffee
 #app/assets/javascripts/channels/room.coffee

App.room = App.cable.subscriptions.create "RoomChannel",
  connected: ->
    # Called when the subscription is ready for use on the server

  disconnected: ->
    # Called when the subscription has been terminated by the server

  received: (data) ->
    # Called when there's incoming data on the websocket for this channel

  speak: ->
    @perform 'speak'
```

 In the JavaScript file, the client subscribes to the server through  `App.room = App.cable.subscriptions.create "RoomChannel"`.There are three default methods - `connected` , `disconnected` which, as you might have probably guessed, handle the state of the connection. The third one is `received` which is going to handle how received data from the server-side is going to be handled. The `speak` method in the JavaScript file will be used to send data to its server-side representation.

Start your rails server by typing `rails s` and go to your JavaScript console. 
![ActionCable client-side](https://raw.githubusercontent.com/pluralsight/guides/master/images/53df9c0c-b325-4feb-9b8f-97ed51ed9f62.cable)

Typing `App.cable` will show you what the actual representation of ActionCable looks like client-side. `App.room` is the channel we just created. By typing `App.room.speak` we call the `speak()` function from the `room.coffee` file, which then transmits data to the `speak` method in `room_channel`.

![ActionCable server-side](https://raw.githubusercontent.com/pluralsight/guides/master/images/02531309-4849-443d-8245-cb2ca979f7c5.cable)

In the terminal you can see that the method has been successfully called from the client. Awesome!

Trasmitting data
--

 Now that we are done with the connection, it is time to transmit and create our first message via ActionCable. First, let's make the `speak()` function accept a parameter:


```coffee
#app/assets/javascripts/channels/room.coffee

App.room = App.cable.subscriptions.create "RoomChannel",
  #rest of the code
  speak: (message) ->
    @perform 'speak', message: message
```

This will send a `message` object serialized as JSON to the server-side `speak` method in the RoomChannel. Consequently, we need to change the method so that it accepts parameters! Let's do that.

```ruby
 #app/channels/room_channel.rb

  def speak(data)
    ActionCable.server.broadcast "room_channel", message: data['message']
  end

```

 Now the speak method will take the message and broadcast it to `room_channel`. But how do we get the broadcasts from it? In order to do that, we will assign all the subscribers to listen to it by using `stream_from` in the `subscribed` method. 

```ruby
 #app/channels/room_channel.rb

class RoomChannel < ApplicationCable::Channel
  def subscribed
     stream_from "room_channel"
  end

  def unsubscribed
    # Any cleanup needed when channel is unsubscribed
  end

  def speak(data)
    ActionCable.server.broadcast "room_channel", message: data['message']
  end
end
```

Essentially, `room_channel` is the environment in the ActionCable server where the data comes and gets bounced back to all clients. We can receive the data from `subscribed` by, surprise-surprise, using the `received()` function in `room.coffee`. Here is how:

```coffee
#app/assets/javascripts/channels/room.coffee

  received: (data) ->
    alert(data['message'])

  #speak function

$(document).on 'keypress', '[data-behavior~=room_speaker]', (event) ->
  if event.keyCode is 13 # return/enter = send
    App.room.speak event.target.value
    event.target.value = ''
    event.preventDefault()
```
 An event listener has been added on the bottom of the file for the textbox we put in the template.

Restart your rails server, and try typing a message in the text field and clicking enter.
Did you get an alert? Great! If you open multiple browsers or tabs on `localhost:3000` and you type something, you will get alerts on all of them because they'll all be subscribed to the same channel. We have completed a full cycle - the message goes from the client to the server and bounces back to the server to all clients that have subscribed to the channel.

![Alert from websockets](https://raw.githubusercontent.com/pluralsight/guides/master/images/46534b4f-ab02-4481-a92a-685034817eef.PNG)

Persisting messages into the database
--

 Our application is working, but there are two issues:
1. The messages are not stored in the database
2. We don't have enough control in what format the messages are broadcasted.

Solving the first problem is easy - we are simply going to create a new message once the server receives the message instead of broadcasting it:

```
 #app/channels/room_channel.rb

  #the rest of the methods
  def speak(data)
    Message.create!
  end
```

What we are aiming to achieve is to broadcast the `_message.html.erb` partial we created earlier to the `room_channel`. Usually, we would do this in the controller, but we do not have one. Rendering it in the model, view or the channel seems inappropriate. So where can we put it? Enter [Rails Jobs](http://edgeguides.rubyonrails.org/active_job_basics.html).


Here is how we are going to do it: 
 First, we will create a `after_create` hook in the model.

```ruby
#app/models/message.rb

class Message < ApplicationRecord
  after_create_commit { MessageBroadcastJob.perform_later self }
end 

```

Once the message has been created, it will send it to  the `MessageBroadCastJob` that will handle the broadcasting.

Let's generate  `MessageBroadCastJob`:

```bash
rails g job MessageBroadcast
```

```ruby
#app/jobs/message_broadcast_job.rb
class MessageBroadcastJob < ApplicationJob
  queue_as :default

  def perform(message)
    ActionCable.server.broadcast 'room_channel', message: render_message(message)
  end

  private

  def render_message(message)
    ApplicationController.renderer.render(partial: 'messages/message', locals: { message: message })
  end
end
```

The `perform` method of the job is going to receive the message, do its magic by rendering the message into the `_message.html.erb` partial, and broadcast directly from the job. 
Another new feature in Rails 5 is that you can render templates out of the scope of a controller, using the `renderer` of the `ApplicationController` . This nifty feature is used exactly for a case like this in which we do not have a `MessageController` defined but we would like to use its templates.

There is only one thing left to do - remove the `alert` and replace it with a function that will append the partial to our view:

```coffee
#app/assets/javascripts/channels/room.coffee

  received: (data) ->
    $('#messages').append data['message']
```

 