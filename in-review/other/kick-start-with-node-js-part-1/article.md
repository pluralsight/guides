In this article , I am going to discuss about the basics of Node JS starting with its introduction, features and where it is used commonly. Also if you going for an interview for node JS profile, questions with red asterik can be helpful.

Before starting with our first Node Js app, let us Know what is node Js all about?
<br><h4>What Is Node JS ? <i style="color:red;font-size:30px">*</i></h4>

It's a very powerful Javascript-based framework (platform) built on Google Chrome's Javascript V8 Engine.

<h4>Features of Node Js <i style="color:red;font-size:30px">*</i></h4>
1.  Asynchronous And Event Driven.
2.  Very Fast
3.  Single Threaded but Highly scalable.
4.  No Buffering

<br><h4>Who are using Node JS</h4>
Following is the link on github Wikip containing an exhaustive listof projects, applications nd companies which are using Node Js. The list includes eBay, General Electric, GoDaddy, Microsoft,Paypal, Uber, Wikipedia,Yahoo and Yammer to name a few 

Link : https://github.com/nodejs/node

<br><h4>Where to use Node Js?</h4>
1. I/O Bound Applications.
2. Data streaming Applications
3. Data Intensive Real Time Applications (DIRT)
4. JSON API based Applications
5. Single PAge Applications

As it is an open source , easy to learn ,Node Js is a preferable choice of most developers in the world. 
So Guys, before proceeding make sure you have some basics of Javascript and it will be good to have some understanding of web technologies like HTML ,CSS and AJAX.

Now , lets do some action with Node.
First of all, download the Node Js on your native machine using the following link.
https://nodejs.org/en/

And after the installation , verify it by creating a js file named example.js or (anyName.js) having the following code.

<code>console.log("Hello World, Learning Node Js");</code>

and execute it with Node.js using the command.
<code>$ node example.js</code>

if it successfully prints <h5>Hello World, Learning Node Js</h5> then you have achieved your first goal.

<h5>Now lets have a real world node example</h5>
Points we will cover are:
We will require a  directive to load a Node.js module (http module in this case) and a server which will listen to client's request similar to Apache HTTP Server which will read HTTP request made by client which can be a browser or console and return the response.

In the same file which was created earlier (example.js) write the following code:
<code>var http = require("http");</code>
Here we are using require directive to load http module and store returned HTTP instance into it.

Next we eill create server instance by using the inbuilt http.createServer() method and then we bind it at port 8081 using listen method associated with server instance as follows:

<code>
http.createServer(function (request, response) {

   // Send the HTTP header 
   // HTTP Status: 200 : OK
   // Content Type: text/plain
   response.writeHead(200, {'Content-Type': 'text/plain'});
   
   // Send the response body as "Hello World"
   response.end('Hello World\n');
}).listen(8081);

// Console will print the message
console.log('Server running at http://127.0.0.1:8081/');
</code>

We are passing a function with request and response paramters which will make a server request
and finally listens ie. wait for a request over 8081 port on local machine and respond with the output mentioned.

so our final file we as follows:
<code>
var http = require("http");

http.createServer(function (request, response) {

   // Send the HTTP header 
   // HTTP Status: 200 : OK
   // Content Type: text/plain
   response.writeHead(200, {'Content-Type': 'text/plain'});
   
   // Send the response body as "Hello World"
   response.end('Hello World\n');
}).listen(8081);

// Console will print the message
console.log('Server running at http://127.0.0.1:8081/');

</code>

<h5>Now run the file to start the server as follows:</h5><code>$ node example.js</code>

Open http://127.0.0.1:8081/ in any browser and see your achievement.


<h4>Congratulations, you have your first HTTP server up and running which is responding all the HTTP requests at port 8081.</h4>
<p>We will learn a new program in next tutorial of Node Js</p>
<h4 style="color:purple">Happy Coding.</h4>

