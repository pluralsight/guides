Every day we try to simplify the way we build web solutions so that more individuals appreciate and join the trend. "Serverless" concepts and realtime stores are examples of such simplification. These ideas are powerful because they improve how we work, reduce boilerplate code, and most importantly assure peace of mind.

In this article, we are going to see how simple it is to use [stdlib](https://stdlib.com/) and [deepstreamHub](https://deepstreamhub.com) to build a clock. The clock will retrieve time information from the server, then relay it to the browser for rendering.

## stdlib

stdlib is an implementation of the "serverless" concept which is best described through cloud functions. The term "serverless" does not mean that we don't need servers; it just means that we don't have to know the server details to get productive. In short, you get to write functions that are deployed and result to an API endpoint. You can read more about serverless technology [here](https://martinfowler.com/articles/serverless.html). 

### Setting Up stdlib
Now its time to set everything up using our favorite tool. (If you guessed **npm*** (Node Package Manager), you are correct.) Before that though, you need to [create an stdlib](https://stdlib.com/) account that links you to functions/services that you will need down the line.

Next, using npm, install the CLI tool for interacting with your account. The CLI tool simplifies creating, running, and deploying functions:

```bash
npm install lib.cli -g
```

You need to create a workspace. A workspace is a directory where a group of functions registered under a given user live. Creating a workspace is as simple as making a directory:

```bash
mkdir your-workspace-name
```

In your workspace, you can initialize stdlib by authenticating with the credentials you created while signing up:

```bash
lib init
```

Right now, your machine has been mapped to your account. You can then create a new service/function and hope to sync with your stdlib server:

```bash
lib create clock
```

This creates a directory named `clock` which contains a `main` folder. This folder is where your function lives. The `index.js` file contains a boilerplate function which you can run to see how stdlib serves your function:

```bash
lib http
```

The above command will serve your function locally at `localhost:8170` by default.

## deepstream for Realtime PubSub
stdlib is capable of converting your functions to API endpoints. This is great. Sometimes though, you would love to send some realtime data. With **deepstream**, realtime data transfer has never been easier. We will use deepstream's PubSub feature to emit realtime events from our stdlib function.

You will need to [create a free deepstreamHub](https://dashboard.deepstreamhub.com/signup) account which generates an API key to interact with the server.

First, let's install deepstream client on to our `clock` function:

```bash
npm install --save deepstream.io-client-js
```

Import deepstream t `main/index.js`:

```js
const ds = require('deepstream.io-client-js');
```

Create a deepstream instance with the deepstreamHub URL and login as anonymous to start communicating with the sever:

```js
const ds = require('deepstream.io-client-js');

module.exports = (params, callback) => {
  const client = ds('<APP-URL>').login()
  callback(null, 'hello world');
};
```

You can start emitting events via the object that the `client` variables stores. We are building a clock, therefore, the best way to emit events at intervals is to use the `setInterval` method:

```js
const ds = require('deepstream.io-client-js');
const moment = require('moment');

module.exports = (params, callback) => {
  const client = ds('<APP-URL>').login()
  setInterval(() => {
    client.event.emit('time', moment().format('MMMM Do YYYY, h:mm:ss a'))
  }, 1000)
  
  callback(null, 'hello world');
};
```

We've also imported **`moment`** to help us format time. An example format would look like the following:

```
May 1st 2017, 7:26:39 AM
```

## Subscribing via a Client

Right now, at the `/clock` route, our logic emits a realtime data via web socket. We can consume this realtime data from anywhere as long as we have the **connection URL** from deepstream. 

We can do this by importing deepstream from CDN (Content Delivery Network), logging in, and logging the emitted events to the console:

```html
<html>

  <body>
    
    <script src="https://code.deepstreamhub.com/js/2.x/deepstream.min.js"></script>
    <script>
	    var client = deepstream('APP-URL')
	
		client.login()
		
		client.event.subscribe('time', time => {
		  console.debug(time)
		})
    </script>
  </body>
</html>
```

Rather than emitting, we subscribe to the `time` event and log the payload whenever the event gets emitted:

![Image of console](http://imgur.com/SUDoTgP.jpg)

Rather than print in the console, we can display the time in the browser:

```js
var client = deepstream('APP-URL')

client.login()

var timeP = document.getElementById('time');
client.event.subscribe('time', time => {
  timeP.innerHTML = time
})
```

## Conclusion
Hopefully, you have a learned a better and faster way to build realtime apps. The clock example is basic, but, depending on your situation, the task can get pretty complex. Realtime data stores and serverless ideology have unimaginable powers that you can harness to make your applications more reliable and flat-out cooler.
____

Thanks for reading this guide. Feel free to leave comments and feedback in the discussion section below. As always, click the "Favorites" button if you enjoyed this guide!