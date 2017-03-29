Serverless architectures are a popular trend right now.

When thinking of serverless computing, names like Amazon AWS Lambda or Windows Azure Functions may come to mind, but the fact is that there are several companies focusing on this model. One of these companies is PubNub.

PubNub allows you to build real-time applications by providing APIs for publish/subscribe messaging, online presence detection, and mobile push notifications among other features.

Using PubNub, you can publish a message (data) into a channel (or channels) so the applications or devices subscribed to that channel receive the message in *real-time*:

![Image]()

That works great, and many times, that would be all you need for your real-time applications.

However, what if we need to process somehow the message before (or after) we send it?

We will have to add a server to be in charge of that processing:

![Image]()

For some applications, which need a server anyway, this may not represent a problem. But, wouldn't be great if someone else could take care of problems like management and scaling entirely?

This is the problem PubNub tries to solve with PubNub BLOCKS.

With BLOCKS (in uppercase refers to the PubNub feature), instead of simply passing messages, you can set up a piece of code (a block) to process a message before or after it's published:

![Image]()

This block is deployed as a serverless function (you can call it a microservice if you want) in the PubNub network. You just have to code it in JavaScript, hit deploy and PubNub will handle the rest.

Here are some things you can do with a block:
- Alter the message by adding a new attribute, or modifying or deleting an existing one.
- Call a web service.
- Store and retrieve data in a key-value store.
- Chain and forward messages between BLOCKS.

There's even a catalog of pre-built blocks that allows you to integrate the functionality of external APIs for things like notifications, geocoding, machine learning, and much more.

Sounds good?

Let's review the concepts and features of BLOCKS with two examples. First, let's create a block to reverse strings, and then, a more complex block that will take an URL and will call an API to generate a QR code to return its base64 representation, storing the result in a database to only call the API when necessary.

Let's get started!

