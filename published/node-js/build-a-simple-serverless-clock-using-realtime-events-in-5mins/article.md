Whenever we work in the arena of web development, we try to simplify our solutions as much as possible to make them more usable and easier to develop. "Serverless" concepts and realtime stores exemplify such simplification. These ideas improve how we work, reduce boilerplate code, and most importantly assure peace of mind to the developer.

This article will exhibit the simplicity of using [stdlib](https://stdlib.com/) and [deepstreamHub](https://deepstreamhub.com) to build a clock application. The clock will retrieve time information from the server, then relay it to the browser for rendering. 

## stdlib and "serverless"-ness

**stdlib** is an implementation of the "serverless" concept. As commonly mistaken, "serverless" does not mean "no servers;" it just means that we don't have to handle the server details and operations to be productive. Taking the "serverless" route, a developer can write functions that deploy and result in an API endpoint. You can read more about serverless technology on [its Wikipedia page](https://en.wikipedia.org/wiki/Serverless_computing)

### Setting Up stdlib

Now its time to set everything up using our favorite tool -- if you guessed **npm*** (Node Package Manager), you are correct. Before that though, you need to [create an stdlib](https://stdlib.com/) account that links you to functions/services that you will need down the line.

Once your account has been created, use npm to install the CLI (Command Line Interface) tool for interacting with your account. The CLI tool simplifies creating, running, and deploying functions:

```bash
npm install lib.cli -g
```

You need to create a workspace. A workspace is a directory in which a group of functions registered under a given user live. Creating a workspace is as simple as making a directory:

```bash
mkdir your-workspace-name
```

In your workspace, initialize stdlib by authenticating with the credentials you created while signing up:

```bash
lib init
```

Right now, your machine has been mapped to your account. You can then create a new service/function and hope to sync with your stdlib server:

```bash
lib create clock
```

This creates a directory named `clock` which contains a `main` folder that contains your function. The `index.js` file contains a boilerplate function that you can run to see an example of how stdlib works:

```bash
lib http
```

The above command will serve your function locally at `localhost:8170` by default.

## deepstream for Realtime PubSub
stdlib is capable of converting your functions to [API endpoints](https://dev.socrata.com/docs/endpoints.html). Although this feature is organizationally handy for users and developers alike, ["endpoints need to be static"](https://en.wikipedia.org/wiki/Web_API). Which means that stdlib's API endpoints are problematic if you need to send some realtime data. With **deepstream**, however, realtime data transfer has never been easier. We will use deepstream's PubSub feature to emit realtime events from our stdlib function. 

You will need to [create a free deepstreamHub](https://dashboard.deepstreamhub.com/signup) account which generates an API key to interact with the server.

First, let's install deepstream client on to our `clock` function:

```bash
npm install --save deepstream.io-client-js
```

Import deepstream to `main/index.js`:

```js
const ds = require('deepstream.io-client-js');
```

Create a deepstream instance with the deepstreamHub URL and log in as anonymous to start communicating with the sever:

```js
const ds = require('deepstream.io-client-js');

module.exports = (params, callback) => {
  const client = ds('<APP-URL>').login()
  callback(null, 'hello world');
};
```

You can start emitting events through the object represented by the `client` variable. We are building a clock, therefore, the best way to emit events at intervals is to use the `setInterval` method:

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

We can display the time in the browser instead of printing it to the console:

```js
var client = deepstream('APP-URL')

client.login()

var timeP = document.getElementById('time');
client.event.subscribe('time', time => {
  timeP.innerHTML = time
})
```

## Conclusion

Hopefully, you have a learned a quicker way to build realtime (and static) web apps. The clock example is basic, but should help you if you're dealing with a more complex project. Realtime data stores and serverless ideology have unimaginable powers that you can harness to make your applications more reliable and flat-out cooler.
____

Thanks for reading! Feel free to leave comments and feedback in the discussion section below. As always, click the "Favorites" button if you enjoyed this guide.