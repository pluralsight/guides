Today we are going to tackle an interesting app. We are going to build a collaborative editor that uses RethinkDB to sync data between users. 

First off, however, we have to answer the big question out there:

## [What is RethinkDB?](http://tutorials.pluralsight.com/nosql-databases/a-practical-introduction-to-rethinkdb)

RethinkDB is a new take on the typical database. It pushes changed data to your app in realtime, which makes building realtime applications less stressful. Instead of polling for data, you can wait for the data to change. Once the data changes, RethinkDB notifies you, allowing you to make changes as needed. It is open source, it scales very well, and it is just plain cool.

![Imgur](http://i.imgur.com/RkSKcAq.png)

## Ok, So how do we build this?

We are going to use a handful of technologies to help us build this app. I want to keep it small to demonstrate what it is doing well. You will need the following:

* Node.js
    * This allows us to run JavaScript on the back-end and much, much more.
* Express Framework
    * This framework allows us to serve the static files and assets, as well as build a small API to retrieve data.
* Socket.IO
    * With Socket.IO, we can send realtime data to all connected clients.
* RethinkDB
    * The core of our document storage and retrieval.
* CodeMirror
    * This JavaScript library builds a fully-featured code editor with little work.
* jQuery
    * The ever-popular JavaScript library that makes things "easier." We are going to be using jQuery's AJAX (Asynchronous JavaScript and XML).
* Bower
    * An awesome package manager for front-end development.

## Getting Started - Installing RethinkDB

To install RethinkDB just visit their site, find the package matching your operating system, and follow the instructions. I am using Ubuntu on my dev server, so here is the script I'll run in bash to install RethinkDB

```bash
source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install rethinkdb
```

After it is installed, you can run RethinkDB with a simple command `rethinkdb`. However because I want to free up the **8080 port**, I am going to run RethinkDB without the admin interface... `rethinkdb --no-http-admin`.

RethinkDB should be up and running on your system now.

## Getting Started: Setting Up Node.js

I already have NodeJS setup on all my dev systems automatically. However, to install it on Ubuntu, just run the following script in the console.

```bash
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
```

Once installed, you can `cd` to a folder you wish to store your project in.

##### Setup NodeJS Package

Run the `NPM Init` command and follow the on-screen prompts. This will set up your Node.js package.

##### Setup RethinkDB Node Package

Install the RethinkDB adapter with the command `npm install --save rethinkdb`.

##### Setup Express Node Package

Install the Express package with the command `npm install --save express`.

##### Setup Socket.IO Node Package

Install the Socket.IO package with the command `npm install --save socket.io`.

## Getting Started - Setting Up Bower

Install Bower using the following command `npm install -g bower`. This will install Bower globally, so you can use it in future projects as well.

Once Bower is installed, you can initialize it with `bower init`. Follow the on-screen prompts. If you've already set up your NPM Init, then you should have all your prompts pre-filled from the npm package info.

After you have initialized Bower, it is time to install our front-end dependencies. 

##### Setup jQuery

Install jQuery with the following command: `bower install --save jquery`

##### Setup Codemirror

Install Codemirror with the following command: `bower install --save codemirror`

## Setting up your HTML page

By now, you're ready to use the technologies that we have installed. First, we need to create the two files that we will need. In the folder of your project, run the console commands `touch index.html` and `touch index.js` to create two blank files.

In the HTML, you want to put the following Code:

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

The code block above is going to be the basis for the editor's HTML page. In the block, we create a text area with an ID of the editor that we can refer to later. Below this, we include all of the javascript files that we want to use. These include files for CodeMirror, Socket.IO (which will be served by Socket.IO itself), and jQuery. We then initialize the CodeMirror editor, with which the user will interact. 

Next, open up `index.js`, and input the following code:

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

We start off by importing all of the packages that we installed earlier. Then we set up two routes using Express. The first route serves our `index.html` page, and the second route serves our `bower_components` folder. As a result, the HTML page can access the bower folder. If you were to run `node index.js` and navigate to `http://localhost:8080`, then you should see an HTML page with the CodeMirror editor. If you edit it, then the text on the page changes.

![Imgur](http://i.imgur.com/zj7FMk7.png)

### Now let's work with Socket.IO.

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

The first block tells the server what to do on a Socket connection. In this case, when someone connects,  we are going to display to the node console `a user connected`. The opposite will occur for a disconnect; we will display `user disconnected`. 

Go ahead and run `node index.js`, and navigate to your url `http://localhost:8080`. When you do so you should see the messages pop up in your console.

![Imgur](http://i.imgur.com/TII9xEC.png)

Now I know when I create this app that I want to be able to separate editors into different rooms. Allowing for multiple documents to be edited and shared on a single URL. To do this, I need to add the query parameter to my URL and build a way to get it. Add the following code to your javascript section in `index.html`

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

The first part is a function that parses out query parameters from the URL bar based on the given name. The second part stores a room and a user. Doing this, we need to format the URL like so: `http://localhost:8080/?room=1&user=username`. 

This will pass `1` as the room name and `username` as the user.

### Application-specific RethinkDB setup

Next, we are going to set up the RethinkDB databases. Place the following code above your Socket section.

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

When you install RethinkDB, it automatically sets up a test database. We are going to use that for our project. (feel free to try and modify your code to add a new database for you to use, if you want). 

The first part of this code block connects us to RethinkDB. Then anything we want to do with the database needs to happen within that connection block. Next we call a `.tableList()` on the database. We want to see if our edit table is already created; if it is, then we are just going to list the current tables in the database. If the table doesn't exist, then we are going to create it. It is a simple way to create the database on the first run, and skip that code every run after that. Otherwise we would get an error that the table already exists every time we tried to created it.

Most of the stuff we are going to be doing with data is going to require RethinkDB, so patch up the RethinkDB and Socket.IO sections to look like this:

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

This means that anything we do in the Socket block will have access to the connection of the RethinkDB data.

Moving onto the `index.html` file, let's add a new section of code: 

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

In the `index.js` file, add the following code to your socket block.

```javascript
socket.on('document-update', function(msg){
    console.log(msg);
    r.table('edit').insert({id: msg.id,value: msg.value, user: msg.user}, {conflict: "update"}).run(conn, function(err, res){
      if (err) throw err;
      //console.log(JSON.stringify(res, null, 2));
    });
  });
```

The first set of code makes it so that every time a keypress goes up on the CodeMirror editor, we will do something. In this case, we are going to send a Socket edit labeled `document-update` with the value of the editor's contents. 

In the second set of code, we are going to receive that socket message and store it into the RethinkDB table. Notice that it is an insertion and that I have a piece of code that looks like `{conflict: "update"}`. This allows me to insert data when it comes in. If the data is new, then the code will insert a new record into RethinkDB. However, if the incoming data matches a pre-existing room ID in the RethinkDB database, then we will update that record instead of inserting a new one.

But if you were to run this and type in the CodeMirror editor, then you wouldn't see anything happening at all. Let's fix that by adding the following code:

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

Add the following to `index.html` in your javascript section:

```javascript
socket.on('doc', function(msg){
    if(msg.new_val.id === room && msg.new_val.user != user) {
        var current_pos = myCodeMirror.getCursor();
        myCodeMirror.getDoc().setValue(msg.new_val.value);
        myCodeMirror.setCursor(current_pos);    
    }
});
```

The first block of code works the magic that is RethinkDB. We are using their `changes` function to watch a table of data. Any time a row of data changes, the database will inform us. When it does inform us of the change, we will send that document out through the Socket to all connected clients. This will happen in realtime as new data is pushed to the server.

The second block of code receives the data on the `doc` socket and checks to make sure it is intended for this client. If the data is for the current room and it wasn't submitted by the current user, then we want it. 

*The reason we don't want data submitted by the current user is because it can cause a typing glitch. As data is sent to the server, it is sent back to us. And when we retrieve it we are storing it into the codemirror editor. If we were the ones that sent it, we may have already typed a new letter before we recieved the update. This would cause our typed letter to disappear. By checking that the data wasn't sent by the current user, then we can avoid this issue completely.*

Next let's add a few more details. Add the following code to `index.js` It needs to be added **within the RethinkDB connection block:**

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

The first block of code here sets up an API for our client side to load in data for the current room. It passes an ID parameter through the URL and then uses that to query RethinkDB and return the data.

The second block of code sets up the loading call from the client side. It stores the result of the call into the CodeMirror editor. Now when we join a room, if there is already data in the editor, we will retrieve it and show it. Otherwise it will be empty and we can start the new room by typing in the editor.

Now let us spruce the page up a little bit and add some CSS.

Add the following CSS to your `index.html` page:

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

This will spruce up the page a bit, and add the username you specified to the page. By now you should have a working application, and it should look something like this.

![Imgur](http://i.imgur.com/aeY9bD7.png)

And here is a recorded demo of using the editor...

![Imgur](http://i.imgur.com/e1QhrZk.gif)

_________


If your application is breaking, and you do not know why, then here is exactly what your two files should be looking like by now...

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

## Summary

Using this method, you can create a very simple editor with realtime collaboration features. However, this is just the beginning! To make the editor even better,  you can implement a login system, chats, data validation, and much, much more. 

If you are interested in designing a larger collaborative editor, make sure that you understand [Operational Transformation](https://en.wikipedia.org/wiki/Operational_transformation). This is what large-scale collaborative editors use to make sure that every edit you make doesn't overwrite something else that someone is doing. It also handles collision of data, and can help relay information correctly with more than just text data.

Stay tuned, as I plan on writing an article about Operational Transformation. I'll discuss all of the aspects of it, as well as how to add it into your current projects.

I hope this tutorial piques your interest, and you can go off and make amazing apps with RethinkDB.

Shannon Duncan is an Author at [Unrestricted Coding](http://unrestrictedcoding.com). He mentors and teaches people of all ages how to code and enjoy the art of programming. Find him on [Twitter](https://twitter.com/TheUCofficial), [Linkedin](https://www.linkedin.com/in/jsduncan98), and [GitHub](https://github.com/shadowcodex).

