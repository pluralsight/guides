When building real-time applications, we often want to log or identify when a process or event occurs. For example, in an instant messaging application, we want to know if and when our message was delivered to the intended client. We see this in WhatsApp where messages are sent in realtime and you see status of each message when it is delivered and read, with double grey tick when delivered and double blue tick when the message has been read by another user. We can easily build a message delivery status using Pusher and JavaScript.

## Introducing Pusher and the app idea

[Pusher](https://pusher.com/) is built on channels and events. We can send a message to a client through a channel and have that client notify us of a read receipt by triggering an event to which the sender will respond.

**Channels** provide a way of filtering data and controlling access to different streams of information, while **events** are the primary method of packaging messages in the Pusher system which forms the basis of all communication.

To implement a message delivery status with Pusher, we'll have to subscribe to a channel and listen for events on the channel. We'll build a simple chat application in JavaScript that will send out messages to a client and the client will trigger an event when received.

## Application setup

To use the Pusher API, we have to sign up and create a Pusher app from the dashboard. We can create as many applications as we want and each one will get an application ID and secret key that enables the developer to initialize a Pusher instance in client- or server- side code.

### Create a new Pusher account

1. [Sign Up](https://pusher.com/signup) to Pusher, or login if you already have an account.
2. After signup we get to the dashboard and shown a screen to setup up a new pusher app.
     1. Enter a name for the application. In this case I'll call it "chat".
     2. Select a cluster
     3. Select the option "Create app for multiple environments" if you want to have different instances for development, staging and production
     4. Choose a front-end tech. I'll choose VanillaJS since I won't be using any framework
     5. Select NodeJS as my back-end
 
3. Click **Create App** to create the Pusher app.
 
<img src="http://blog.pusher.com/wp-content/uploads/2017/03/how-to-build-a-message-delivery-status-in-javascript-create-pusher-app.png" alt="" width="2620" height="1456" class="alignnone size-full wp-image-2829" />

## Code

We will use channels as a means to send messages and trigger events through the channel. There are 3 types of [channels](https://pusher.com/docs/client_api_guide/client_channels) in Pusher:

- **Public Channel** &mdash; a channel that can be subscribed to by anyone who knows the name of the channel.
- **Private Channel** &mdash; a channel that lets your server control access to the data you are broadcasting.
- **Presence Channel** &mdash; an extension of the private channel, but forces channel subscribers to register user information when subscribing. It also enables users to know who is online.
 
To use the private and presence channel, clients must first be authenticated. For the sample app, we'll build the client using Vanilla JS and the server (for authentication) using Node.js. Because I want messages to from client to client rather than through the server, and because I don't need to know if the user is online or away, I'll use a **private channel** for this demonstration. However, the same technique will apply using any channel type. Our Node back-end will handle channel authentication.

> Client events can only be triggered in private or presence channels, and to use any of these channel types, the user/client must be authenticated.

Also, to use client events, they must be enabled for the application. Go to your Pusher dashboard and on the **App Settings** tab, select "Enable Client Event" and update.

### Back-End

Since we're building our backend in Node using Express, let's initialize a new node app and install the needed dependencies. Run the following command:

- `npm init` and select the default options
- `npm i --save body-parser express pusher` to install express and the Pusher node package
 
Add a new file called `server.js` which will contain logic to authenticate the Pusher client and also render the static files we'll be adding later. This file will contain the content below

``` language-javascript
var express = require('express');
var bodyParser = require('body-parser');

var Pusher = require('pusher');

var app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));

var pusher = new Pusher({ appId: APP_ID, key: APP_KEY, secret:  APP_SECRET, cluster: APP_Cluster });

app.get('/',function(req,res){      
     res.sendFile('index.html', {root: __dirname });
});

app.use(express.static(__dirname + '/'));

app.post('/pusher/auth', function(req, res) {
  var socketId = req.body.socket_id;
  var channel = req.body.channel_name;
  var auth = pusher.authenticate(socketId, channel);
  res.send(auth);
});

var port = process.env.PORT || 5000;
app.listen(port, function () {
  console.log(`Example app listening on port ${port}!`)
});
```
We instantiate Pusher by passing in an object that contains the details of our app ID and secret key, which can be found in the Pusher dashboard, on the **App Keys** tab. The line `var auth = pusher.authenticate(socketId, channel);` authenticates the client with Pusher and returns an authentication code to the calling client. To allow this file to run when we start npm, we update package.json with the following value:

``` language-json
"scripts": {
    "start": "node server.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```
### Front-end

With the back-end in place, we now move on to crafting the front-end. We'll be using the template from this [site](http://bootsnipp.com/snippets/6eWd) with a slight modification.

Add a new file named `index.html` and `style.css` with the following content in each file:

**Index.html**

``` language-html
<!DOCTYPE html>
<html>
<head>

    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">


    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">

    <scriptsrc="https://code.jquery.com/jquery-2.2.4.min.js"
        integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="
        crossorigin="anonymous"></script>


    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>

    <link rel="stylesheet" href="style.css">
    <script src="https://js.pusher.com/4.0/pusher.min.js"></script>
    <script src="index.js"></script>
</head>
<body>
    <div class="container">
    <div class="row form-group">
        <div class="col-xs-12 col-md-offset-2 col-md-8 col-lg-8 col-lg-offset-2">
            <div class="panel panel-primary">
                <div class="panel-heading">
                    <span class="glyphicon glyphicon-comment"></span> Comments
                    <div class="btn-group pull-right">
                        <button type="button" class="btn btn-default btn-xs dropdown-toggle" data-toggle="dropdown">
                            <span class="glyphicon glyphicon-chevron-down"></span>
                        </button>
                        <ul class="dropdown-menu slidedown">
                            <li><a href="http://www.jquery2dotnet.com"><span class="glyphicon glyphicon-refresh">
                            </span>Refresh</a></li>
                            <li><a href="http://www.jquery2dotnet.com"><span class="glyphicon glyphicon-ok-sign">
                            </span>Available</a></li>
                            <li><a href="http://www.jquery2dotnet.com"><span class="glyphicon glyphicon-remove">
                            </span>Busy</a></li>
                            <li><a href="http://www.jquery2dotnet.com"><span class="glyphicon glyphicon-time"></span>
                                Away</a></li>
                            <li class="divider"></li>
                            <li><a href="http://www.jquery2dotnet.com"><span class="glyphicon glyphicon-off"></span>
                                Sign Out</a></li>
                        </ul>
                    </div>
                </div>
                <div class="panel-body body-panel">
                    <ul class="chat">

                    </ul>
                </div>
                <div class="panel-footer clearfix">
                    <textarea id="message" class="form-control" rows="3"></textarea>
                    <span class="col-lg-6 col-lg-offset-3 col-md-6 col-md-offset-3 col-xs-12" style="margin-top: 10px">
                        <button class="btn btn-warning btn-lg btn-block" id="btn-chat">Send</button>
                    </span>
                </div>
            </div>
        </div>
    </div>
</div>

<script id="new-message-other" type="text/template">
    <li class="left clearfix">
        <span class="chat-img pull-left">
            <img src="http://placehold.it/50/55C1E7/fff&text=U" alt="User Avatar" class="img-circle" />
        </span>
        <div class="chat-body clearfix">
            <p>
                {{body}}
            </p>
        </div>
    </li>
</script>

<script id="new-message-me" type="text/template">
    <li id="{{id}}" class="right clearfix">
        <span class="chat-img pull-right">
            <img src="http://placehold.it/50/FA6F57/fff&text=ME" alt="User Avatar" class="img-circle" />
        </span>
        <div class="chat-body clearfix">
            <div class="header">
                <small class="text-muted">{{status}}</small>

            </div>
            <p>
                {{body}}
            </p>
        </div>
    </li>
</script>

</body>
</html>
```
**style.css**

``` language-css
@import url("http://netdna.bootstrapcdn.com/font-awesome/4.0.3/css/font-awesome.css");
.chat
{
    list-style: none;
    margin: 0;
    padding: 0;
}

.chat li
{
    margin-bottom: 10px;
    padding-bottom: 5px;
    border-bottom: 1px dotted #B3A9A9;
}

.chat li.left .chat-body
{
    margin-left: 60px;
}

.chat li.right .chat-body
{
    margin-right: 60px;
}


.chat li .chat-body p
{
    margin: 0;
    color: #777777;
}

.panel .slidedown .glyphicon, .chat .glyphicon
{
    margin-right: 5px;
}

.body-panel
{
    overflow-y: scroll;
    height: 250px;
}

::-webkit-scrollbar-track
{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    background-color: #F5F5F5;
}

::-webkit-scrollbar
{
    width: 12px;
    background-color: #F5F5F5;
}

::-webkit-scrollbar-thumb
{
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,.3);
    background-color: #555;
}
```
The page we added holds a 1-to-1 chat template. At line **18** we added a script to load the Pusher JavaScript library, and at **19** we're loading a custom JavaScript file which we will use to handle interactions from the page. Add this file with the following content:

**index.js**

``` language-javascript
$(document).ready(function(){
    // Enable pusher logging - don't include this in production
    Pusher.logToConsole = true;

    var pusher = new Pusher('APP_KEY', {
        cluster: 'eu',
        encrypted: false
    });

    var channel = pusher.subscribe('private-channel');
    //channel name prefixed with 'private' because it'll be a private channel
});
```
From the code above we first connect to Pusher by creating a Pusher object with the **App_Key** and cluster. These values are gotten from the Pusher dashboard. `encrypted` is set to false to allow it to send information on an unencrypted connection.

Afterwards, we subscribe to a channel which is to be used for sending out messages. Channel names can be anything, but must be a maximum of 164 characters. Another restriction on a private channel is that it must be prefixed with `private-`.

Next we bind to events. This way we can receive messages from a client through the channel we subscribed to. Add the following line to `index.js`

``` language-javascript
channel.bind('client-message-added', onMessageAdded);
channel.bind('client-message-delivered', onMessageDelivered);

$('#btn-chat').click(function(){
    const id = generateId();
    const message = $("#message").val();
    $("#message").val("");

    let template = $("#new-message-me").html();
    template = template.replace("{{id}}", id);
    template = template.replace("{{body}}", message);
    template = template.replace("{{status}}", "");

    $(".chat").append(template);

    //send message
    channel.trigger("client-message-added", { id, message });
});
function generateId() {
    return Math.round(new Date().getTime() + (Math.random() * 100));
}

function onMessageAdded(data) {
    let template = $("#new-message-other").html();
    template = template.replace("{{body}}", data.message);

    $(".chat").append(template);

    //notify sender
    channel.trigger("client-message-delivered", { id: data.id });
}

function onMessageDelivered(data) {
    $("#" + data.id).find("small").html("Delivered");
}
```

Remember, I will be triggering events from the client and don't want these to go through the back-end or be validated. This is just for this demo. [Client events](https://pusher.com/docs/client_api_guide/client_events#trigger-events) must be prefixed by `client-`, as can be seen in the code above. Events with any other prefix will be rejected by the Pusher server, as will events sent to channels to which the client is not subscribed.

> It is important that you apply additional care when when triggering client events because these originate from other users and could be tampered with by a malicious user of your site.

`client-message-added` will be triggered when a user enters a new message. Once the other user gets the message, it is displayed on the page, and `client-message-delivered` event is triggered to notify the sender of receipt. This way we achieve the objective of getting notified when messages are delivered in our application.

Run the application and see how it works!

<img src="http://blog.pusher.com/wp-content/uploads/2017/03/how-build-a-message-delivery-status-in-javascript-final-app.gif" alt="" width="1425" height="764" class="alignnone size-full wp-image-2830" />

## Wrapping Up

This guide aimed to demonstrate how to implement a message delivery status notification using Pusher, a realtime service, and JavaScript. You can find the code on [GitHub](https://github.com/pmbanugo/pusher-message-delivery-status). I hope that this exercise has helped you see other ways that Pusher can be used for messaging and other applications.

Until next time!

___

Thank you for reading this tutorial. Please leave your comments and feedback in the discussion section below. Be sure to add this guide to your favorites as well!

*This was originally published on [Pusher](https://blog.pusher.com/how-to-build-a-message-delivery-status-in-javascript/)*