Security is hard. When we build applications, we want to allow only registered users to access the application. We want to be able to manage user accounts, see when they last logged in, disable suspicious accounts, and have a dashboard to manage incoming data. We might also need to support multi-factor authentication and social login. 

But security isn’t just hard, it also takes a while to implement. What if there’s a service that could handle some part of the development hassle for you? Why spend weeks or months rolling your own auth?  This is where **Auth0** shines. 

This tutorial will give you an in-depth view of how to: 
* build a chat application with Pusher
* add user authentication with Auth0 Lock
* manage users from the Auth0 dashboard. 


  ## Introduction to Auth0 and Pusher

[Auth0](https://auth0.com/) is an Authentication-as-a-Service (or Identity-as-a-Service) provider focused on encapsulating user authentication and management. The service provides an SDK to allow developers to easily add authentication and manage users. Its user management dashboard  assists with breach detection, multi-factor authentication, and passwordless login. 

[Pusher's](https://pusher.com/) APIs and hosted infrastructure allow us to build scalable and reliable realtime applications. At its core, Pusher works through **channels** and **events**. Channels provide a way of filtering data and controlling access to different streams of information. Events are the primary method of packaging messages in the Pusher system and form the basis of all communication.

## Building the application

We will be building a chat application that will allow users to communicate so that everyone registered to a particular channel can see other users' messages to the channel. It’ll work similarly to how channels work in Slack: just one channel for everybody to communicate.

Here’s what we’ll be building:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502274258315_build-secure-chat-app-with-auth0-and-pusher-in-javascript.gif)


### Setting up the backend
We’ll start by building the backend which will facilitate receiving and broadcasting chat messages, serving static files, and also setting up Auth0 and Pusher. 

First, you’ll need to signup for a Pusher and Auth0 account. Go to [pusher.com](https://pusher.com/) and [auth0.com](https://auth0.com) and sign up for an account. To use Pusher API we have to signup and create a Pusher app from the dashboard. We can create as many applications as we want, and each one will get an application ID and secret key which we’ll use to initialize a Pusher instance on client or server side code. 

**Create a new Pusher account**
To create a new Pusher app, click the **Your apps** side menu, then click the **Create a new app** button below the drawer. This brings up the setup wizard.

1. Enter a name for the application. In this case I’ll call it “chat”.
2. Select a cluster.
3. Select the option “Create app for multiple environments” if you want to have different instances for development, staging and production.
4. Select **Vanilla JS** as the frontend and **NodeJS** as the backend.
5. Complete the process by clicking `Create App` button to set up your app instance.


![Create new app on Pusher](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1501241683440_Screen+Shot+2017-07-28+at+12.31.27.png)


Since we’re building our backend in Node using Express, let’s initialise a new Node app and install the needed dependencies. Run the following command:

1. **npm init** and select the default options
2. **npm i --save body-parser express pusher** to install express and the Pusher node package

Add a new file called `server.js` which will contain logic to authenticate the Pusher client and also render the static files we’ll be adding later. This file will contain the content below:

    var express = require('express');
    var bodyParser = require('body-parser');
    var Pusher = require('pusher');
    
    var app = express();
    app.use(bodyParser.json());
    app.use(bodyParser.urlencoded({ extended: false }));
    
    var pusher = new Pusher({ appId: APP_ID, key: APP_KEY, secret:  APP_SECRET, cluster: eu });
    
    app.post('/pusher/auth', function(req, res) {
      var socketId = req.body.socket_id;
      var channel = req.body.channel_name;
      var auth = pusher.authenticate(socketId, channel);
      res.send(auth);
    });
    
    app.post('/message', function(req, res) {
      var message = req.body.message;
      var name = req.body.name;
      pusher.trigger( 'private-chat', 'message-added', { message, name });
      res.sendStatus(200);
    });
    
    app.get('/',function(req,res){      
         res.sendFile('/public/index.html', {root: __dirname });
    });
    
    app.use(express.static(__dirname + '/public'));
    
    var port = process.env.PORT || 5000;
    app.listen(port, function () {
      console.log(`app listening on port ${port}!`)
    });

To break down the code, we first instantiate Pusher by passing in an object containing the details of our app ID and secret key. These can be found on the **App Keys** tab in your Pusher dashboard. Pusher also provides a mechanism for authenticating users to a channel at the point of subscription. To do this, we expose an endpoint on the server that will validate the request and respond with a success or failure. This endpoint will be called by Pusher client libraries and can be named anything. We used the default name for this endpoint on Pusher, which is `/pusher/auth`. The line `var auth = pusher.authenticate(socketId, channel);` authenticates the client with Pusher and returns an authentication code to the calling client. 

To allow this file to run when we start npm, we update **package.json** with the following value:


    "scripts": {
        "start": "node server.js",
        "test": "echo \"Error: no test specified\" && exit 1"
      }


#### Create an Auth0 client
To create an Auth0 client

1. Select  **Clients** from the side menu.
2. On the new page, click the **Create Client** button
3. Enter a name for the app and select **Single Page App** as an option
4. Click the **Create** button to create the client.


![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502092704077_Screen+Shot+2017-07-28+at+13.14.03.png)


An Auth0 client provides us with Client ID and Secret which we’ll use to interact with Auth0 from the code. On the settings tab, we can see the Name, Client Id, Secret, Client Type and many more. I want to enable Cross-Origin Resource Sharing (CORS) for my domain `http://localhost:5000` and set the log out URL and the post-authentication redirect URL. Update the following settings with **http://localhost:5000**:


1. Allowed Callback URLs
2. Allowed Logout URLs
3. Allowed Origins (CORS)

### Building the frontend
With the backend all good to go, we build the web page that will facilitate messaging. Create a folder named **public** which will contain the html and javascript file. Create two new files **style.css and** **index.html** with the following content:

**style.css** 

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

**index.html** 

    <!-- template from http://bootsnipp.com/snippets/6eWd -->
    <!DOCTYPE html>
    <html>
    <head>
        <!-- Latest compiled and minified CSS -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous">
        <!-- Optional theme -->
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous">
        <script
            src="https://code.jquery.com/jquery-2.2.4.min.js"
            integrity="sha256-BbhdlvQf/xTY9gja0Dq3HiwQF8LaCRTXxZKRutelT44="
            crossorigin="anonymous"></script>
        <!-- Latest compiled and minified JavaScript -->
        <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
        <link rel="stylesheet" href="style.css">
        <script src="https://cdn.auth0.com/js/lock/10.18.0/lock.min.js"></script>
        <script src="https://js.pusher.com/4.0/pusher.min.js"></script>
        <script src="index.js"></script>
    </head>
    <body>
        <div class="container">
        <div class="row form-group">
            <div class="col-xs-12 col-md-offset-2 col-md-8 col-lg-8 col-lg-offset-2">
                <div class="panel panel-primary">
                    <div class="panel-heading">
                        <span class="glyphicon glyphicon-comment"></span> <span id="username"></span>
                        <div class="btn-group pull-right">
                            <button type="button" class="btn btn-default btn-xs dropdown-toggle" data-toggle="dropdown">
                                <span class="glyphicon glyphicon-chevron-down"></span>
                            </button>
                            <ul class="dropdown-menu slidedown">
                                <li><a><span class="glyphicon glyphicon-refresh">
                                </span>Refresh</a></li>
                                <li><a><span class="glyphicon glyphicon-ok-sign">
                                </span>Available</a></li>
                                <li><a><span class="glyphicon glyphicon-remove">
                                </span>Busy</a></li>
                                <li><a><span class="glyphicon glyphicon-time"></span>
                                    Away</a></li>
                                <li class="divider"></li>
                                <li><a id="logout"><span class="glyphicon glyphicon-off"></span>
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
    <script id="new-message" type="text/template">
        <li id="{{id}}" class="right clearfix">
            <div class="chat-body clearfix">
                <div class="header">
                    <small class="text-muted">{{name}}</small>
                </div>
                <p>
                    {{body}}
                </p>
            </div>
        </li>
    </script>
    </body>
    </html>

This file uses template from [bootsnip](http://bootsnipp.com/snippets/6eWd) and also includes a script reference to Auth0 Lock `<script src="https://cdn.auth0.com/js/lock/10.18.0/lock.min.js"></script>`. [Lock](https://auth0.com/docs/libraries#lock-login-signup-widgets) is  a drop-in authentication widget that provides a standard set of behaviours required for login and a customizable user interface. It provides a simple way to integrate with Auth0 with very minimal configuration. 

We want to allow users to sign in when they enter the application, get authenticated, and only then be able to send messages. Add a new file **index.js** with the following content:


    $(document).ready(function(){
        // Initiating our Auth0Lock
        let lock = new Auth0Lock(
            'CLIENT_ID',
            'CLIENT_DOMAIN',//example: lotus.auth0.com
            {
                auth: {
                    params: {
                        scope: 'openid profile'
                    }   
                },
                autoclose: true,
                closable: false,
                rememberLastLogin: true
            }
        );
    
        let profile = JSON.parse(localStorage.getItem('profile'));
        let isAuthenticated = localStorage.getItem('isAuthenticated');
    
        function updateValues(userProfile, authStatus) {
            profile = userProfile;
            isAuthenticated = authStatus;
        }
        
        if(!isAuthenticated && !window.location.hash){
            lock.show();//show Lock widget
        }
    
        // Listening for the authenticated event
        lock.on("authenticated", function(authResult) {
            // Use the token in authResult to getUserInfo() and save it to localStorage
            lock.getUserInfo(authResult.accessToken, function(error, profile) {
                if (error) {
                    // Handle error
                    return;
                }
                
                localStorage.setItem('accessToken', authResult.accessToken);
                localStorage.setItem('profile', JSON.stringify(profile));
                localStorage.setItem('isAuthenticated', true);
                updateValues(profile, true);
                $("#username").html(profile.name);
            });
        });
    });

We initialise Lock by passing it the Client Id of the app, your user domain which starts with your username followed by `.auth0.com` or `.{YOUR_SELECTED_REGION}.auth0.com` e.g `lotus.eu.auth0.com`. The widget is configurable, so we can send in configuration options like *closable*, *autoClose*, and *auth*. Within the *auth* option, we tell it to return the `openid`  and `profile` claims. 

We check if the user is authenticated and show the widget when they’re not. Once the user is authenticated, Lock emits the `authenticated` event to which we’ve subscribed. When the event is raised, we store the user profile and other credentials to `localStorage` and set the user’s name to be displayed on the page. Once the user is authenticated, we want to connect to Pusher and send messages across. Update index.js with the following code: 


    if(!isAuthenticated && !window.location.hash){
        lock.show();
    }
    else{
        
        // Enable pusher logging - don't include this in production
        Pusher.logToConsole = true;
    
        var pusher = new Pusher('APP_SECRET', {
            cluster: 'e.g eu',
            encrypted: false
        });
    
        var channel = pusher.subscribe('private-chat');
        channel.bind('message-added', onMessageAdded); 
    }
    
    function onMessageAdded(data) {
        let template = $("#new-message").html();
        template = template.replace("{{body}}", data.message);
        template = template.replace("{{name}}", data.name);
    
        $(".chat").append(template);
    }

Pusher is initialised with the **APP_SECRET** and **CLUSTER** which you can get from the app dashboard on Pusher. We subscribe to a channel called `private-chat`. Pusher has 3 types of channels: Public, Private and Presence channel. Private and Presence channels let your server control access to the data you are broadcasting. Presence channels go further to force subscribers to register user information when subscribing. Private channels are named starting with `private-` and authenticated in the server when subscribing. 

And finally we want to send the message to the user when they click send and also log them out when they select signout. Update **index.js** with the code below


    $('#btn-chat').click(function(){
        const message = $("#message").val();
        $("#message").val("");
            //send message
        $.post( "http://localhost:5000/message", { message, name: profile.name } );
    }); 
    
    $("#logout").click((e) => {
        e.preventDefault();
        logout();
    });
    
    function logout(){
        localStorage.clear();
        isAuthenticated = false;
        lock.logout({ 
            returnTo: "http://localhost:5000" 
        });
    }

When the user clicks the send button, we take the message and put it in an object with the user’s profile name and send it to the `/message` endpoint on the server. When the logout button is clicked, it calls the logout function which clears the data stored in localStorage and call `lock.logout()` which logs the user out on Auth0 and redirects them back to our website. With all these additions, index.js should have the following content:


    $(document).ready(function(){
        // Initiating our Auth0Lock
        let lock = new Auth0Lock(
            'CLIENT_ID',
            'CLIENT_DOMAIN',
            {
                auth: {
                    params: {
                        scope: 'openid profile'
                    }   
                },
                autoclose: true,
                closable: false,
                rememberLastLogin: true
            }
        );
    
        // Listening for the authenticated event
        lock.on("authenticated", function(authResult) {
            // Use the token in authResult to getUserInfo() and save it to localStorage
            lock.getUserInfo(authResult.accessToken, function(error, profile) {
                if (error) {
                    // Handle error
                    console.log(error);
                    return;
                }
                
                localStorage.setItem('accessToken', authResult.accessToken);
                localStorage.setItem('profile', JSON.stringify(profile));
                localStorage.setItem('isAuthenticated', true);
                updateAuthenticationValues(profile, true);
                $("#username").html(profile.name);
            });
        });
    
        let profile = JSON.parse(localStorage.getItem('profile'));
        let isAuthenticated = localStorage.getItem('isAuthenticated');
    
        function updateAuthenticationValues(userProfile, authStatus) {
            profile = userProfile;
            isAuthenticated = authStatus;
        }
    
        $("#logout").click((e) => {
            e.preventDefault();
            logout();
        });
    
        function logout(){
            localStorage.clear();
            isAuthenticated = false;
            lock.logout({ 
                returnTo: "http://localhost:5000" 
            });
        }
        
        function onMessageAdded(data) {
            let template = $("#new-message").html();
            template = template.replace("{{body}}", data.message);
            template = template.replace("{{name}}", data.name);
    
            $(".chat").append(template);
        }
    
        if(!isAuthenticated && !window.location.hash){
            lock.show();
        }
        else{
            if(profile){
                $("#username").html(profile.name);
            }
            
            // Enable pusher logging - don't include this in production
            Pusher.logToConsole = true;
    
            var pusher = new Pusher('APP_SECRET', {
                cluster: 'eu',
                encrypted: false
            });
    
            var channel = pusher.subscribe('private-chat');
            channel.bind('message-added', onMessageAdded);
    
            $('#btn-chat').click(function(){
                const message = $("#message").val();
                $("#message").val("");
                 //send message
                $.post( "http://localhost:5000/message", { message, name: profile.name } );
            });  
        }
    });

To test the app, run `npm start` on the terminal and open `http://localhost:5000` on two separate browsers. Here’s a run through of it:


![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502274209684_build-secure-chat-app-with-auth0-and-pusher-in-javascript.gif)

# Wrapping up

This is an app to show how you can use Pusher to send messages in real-time and secure the channels, add user authentication and account management with Auth0, and easily integrate to Auth0 using Auth0 Lock. On your Auth0 [dashboard](https://manage.auth0.com/#/) you can see the total number of users, logins and new signups. 

![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502296411642_Screen+Shot+2017-08-09+at+17.27.18.png)

![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502296422795_Screen+Shot+2017-08-09+at+17.27.33.png)


You can also see all your users when you click on the **Users** side menu.  On this page you can see the list of your users and their mode of login. 


![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502296856204_Screen+Shot+2017-08-09+at+17.28.06.png)


Selecting a user takes you to a more detailed page where you can take various actions on the account, for example,  blocking an account or sending a verification email. 


![](https://d2mxuefqeaa7sj.cloudfront.net/s_98BAD045E14A60425D95386F6B1F7ED1D8E7A24C1FDC54BA88FB765B2ACC0033_1502296864217_Screen+Shot+2017-08-09+at+17.29.27.png)


Pusher also allows you to view statistics regarding your application. To do this, go to your application dashboard, under **Stats,** where you’ll see statistics concerning your application, such as connection frequency and how many messages were sent through that app. The combination of these two technologies makes it faster and easier to build real-time secured applications. 

You can find the code here on [GitHub](https://github.com/pmbanugo/Pusher-Auth0-ChatApp). Hopefully this guide has been an effective introduction to Auth0 and Pusher and a quality springboard to help you set up authentication faster in your upcoming applications.
____

Thanks for reading this guide. As always, please drop your comments and feedback in the discussion section, and don't be afraid to add this guide to your favorites. Till next time.

*This was originally published on [Pusher](https://blog.pusher.com/build-secure-chat-web-app-javascript-auth0-pusher/)*.