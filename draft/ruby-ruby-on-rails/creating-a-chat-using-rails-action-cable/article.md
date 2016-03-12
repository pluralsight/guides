
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

As you have guessed from the name of the new application, we are going to build a very simple chat that leverages the power of Action Cable. Here is how it is going to work in a few steps:




 First, let's start by scaffolding a simple controller for the chat room. It will contain the view for the chat itself.

```bash
cd chatapp

```
```bash

rails g controller rooms show
```

 The controller and the action are done, let's put them as the root for the application:

```ruby
#config/routes.rb

Rails.application.routes.draw do

root to: 'rooms/show'

```