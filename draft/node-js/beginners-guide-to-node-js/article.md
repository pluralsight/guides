![node.js](https://nodejs.org/static/images/logos/nodejs-new-white-pantone.png "node.js")

## Introduction

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js is especially efficient and lightweight because it uses an event-driven, non-blocking I/O model.
<br/><br/>
V8 is Google's open source high-performance JavaScript engine, written in C++ and used in Google Chrome, which is Google's open-source web browser. V8 can run alone or can be embedded into any C++ application.


### Installation

To install Node.js on your machine using the terminal:

First we will update our packages.

```
hassan@home:~$ sudo apt-get update
```

Then we will install node.js using the below command.

```
hassan@home:~$ sudo apt-get install nodejs

```

Next we will install the **node package manager** (or npm), which is the default package manager for Node.js.

In case you do not have npm already installed, use this command to install it:

```
hassan@home:~$ sudo apt-get install npm

```

### Hello World

Let's create our first simple task: a hello world application using Node.js. The most fascinating part about Node is that it allows users to start from the ground-up. We first create a server, then handle requests and responses. 

For instance, suppose we create a file index.js on the application directory.

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

Then we run the the application using the following command:

```
hassan@home:~$ node index.js
```

If you open Chrome and hit http://localhost:8080, you will see your application as it's running on Port 8080.

### How do we use npm

NPM will give you the ability to crowd-source, or install modules that other people develop and publish on (the npm main site) [https://www.npmjs.com].

Using npm will reflect everything you install on a package.json file which lists all modules and information about your application.

All modules that you install will be placed in a node_modules folder on the root of your application. You can use any module by requiring it exactly like the HTTP module.

Let's create the package.json file :

 ```
hassan@home:~$ sudo npm init 

```

The command will prompt you with some questions about your application to add to the package.json file. Now we can install the Express framework module as an example:

 ```
hassan@home:~$ npm install express

```

If you get any errors, just try the same command again, but with "sudo" before "npm install express".

### Hello world using Express

```javascript
var express = require('express')
var app = express()
 
app.get('/', function (req, res) {
  res.send('Hello World')
})
 
app.listen(8080)
```

Hello World should now appear. 
