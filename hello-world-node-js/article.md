**Getting Started With Node.js**

![node.js](https://nodejs.org/static/images/logos/nodejs-new-white-pantone.png "node.js")

<hr/>
<h1>Introduction</h1>
Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. 
<br/><br/>
V8 is Google's open source high-performance JavaScript engine, written in C++ and used in Google Chrome, the open source browser from Google.V8 can run standalone, or can be embedded into any C++ application.
<br/><br/>
<hr/>
<h1>Installation</h1>

To install Node.js on your machine using your terminal :

<br/><br/>

_first lets update our packages_

```
hassan@home:~$ sudo apt-get update
```
<br/>
_now we will install node.js using this command_


```
hassan@home:~$ sudo apt-get install nodejs

```

<br/>


Now we need to install **npm** (the node package manager) is the default package manager for the JavaScript runtime environment Node.js.

_use this command to install the node package manager_

```
hassan@home:~$ sudo apt-get install npm

```

<hr/>
<h1>Hello World</h1>

lets create our first simple hello world application using node.js , whats fascinating about node is that you start from the ground up by creating a server and handling requests and response , check this out :

<br/>
_create a file index.js on you application directory_

```javascript
// requiring the HTTP interfaces in node
var http = require('http');

// create an http server to handle requests and response
http.createServer(function (req, res) {

  // sending a response header of 200 OK
  res.writeHead(200, {'Content-Type': 'text/plain'});

  // print out Hello World
  res.end('Hello World\n');

// use port 8080
}).listen(8080);


console.log('Server running on port 8080.');
```

Now run the the application using the following command :

```
hassan@home:~$ node index.js
```

your application is running on port 8080 now so just open your browser and hit http://localhost:8080

<br/>
<hr/>
<h1>How to use npm ? </h1>

the node package manager will give you the ability to install modules that other people develop and published on https://www.npmjs.com.

using npm will reflect everything you install on a package.json file that lists all modules and information about your application.

all the modules you install will be placed in a node_modules folder on the root of your application and you can use any module by requiring it exactly like the http module.

lets create the package.json file :

 ```
hassan@home:~$ sudo npm init 

```

the command will prompt you with some questions about your application to add to the package.json file.

now lets install the express framework module as an example :

 ```
hassan@home:~$ npm install express

```

if you get any errors just try it again with "sudo" before it.


<hr/>
<br/>
<h1>Hello world using express</h1>

```javascript
var express = require('express')
var app = express()
 
app.get('/', function (req, res) {
  res.send('Hello World')
})
 
app.listen(8080)
```

