# Building a Realtime Collaborative Editor with RethinkDB

Today we are going to tackle an interesting app. We are going to build a collaborative editor that uses RethinkDB to sync data between users. First off we have to answer the blarring question out there.

## What is RethinkDB? [https://www.rethinkdb.com/](https://www.rethinkdb.com/)

RethinkDB is a new concept on the old database. It pushes changed data to your app in realtime, which makes building realtime applications less stressfull. Instead of polling for data, you just sit and wait until the data changes. When the data is changed RethinkDB notifies you that instant. It is open source, it scales very well, and it is just plain cool.

![Imgur](http://i.imgur.com/RkSKcAq.png)

## Ok, So how do we build this?

We are going to use a handful of technology to help us build this app. I want to keep it small to demonstrate what it is doing well. The technology is as follows - with a little description:

* NodeJS
    * This allows us to run JavaScript on the back-end. It does so much more than just that, but that pretty well sums it up.
* Express Framework
    * This framework allows us to serve the static files and assets, as well as build a small api to retrieve data with.
* Socket.IO
    * With socket.io we can send realtime data to all connected clients with the use of sockets. This plus rethinkDB equals heaven.
* RethinkDB
    * We discussed this already, the core behind our document storage and retrieval.
* CodeMirror
    * This is a JavaScript library that builds a full featured code editor with little work.
* Jquery
    * The ever popular JavaScript library that makes things 'easier?'... We are going to be using the AJAX Portion of it.
* Bower
    * An awesome package manager for the front-end.

## Getting Started - Installing RethinkDB

To install rethinkDB just visit their site, find the appropriate operating system, and follow the instructions. I am using Ubuntu on my dev server, so here is the script I'll run in bash to install rethinkDB

```bash
source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install rethinkdb
```

After it is installed you can run RethinkDB with a simple command `rethinkdb` however because I want to free up the **8080 port**, I am going to run RethinkDB without the admin interface... `rethinkdb --no-http-admin`.

RethinkDB should be up and running on your system now.

## Getting Started - Setting Up NodeJS

I already have NodeJS setup on all my dev systems automatically. However to install on Ubuntu just run the following script in the console.

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Once installed, you can cd to a folder you wish to store your project in.

##### Setup NodeJS Package

Run the `NPM Init` command and follow the on screen prompts. This will setup your nodejs packate.

##### Setup RethinkDB Node Package

Install the RethinkDB adapter with the command `npm install --save rethinkdb`.

##### Setup Express Node Package

Install the Express package with the command `npm install --save express`.

##### Setup Socket.IO Node Package

Install the Socket.IO package witht he command `npm install --save socket.io`.

## Getting Started - Setting Up Bower

Install bower using the following command `npm install -g bower`. This will install it globally so that you can use it in future projects as well..

Once bower is installed then you can initialize it with `bower init`. Follow the on screen prompts, and if you already setup your NPM Init then you should have all your prompts pre-filled from the npm package info.

After you have initialized bower it is time to install our front-end dependencies. 

##### Setup Jquery

Install Jquery with the following command `bower install --save jquery`

##### Setup Codemirror

Install Codemirror with the following command `bower install --save codemirror`

## Writing Actual Code

By now you have your project setup and ready to use the techonlogies that we have installed. The first thing we want to do is create the two files that we will need. To do this in the folder of your project, and in your console run `touch index.html` and `touch index.js`. This will create two blank files.

In the HTML you want to put the following Code:

```html
<!doctype html>
<html>
  <head>
    <title>Collaborative Editor</title>
  </head>
  <body>
    <textarea id="editor"></textarea>
     
    <script src="/bower_components/codemirror/lib/codemirror.js"></script>
    <link rel="stylesheet" href="/bower_components/codemirror/lib/codemirror.css">
    <script src="/bower_components/codemirror/mode/javascript/javascript.js"></script>
    <script src="/socket.io/socket.io.js"></script>
    <script src="/bower_components/jquery/dist/jquery.min.js"></script>
    
    <script>
        var myCodeMirror = CodeMirror.fromTextArea(document.getElementById("editor"), {
            lineNumbers: true,
            mode: "javascript"
        });
    </script>
  </body>
</html>
```

Let's talk about this block of code. This is going to be the basis for the editor's html page. We create a text area with an ID of editor that we can refer to later. Below the text editor we include all of the javascript files that we want to use. These include files for codemirror, socket.io (Which will be served by socket.io itself) and Jquery.

We then initialize the codemirror editor for the user to interact with.

Next we need to open up index.js and input the following code:

```javascript
var r = require('rethinkdb');
var express = require('express');
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

// Serve HTML
app.get('/', function(req, res) {
  res.sendFile(__dirname + '/index.html');
});

app.use('/bower_components', express.static('bower_components'));

// Setup Express Listener
http.listen(8080, '0.0.0.0', function(){
  console.log('listening on: 0.0.0.0:8080');
});
```

We start out by importing all of the packages that we installed earlier. Then we setup two routes using express. The first route serves our `index.html` page, and the second route serves our `bower_components` folder so the html page can access it. If you were to run `node index.js` and navigate to `http://localhost:8080`, then you should see an html page that shows the codemirror editor. If you edit it, text changes and that is all..

![Imgur](http://i.imgur.com/zj7FMk7.png)

Now lets add some socket stuff...

Add the following code above the line that starts with `// Serve HTML` in `index.js`.

```javascript
io.on('connection', function(socket){
  console.log('a user connected');
  socket.on('disconnect', function(){
    console.log('user disconnected');
  });
});
```

Add the following code above the `</script>` tag in `index.html`.

```javascript
var socket = io();
```

Now the first block tells the server what to do on a socket connection. In this case when someone connects then we are going to display to the node console `a user connected` and vice versa for a disconnect we will display `user disconnected`. 

Go ahead and run `node index.js` and navigate to your url `http://localhost:8080`. When you do so you should see the messages pop up in your console.

![Imgur](http://i.imgur.com/TII9xEC.png)

Now I know when I create this app that I want to be able to separate editors into different rooms. Allowing for multiple documents to be edited and shared on a single url. To do this I need to add the query parameter to my url and build a way to get it. Add the following to your javascript section in `index.html`

```javascript
function getParameterByName(name, url) {
    if (!url) url = window.location.href;
    name = name.replace(/[\[\]]/g, "\\$&");
    var regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)"),
        results = regex.exec(url);
    if (!results) return null;
    if (!results[2]) return '';
    return decodeURIComponent(results[2].replace(/\+/g, " "));
}
var room = getParameterByName('room');
var user = getParameterByName('user');
```

The first part is a function that parses out query parameters from the url bar based on the given name. The second part stores a room and a user. Doing this we need to format the url like so `http://localhost:8080/?room=1&user=username` This will past `1` as the room name and `username` as the well... user...

Next we are going to setup the rethinkDB databases. Place the following code above your socket section.

```javascript
r.connect({ host: 'localhost', port: 28015 }, function(err, conn) {
    if(err) throw err;
    r.db('test').tableList().run(conn, function(err, response){
      if(response.indexOf('edit') > -1){
        // do nothing it is created...
        console.log('Table exists, skipping create...');
        console.log('Tables - ' + response);
      } else {
        // create table...
        console.log('Table does not exist. Creating');
        r.db('test').tableCreate('edit').run(conn);  
      }
    });
});
```

When you install RethinkDB it automatically sets up a test database. We are going to use that for our project. (feel free to try and modify your code to add a new database for you to use, if you want). This makes it easy, and quick. The first part of this connects to rethinkDB. Then anytying we wantot o do with rethink needs to happen within that connection block. The second thing we do is call a `.tableList()` on the database. We want to see if our edit table is already created, if it is we are just going to list the current tables in the database. If the table doesn't exist then we are going to create it. It is a simple way to create the database on the first run, and skip that code every run after that. Otherwise we would get an error that the table already exists everytime we tried to created it.

Now most of the stuff we are going to be doing with data is going to require the rethinkDB so replace the rethinkDB and Socket.io sections to look like this:

```javascript
r.connect({ host: 'localhost', port: 28015 }, function(err, conn) {
    if(err) throw err;
    r.db('test').tableList().run(conn, function(err, response){
      if(response.indexOf('edit') > -1){
        // do nothing it is created...
        console.log('Table exists, skipping create...');
        console.log('Tables - ' + response);
      } else {
        // create table...
        console.log('Table does not exist. Creating');
        r.db('test').tableCreate('edit').run(conn);  
      }
    });
    
    // Socket Stuff
    io.on('connection', function(socket){
      console.log('a user connected');
      socket.on('disconnect', function(){
        console.log('user disconnected');
      });
    });
});
```

This means that anything we do in the socket block will have access to the connection of the rethinkDB data.

Moving to the `index.html` file lets add a new section of code: 

```javascript
myCodeMirror.on('keyup', function () {
    var msg = {
        id: room,
        user: user,
        value: myCodeMirror.getValue()
    }
    socket.emit('document-update',msg);
});
```

In the `index.js` file add the following code to your socket block.

```javascript
socket.on('document-update', function(msg){
    console.log(msg);
    r.table('edit').insert({id: msg.id,value: msg.value, user: msg.user}, {conflict: "update"}).run(conn, function(err, res){
      if (err) throw err;
      //console.log(JSON.stringify(res, null, 2));
    });
  });
```

The first set of code makes it so that everytime a keypress goes up on the codemirror editor, we will do something. In this case we are going to send a socket emit labeled `document-update` with the value of the editor's contents. 

In the second set of code we are going to recieve that socket message and store it into the rethinkdb table. Notice that it is an insert and that I have a piece of code that looks like `{conflict: "update"}`. This allows me to insert data when it comes in. If it is new data, then it will insert a new record into rethinkDB. However, if the data coming in matches a room id that is already present in the rethinkDB database then we will update that record instead of inserting a new one.

If you were to run this and type in the codemirror editor, then you wouldn't see anything happening at all... To do that let's add some more code blocks.

Add the following to `index.js` below the socket item we just did within the socket block:

```javascript
r.table('edit').changes().run(conn, function(err, cursor) {
    if (err) throw err;
    cursor.each(function(err, row) {
        if (err) throw err;
        io.emit('doc', row);
    });
});
```

add the following to `index.html` in your javascript section:

```javascript
socket.on('doc', function(msg){
    if(msg.new_val.id === room && msg.new_val.user != user) {
        var current_pos = myCodeMirror.getCursor();
        myCodeMirror.getDoc().setValue(msg.new_val.value);
        myCodeMirror.setCursor(current_pos);    
    }
});
```

The first block of code works the magic that is RethinkDB. We are using their changes function to watch a table of data. Anytime a row of data changes it will inform us. When it does inform us of the change we will send that document out through the socket to all connected clients. This will happen in realtime as new data is pushed to the server.

The second block of code receives the data on the `doc` socket and checks to make sure it is for this client. If the data is for the current room and it wasn't submitted by the current user then we want it. The reason we don't want data submitted by the current user is because it can cause a typing glitch. As data is sent to the server it is sent back to us, and when we retrive it we are storing it into the codemirror editor. If we were the ones that sent it we may have already typed a new letter before we recieved the update. This would cause our typed letter to disappear. By checking that the data wasn't sent by the current user, then we can avoid this issue completely.

Next let's add a few more details. Add the following code to `index.js` It needs to be added within the rethinkDB connection block:

```javascript
app.get('/getData/:id', function(req, res, next){
    r.table('edit').get(req.params.id).
        run(conn, function(err, result) {
            if (err) throw err;
            res.send(result);
            //return next(result);
    });
});
```

Add the following the the javascript section of your `index.html`:

```javascript
$.ajax({
    url: '/getData/' + room,
    success: function(result, status, xhr) {
        myCodeMirror.setValue(result.value);
        console.log(result);
    }
});
```

The first block of code here sets up an API for our client side to load in data for the current room. It passes an ID parameter through the URL and then uses that to query rethinkDB and return the data.

The second block of code sets up the loading call from the client side. It stores the result of the call into the codemirror editor. Now when we join a room, if there is already data in the editor we will retrieve it and show it. Otherwise it will be empty and we can start the new room by typing in the editor.

Now let us spruce the page up a little bit and add some css.

Add the following css to your `index.html` page:

```css
html, body {
  height: 100%;
  width: 100%;
  background-color:#333;
  font-family: arial;
}
#editor-wrapper {
  width: 70%;
  margin-left:auto;
  margin-right:auto;
  margin-top:3em;
}
#username {
  color: #f3f3f3;
}
.cuser {
  color: #999;
}
.ouser {
  color: #fff;
}
```

Replace `<textarea id="editor"></textarea>` in `index.html` with the following code:

```html
<div id="editor-wrapper">
    <span class="cuser">Current User: </span><span id="username"></span><br>
    <textarea id="editor"></textarea>    
</div>
```

Add the following javascript to the script section of `index.html`:

```javascript
document.getElementById("username").innerHTML = user;
```

This will spruce up the page a bit, and add the username you specified to the page. By now you should have a working application, and it should look something like this...

![Imgur](http://i.imgur.com/aeY9bD7.png)

And here is a recorded demo of using the editor...

![Imgur](http://i.imgur.com/e1QhrZk.gif)


If your application is breaking and you do not know why, then here is exactly what your two files should be looking like by now...

Here is `index.js`:

```javascript
var r = require('rethinkdb');
var express = require('express');
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);

// Setup Database
r.connect({ host: 'localhost', port: 28015 }, function(err, conn) {
    if(err) throw err;
    r.db('test').tableList().run(conn, function(err, response){
      if(response.indexOf('edit') > -1){
        // do nothing it is created...
        console.log('Table exists, skipping create...');
        console.log('Tables - ' + response);
      } else {
        // create table...
        console.log('Table does not exist. Creating');
        r.db('test').tableCreate('edit').run(conn);  
      }
    });
    
    // Socket Stuff
    io.on('connection', function(socket){
      console.log('a user connected');
      socket.on('disconnect', function(){
        console.log('user disconnected');
      });
      socket.on('document-update', function(msg){
        console.log(msg);
        r.table('edit').insert({id: msg.id,value: msg.value, user: msg.user}, {conflict: "update"}).run(conn, function(err, res){
          if (err) throw err;
          //console.log(JSON.stringify(res, null, 2));
        });
      });
      r.table('edit').changes().run(conn, function(err, cursor) {
          if (err) throw err;
          cursor.each(function(err, row) {
              if (err) throw err;
              io.emit('doc', row);
          });
      });
      
    });
    
    app.get('/getData/:id', function(req, res, next){
      r.table('edit').get(req.params.id).
        run(conn, function(err, result) {
            if (err) throw err;
            res.send(result);
            //return next(result);
        });
    });
});

// Serve HTML
app.get('/', function(req, res) {
  res.sendFile(__dirname + '/index.html');
});

app.use('/bower_components', express.static('bower_components'));

// Setup Express Listener
http.listen(process.env.PORT, process.env.IP, function(){
  console.log('listening on:  ' + process.env.IP + ':' + process.env.PORT);
});
```

Here is `index.html`:

```html
<!doctype html>
<html>
  <head>
    <title>Collaborative Editor</title>
    <style>
      html, body {
          height: 100%;
          width: 100%;
          background-color:#333;
          font-family: arial;
      }
      #editor-wrapper {
          width: 70%;
          margin-left:auto;
          margin-right:auto;
          margin-top:3em;
      }
      #username {
          color: #f3f3f3;
      }
      .cuser {
          color: #999;
      }
      .ouser {
          color: #fff;
      }
    </style>
  </head>
  <body>
      <div id="editor-wrapper">
        <span class="cuser">Current User: </span><span id="username"></span><br>
        <textarea id="editor"></textarea>    
      </div>
    <script src="/bower_components/codemirror/lib/codemirror.js"></script>
    <link rel="stylesheet" href="/bower_components/codemirror/lib/codemirror.css">
    <script src="/bower_components/codemirror/mode/javascript/javascript.js"></script>
    <script src="/socket.io/socket.io.js"></script>
    <script src="/bower_components/jquery/dist/jquery.min.js"></script>
    <script>
    
        function getParameterByName(name, url) {
            if (!url) url = window.location.href;
            name = name.replace(/[\[\]]/g, "\\$&");
            var regex = new RegExp("[?&]" + name + "(=([^&#]*)|&|#|$)"),
                results = regex.exec(url);
            if (!results) return null;
            if (!results[2]) return '';
            return decodeURIComponent(results[2].replace(/\+/g, " "));
        }
        var room = getParameterByName('room');
        var user = getParameterByName('user');
        document.getElementById("username").innerHTML = user;
        
        var myCodeMirror = CodeMirror.fromTextArea(document.getElementById("editor"), {
            lineNumbers: true,
            mode: "javascript"
        });
      
        var socket = io();
        
        $.ajax({
            url: '/getData/' + room,
            success: function(result, status, xhr) {
                myCodeMirror.setValue(result.value);
                console.log(result);
            }
        });
        
        myCodeMirror.on('keyup', function () {
            var msg = {
                id: room,
                user: user,
                value: myCodeMirror.getValue()
            }
            socket.emit('document-update',msg);
        });
      
        socket.on('doc', function(msg){
            if(msg.new_val.id === room && msg.new_val.user != user) {
                var current_pos = myCodeMirror.getCursor();
                myCodeMirror.getDoc().setValue(msg.new_val.value);
                myCodeMirror.setCursor(current_pos);    
            }
        });
      
    </script>
  </body>
</html>
```

## About The Author

![Shannon Duncan](https://pbs.twimg.com/profile_images/672481536826937344/GeAx6xl4_200x200.jpg) 

Shannon Duncan is the Author at [Unrestricted Coding](http://unrestrictedcoding.com). He mentors and teaches people of all ages how to code and enjoy the art of programming. Find him on [twitter](https://twitter.com/TheUCofficial), [linked-in](https://www.linkedin.com/in/jsduncan98), and [github](https://github.com/shadowcodex).


