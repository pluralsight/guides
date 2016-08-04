In this tutorial we're going to build a web application very similar to [Proof of Existence](https://proofofexistence.com) and [Origin Stamp](http://www.originstamp.org/) using [Node.js](https://nodejs.org/en/), [Tierion](https://tierion.com), [RethinkDB](http://rethinkdb.com/), [PubNub](https://www.pubnub.com), and [jsreport-core](http://jsreport.net/learn/nodejs).

In a few words, these sites allow you to store the hash of a file in Bitcoin's Blockchain so anyone can certify that the file existed at a specific time.

This has the following advantages:
- It demonstrates data ownership without revealing actual data.
- It checks for document integrity.
- Using the blockchain, you can certify the existence of the document without the need of a central authority. 
- In practice, this decentralized proof can't be erased or modified by anyone.
- It's anonymous, no one knows who you are or your data.

This [page has a better explanation about what is Proof of Existence](http://www.newsbtc.com/proof-of-existence/) and some sites dedicated to it.

Our application will have the following functionality:

You can upload a file by choosing it or dragging it to the marked area. Then, its SHA256 digest will be calculated so you can send it to the Blockchain using [Tierion Hash API](https://tierion.com/docs/hashapi):

![Create hash](https://raw.githubusercontent.com/pluralsight/guides/master/images/c785866d-69cf-4cce-8c93-675d907de3f8.gif)


Notice how the list of last documents registered is updated real-time, we'll be using [PubNub's Storage and Playback](https://www.pubnub.com/products/storage-and-playback/) functionality to implement that.

The hash of the file and the Blockchain receipt given by Tierion is stored in RethinkDB, a NoSQL database with real-time capabilities. When Tierion alerts us that a block of hashes has been processed, using [RethinkDB's changefeed](http://rethinkdb.com/docs/changefeeds/javascript/), we update the verification page (and the list of verified documents) in real-time:

![Confirming Hash](https://raw.githubusercontent.com/pluralsight/guides/master/images/6963ff66-7806-417c-9b58-063c23946054.gif)


Once the hash is anchored to the Blockchain, we can check it using sites like [Block Explorer](https://blockexplorer.com) or [Blockchain.info](https://blockchain.info) and get the receipt in PDF format using jsreport-core.

You can also verify that a file has been certified in the Blockchain by selecting the *Verify* option:

![Verify file](https://raw.githubusercontent.com/pluralsight/guides/master/images/b061a641-34f1-4af6-a0a2-bce474421557.gif)


The entire source code to follow along with this tutorial is on [Github](https://github.com/eh3rrera/blockchain-proof-existence).

However, before diving into the code, let's explain what is the Bitcoin's Blockchain (referred as the Blockchain).

# The Blockchain
[Wayne Vaughan](https://twitter.com/WayneVaughan), founder and CEO of Tierion, launched a challenge in the [2016 Consensus Hackathon](http://www.coindesk.com/events/consensus-2016/) consisting in answering the question *What is the Blockchain?*. The winner was [Lilia Vershinina](https://twitter.com/lilechkafleur):

> "The Blockchain is a trust layer for the Internet".

Interesting. Let's see if we can clarify this statement. By the way, you can see the whole story and other answers [here](https://medium.com/@WayneVaughan/blockchain-fever-at-the-consensus-2016-hackathon-d02a8476611d#.aettnnwlm).

The key idea of Bitcoin is decentralization. Rather than trusting in one party (a government or an institution), Bitcoin is a peer-to-peer system with no central point of failure. And in simple words, the blockchain is a database of transactions distributed across a global network of nodes.

The blockchain is a collection of all bitcoin transactions (represented as a chain of blocks linked together) since 2009 (the year it was born), from the genesis block (the first block) to the latest block.

When bitcoin transactions are broadcasted, *miners* pick them up locally to form a Bitcoin block. This block has to be verified by the miners by solving a complicated mathematical formula.

A [hash](https://en.wikipedia.org/wiki/Cryptographic_hash_function) is used to represent the verification of the block. This hash is computed on the block and the hash of the previous block, which forms a chain of hashes. You can see the concept of trust here if any transaction is modified, the hash, as well as the rest of the chain, becomes invalid.

For this reason, as the chain grows and more copies of it are kept by miners, the more difficult it becomes to modify the blockchain. Moreover, the increasing mining difficulty makes that blocks are generated about every ten minutes.

This is a (very) high-level overview of the blockchain. There are so many articles and books that talk about it, that is hard to recommend one, but if you want to know more, you can try [this search](https://www.google.com/search?q=what%20is%20the%20blockchain&rct=j).

With this knowledge, let's dive into the application.

# Requisites

### Tierion
We'll need a Tierion free account. You can signup [here](https://tierion.com/signup).

In addition to its Hash API, Tierion offers datastores to record any data in the blockchain. Depending on your plan, you can use a certain number of datastores and records per month. However, we'll be using the Hash API (which is used to record just hashes in the blockchain), which at the time of this writing is free to use with [some limitations](https://tierion.com/docs/hashapi).

When your account is ready, go to your Dashboard and in the *API* tab, copy your client secret, we'll need it later:

![Tierion API keys](https://raw.githubusercontent.com/pluralsight/guides/master/images/cb8c1f87-b18d-4f71-84dd-caa09a5bda17.png)

# RethinkDB

You'll need to [install RethinkDB server](http://rethinkdb.com/docs/install/). The easier way is to install it with one of the official packages.

This tutorial won't cover the basics of RethinkDB, but you can consult [RethinkDB's ten-minute guide](http://rethinkdb.com/docs/guide/javascript/) or [my getting started guide](http://tutorials.pluralsight.com/nosql-databases/a-practical-introduction-to-rethinkdb) if you haven't work with it.

### PubNub
Go to https://admin.pubnub.com/#/register and register a new account. When you log in, you will be presented with this screen:

![Pubnub dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/52106251-88ef-4d6b-ba51-ab30d54afd28.png)

You can either create a new app or use the demo app. When you click on this app, you'll something like the following:

![Pubnub dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/d5cf3593-e7b9-4992-951b-4c1df8edb9e3.png)

Save your publish and subscribe keys since we'll need it later. Now click on the keyset panel and go to the *Application add-ons* section to enable the *Storage and Playback* (choose the time period you want the messages to be stored) and *Stream Controller* add-ons. Save the changes:

![Pubnub addons](https://raw.githubusercontent.com/pluralsight/guides/master/images/030b2afc-c058-4eda-b413-62525178a585.png)

We'll be using the brand-new PubNub SDK for Javascript version 4, which is a little different than version 3. Here's the [Node.js documentation](https://www.pubnub.com/docs/nodejs/pubnub-javascript-sdk-v4) and the [Javascript (Web) documentation](https://www.pubnub.com/docs/javascript/pubnub-javascript-sdk-v4).

### Ngrok

Tierion will trigger a webhook (think of it as a callback) when a block of hashes had been anchored in the blockchain. This means an HTTP request will be made to our server, so we'll need to deploy our application on the cloud or keep it locally and use a service like [ngrok](https://ngrok.com/).

Ngrok proxies external requests to your local machine by creating a secure tunnel and giving you a public URL.

ngrok is a Go program, distributed as a single executable file (no additional dependencies). So for now, just download it from [https://ngrok.com/download](https://ngrok.com/download) and unzip the compressed file.


### Node.js

You'll also need Node.js and npm installed. You can download an installer for your platform [here](https://nodejs.org/en/download/).

Now that we have all we need, let's create the app.


# Setting up the project

Create a new directory and `cd` into it:
```
mkdir blockchain-proof && cd blockchain-proof
```

Now create a `package.json` file either with:
```
npm init
```

Or to accept the default values:
```
npm init -y
```

We will use the next dependencies:
- **hashapi-lib-node** (to use the Tierion Hash API)
- **rethinkdbdash** (to use RethinkDB)
- **pubnub** (to use the PubNub API)
- **jsreport-core** (to create the receipt)
- **jsreport-jsrender** (as the template library to create the receipt)
- **jsreport-wkhtmltopdf** (to create the receipt in PDF)
- **express** (as the web framework)
- **ejs** (as the template library)
- **body-parser** (to parse incoming request bodies)

Add them with the next command:
```
npm install --save hashapi-lib-node rethinkdbdash pubnub jsreport-core jsreport-jsrender jsreport-wkhtmltopdf express ejs body-parser
```

The project will have the following directory structure:
```
|— models
| |— receipt.js
| |— subscription.js
|— public
| |— css
| | |— index.css
| | |— verify.css
| |— img
| | |— cloud-upload.css
| |— js
| | |— index.js
| | |— pubnub-index.js
| | |— pubnub-verify.js
| | |— qrcode.min.js
|— routes
| |— app.js
|— templates
| |— receipt.html
|— views
| | |— index.ejs
| | |— verify.ejs
| |— config.js
| |— package.json
| |— server.js
```

The `config.js` file stores all the important configuration in this way (don't forget to use your own keys):
```javascript
module.exports = {
	tierion: {
		client_secret: 'XXXXXXXXXXXXXXXX',
		username: 'xxxxxx@xxxxxx.com',
		password: 'xxxxxxxxxx'
	},

    pubnub: {
        ssl           : false,  
        publish_key   : 'pub-c-xxxxx-xxx-xxxx-xxxx-xxxx',
        subscribe_key : 'sub-c-xxxxxx-xxxx-xxx-xxxx-xxx',
		registered_channel: 'registered_channel',
		confirmed_channel: 'confirmed_channel'
    },

	url: 'http://xxxxxxxx.ngrok.io',
	
	db: 'existence',
	
	port: process.env.APP_PORT || 3000
};
```

Start the RethinkDB server with this command:
```
rethinkdb
```

Go to the [Data Explorer](http://localhost:8080/#dataexplorer) section of your RethinkDB's dashboard and execute the following commands:
```
r.dbCreate('existence');
r.db('existence').tableCreate('subscription');
r.db('existence').table('subscription').insert({'id': 1, 'subscription-id': ''});
r.db('existence').tableCreate('receipt');
r.db('existence').table('receipt').indexCreate('timestamp');
```

They will create the database and tables of the application. You can leave the server running.

Now, in a new terminal window, navigate to the directory where you unzipped ngrok. 

We'll start ngrok by telling it which port we want to expose to the Internet. In our example, the port `3000`:
```
./ngrok http 3000
```

Alternatively, if you're on Windows:
```
ngrok http 3000
```

Now, you should see something like this:

![Ngrok Window](https://raw.githubusercontent.com/pluralsight/guides/master/images/f35ba341-9c64-4457-8bc7-ec78ebdcfd5e.png)

See that URL in the *Forwarding* row(s) with the `ngrok.io` domain? That's your public URL. Yours will be different, ngrok generates a random URL every time you run it.

Change the `url` property on the `config.js` file to that URL.

The only disadvantage is that this URL is not permanent. If you restart ngrok, it will give you another URL.

You can specify a subdomain, for example, to get the URL  `http://app.ngrok.io` use the command:
```
ngrok http -subdomain=app 3000
```

However, this requires a paid plan. You can get more info in this [page](https://ngrok.com/product).

Nevertheless, as long as you don't stop or restart ngrok, the URL won't change, so forget about that terminal window and let's leave it running for now.

Next, let's review the server-side code starting with the Tierion Hash API.



# Tierion Hash API

Tierion Hash API is simple. You only have to send your email and password to get a token that you must include in all requests. Next, you submit the token to get a receipt ID, which you use to get the [Blockchain Receipt](https://tierion.com/docs#blockchain-receipts) about ten minutes later.

With Block Subscription there's no need to check constantly for the receipt, a callback will alert you when a block of hashes has been processed and the corresponding receipts have been generated.

This is a [REST API](http://www.restapitutorial.com/), luckily, Tierion offers [a client library for Node.js](https://github.com/Tierion/hashapi-lib-node) that makes thing easier.

There are only two tricky parts when using this Block Subscription:
- To authenticate the Block Subscription callback we have to access the request raw body.
- Block Subscriptions are not automatically deleted after a certain number of failed requests, they live indefinitely, so we have to manage their creation and update on our side.

Let's start by creating the `server.js` file with a standard Express configuration with EJS as the template engine:
```javascript
var express = require('express');
var app = express();
var bodyParser = require('body-parser');
var path = require('path');
var config = require('./config');

var app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
	extended: true
}));

app.use(express.static(path.join(__dirname, 'public')));
app.set('views', path.join(__dirname, 'views'));

app.set('view engine', 'ejs');

app.listen(config.port, function() {
	console.log('Server up and listening on port %d', config.port);
});
```

To make the raw body available to in our request objects, we can configure `body-parser` like this:
```javascript
app.use(bodyParser.urlencoded({
    extended: true,
    verify: function(req,res,buf) { req.rawBody = buf.toString(); }
}));
```

This way, `rawBody` will contain the raw body of the request, for example:
```javascript
id=5796c41f8f5944de49bc0d1d&startTimestamp=2016-07-26T01%3A49%3A59.233Z&endTimestamp=2016-07-26T01%3A59%3A59.216Z
```

For the Block Subscriptions, we'll store the subscription in the `subscription` table so we can do something like this:
- Check if the table has a subscription ID (from previous requests).
- If there's an ID stored, try to get a reference to the subscription:
	- If the reference is invalid, create a new subscription and update the ID for future requests.
    - If the reference is valid, check if the URL where the callback request is received is still the same:
	    - If it's the same, we're done!
		- If it's different, update it.
- If there's no ID stored, create a new subscription and save the ID for future requests.

Let's translate this into code taking the following approach:
- First, we setup the Hash API client and the block subscription.
- On success, let's configure the routes of the application passing a reference to the Hash API client.
- Then, start the server.
- Finally, setup RethinkDB's changefeed to listen for changes in the status of the receipts.

Create the `models/subscription.js` file. This will contain the code for the Block Subscription set up. The first part is:
```javascript
var config = require('../config');
var r = require('rethinkdbdash')({
    db: config.db
});
var hashapi = require('hashapi-lib-node');

var SUBSCRIPTION_TABLE = 'subscription';

var URL = config.url + '/tierion';
```

It creates the variables required to do the work. For this project, instead of using the official RethinkDB driver for Node.js, we'll be using [rethinkdbdash](https://github.com/neumino/rethinkdbdash). The main advantage is that connections are managed by the driver with a connection pool so we don't need to call `r.connect`, or pass a connection to `run`.

Next, let's create a function to create a block subscription. It looks like this:
```javascript
var createSubscription = function(callback) {
    hashClient.createBlockSubscription(URL, function(err, subscription){
        console.log('Create subscription');
        if(err) {
            console.log(err);
        } else {
            r.table(SUBSCRIPTION_TABLE).get(1).update({'subscription-id': subscription.id}).run().then(function(updateResult) {
                callback(hashClient);
            });
        }
    });
}
```

The `createBlockSubscription` function configures a new Block Subscription callback to our server (with the `URL` variable). If it's created successfully, we update the table in our database with the returned ID of the subscription. When this operation is completed, a callback will be executed.

Now, the function to setup the block subscription:
```javascript
module.exports.setup = function (callback) {
    console.log("Setting up Block Subscription...");

    hashClient.authenticate(config.tierion.username, config.tierion.password, function(err, authToken){
        if(err) {
            console.log(err);
        } else {
            //console.log(authToken);

            r.table(SUBSCRIPTION_TABLE).get(1).run().then(function(result) {
                var id = result['subscription-id'];

                if(id === '') {
                    createSubscription(callback);
                } else {

                    hashClient.getBlockSubscription(id, function(errGet, result){
                        // The subscription doesn't exists
                        if(errGet) { 
                            console.log('Subscription doesnt exists');
                            createSubscription(callback);
                        }
                        // Let's check the callback URL hasn't change. If it has changed, update the URL
                        else {
                            if(result.callbackUrl === URL) {
                                callback(hashClient);
                            } else {
                                hashClient.updateBlockSubscription(id, URL, function(errUpdate, result){
                                    console.log('Update susbcription');
                                    if(errUpdate) {
                                        console.log(errUpdate);
                                    } else {
                                        callback(hashClient);
                                    }
                                });
                            }
                        } // else subscription exists
                    });

                } // else exists subscription ID
            });
            
        } // else authentication
    });
}
```

First, we authenticate to Tierion's API. Then, we get the ID of the subscription from the database. If there's none, we create a subscription. Otherwise, it's validated according to the plan we laid out before.

Now, if we update the last part of `server.js`, it should look like this:
```javascript
var subscription = require('./models/subscription');

subscription.setup(function(hashClient) {
    app.listen(config.port, function() {
        console.log('Server up and listening on port %d', config.port);
    });
});
```

But before starting the server, let's configure the routes of the application. This will the topic of the next section.

# The routes
Our application will have five routes:
- `/` will render the main page
- `/verify/:hash` will render the verification page for the hash passed as a parameter
- `/hash` to register a new hash
- `/tierion` will receive the block subscription callback (you already saw it in the previous section)
- `/pdf/:id` will render the receipt in PDF format

The code of these routes is simple, so maybe it's better to keep them all in one file. They'll use the Hash API client, so let's pass it as a parameter. Modify the last part of `server.js` in this way:
```javascript
var subscription = require('./models/subscription');

subscription.setup(function(hashClient) {
    var routes = require('./routes/app')(app, hashClient);

    app.listen(config.port, function() {
        console.log('Server up and listening on port %d', config.port);
    });
});
```

In the `routes/app.js` file, enter these definitions:
```javascript
var fs = require('fs');
var url = require('url');
var crypto = require('crypto');
var jsreport = require('jsreport-core')();
var model = require('../models/receipt');
var config = require('../config');

jsreport.use(require('jsreport-wkhtmltopdf')());
jsreport.use(require('jsreport-jsrender')());
jsreport.init();

var receiptTemplate = fs.readFileSync(__dirname + '/../templates/receipt.html', 'utf8');

var validateRequest = function(rawBody, providedSignature) {
    var hmac = crypto.createHmac('sha256', config.tierion.client_secret).update(rawBody, 'utf8');
    var calculatedSignature = hmac.digest('hex');

    return providedSignature == calculatedSignature;
}
```

First, we're requiring the modules we're going to use. We'll see the code of the `models/receipt.js` file in the next section.

Then, we load the modules used by jsreport to render a PDF ([jsreport-wkhtmltopdf](https://github.com/jsreport/jsreport-wkhtmltopdf)) and as template engine ([jsreport-jsrender](https://github.com/jsreport/jsreport-jsrender)) to initialize the library.

The following line will read (in a synchronous way) the HTML of the file that will serve as the template for the PDF generated by jsreport:
```javascript
var receiptTemplate = fs.readFileSync(__dirname + '/../templates/receipt.html', 'utf8');
```

Notice the use of `__dirname` to build the absolute path to the file. In Node.js you should be very careful with relative paths. The only place where you can safely use them is in require statements.

The next function will validate that the request received by Tierion's block subscription really comes from Tierion:
```javascript
var validateRequest = function(rawBody, providedSignature) {
    var hmac = crypto.createHmac('sha256', config.tierion.client_secret).update(rawBody, 'utf8');
    var calculatedSignature = hmac.digest('hex');

    return providedSignature == calculatedSignature;
}
```
Every callback includes a header containing the HMAC-SHA256 signature of the request. We have to calculate the signature using the raw body of the request and our account's Client Secret. If the calculated and the provided signatures match, we have a valid request.

Now the routes. The first one simply shows the main page of the site:
```javascript
module.exports = function (app, hashClient) {
    app.get('/', function (req, res) {
        res.render('index', { config: config });
    });
	
}
```

The next one will get the hash from the database through the model object, and it will render the verification page on success
```javascript
module.exports = function (app, hashClient) {

	...

	app.get('/verify/:hash', function (req, res) {
        model.getReceiptByHash(req.params.hash, function (success, result) {
            if(success) {
                res.render('verify', {
                    data: result,
                    config: config
                });
            } else {
                res.sendStatus(404); // Not found
            }
        });
    });
}
```

For the route to registering a new hash, we have to check if the hash the user is sending is already registered. If it's not the case, send it to Tierion and store a receipt object with the information we got at this point:
```javascript
module.exports = function (app, hashClient) {

	...

	app.post('/hash', function (req, res) {
        model.getReceiptByHash(req.body.hash, function (success, result) {
            if(success) { // Exists, redirect to verify
                res.json({
                    status: 'Exists'
                });
            } else { // Doesn't exist, send it to Tierion
                hashClient.submitHashItem(req.body.hash, function(err, result){
                    if(err) {
                        console.log(err);
                        res.json({
                            status: 'Error'
                        });
                    } else {
                        console.log(result);

                        var receipt = {
                            id: result.receiptId,
                            timestamp: result.timestamp,
                            sent_to_blockchain: false,
                            blockchain_receipt: null,
                            created_at: new Date(result.timestamp * 1000)
                                            .toISOString()
                                            .replace(/T/, ' ')
                                            .replace(/.000Z/, ' UTC'),
                            hash: req.body.hash
                        };

                        model.saveReceipt(receipt, function (success, result) {
                            if (success) res.json({
                                status: 'OK'
                            });
                            else res.json({
                                status: 'Error'
                            });
                        });
                    }
                });
            }
        });
    });
}
```

To keep things simple, the application will only support time in UTC. The timestamp returned by Tierion is a UNIX timestamp, which represents the number of seconds since January 1, 1970, 00:00:00 UTC (Unix epoch). Since the argument of the `Date` object in Javascript is the number of milliseconds since the Unix epoch, we have to multiply by 1000.

The next route, for the Tierion callback, uses the `validateRequest` function showed before and just mark the receipts as sent to the blockchain. This will trigger RethinkDB's changefeed to get the actual receipt objects and send the updates using Pubnub (more on this in the next section):
```javascript
module.exports = function (app, hashClient) {

	...

	app.post('/tierion', function(req, res) {
        var validRequest = validateRequest(req.rawBody, req.get('x-tierion-sig'));

        if(validateRequest) {
            model.markReceiptsAsSent(req.body.startTimestamp, req.body.endTimestamp);
        } else {
            return res.status(403).send('Request validation failed');
        }

        res.sendStatus(200); // OK
    });
}
```

Notice that we're using `rawBody` to authenticate the request and a signature provided by Tierion. A request from Tierion looks like this:
```javascript
{ 
	id: '5796c41f8f5944de49bc0d1d',
    merkleRoot: '21e31466867ab47e079421a7eeadaf5fae300a3c9eba8436ddab8b5cb90bf04b',
    transactionId: 'f42f4b074e2d46ab0f3883a907dac52b6bb373b97c1282f5b4111632df1d225f',
    startTimestamp: '2016-07-26T01:49:59.233Z',
    endTimestamp: '2016-07-26T01:59:59.216Z' 
}
```

Finally, the route to present the PDF receipt looks like this:
```javascript
module.exports = function (app, hashClient) {

	...

	app.get('/pdf/:id', function (req, res) {
       model.getReceipt(req.params.id, function(success, receipt) {
           if(success) {
               var requestUrl = url.format({
                    protocol: req.protocol,
                    host: req.get('host')
                });
                
                jsreport.render({ 
                    template: { 
                        content: receiptTemplate, 
                        engine: 'jsrender', 
                        recipe: 'wkhtmltopdf'
                        },
                        data: { 
                            r: receipt,
                            url: requestUrl
                        }
                }).then(function(obj) {
                    res.set(obj.headers);
                    res.send(obj.content);
                });
           } else {
               res.sendStatus(500);
           }
        });
    });
}
```

It gets the receipt object by its ID from the database and uses [jsreport-core](https://github.com/jsreport/jsreport-core) to generate the PDF file [from the HTML template with jsrender](http://jsreport.net/learn/jsrender).

In the next section, we'll review the receipt model.

# The core of the application

The core of the application is the `models/receipt.js` file. It contains the functions that interact with RethinkDB and PubNub to implement the functionality related to the blockchain receipts provided by Tierion.

The first part requires the modules used by the functions and creates the PubNub and RethinkDB objects:
```javascript
var Pubnub = require("pubnub");
var config = require('../config');

var pubnub = new Pubnub({
    ssl           : config.pubnub.ssl,  
    publishKey    : config.pubnub.publish_key,
    subscribeKey : config.pubnub.subscribe_key
});

var r = require('rethinkdbdash')({
    db: config.db
});

var RECEIPT_TABLE = 'receipt';
```

The first function sets up RethinkDB's changefeed, which will get the receipt object from Tierion, save it to the database, and publish the objects so the pages presented to the user can be updated in real-time:
```javascript
module.exports.setup = function (hashClient) {
    // Setup changefeed
    r.table(RECEIPT_TABLE).filter( r.and( r.row('sent_to_blockchain').eq(true), r.row('blockchain_receipt').eq(null) ) ).changes().run().then(function(cursor) {
        cursor.each(function(error, row) {
            if(row && row.new_val) {
                hashClient.getReceipt(row.new_val.id, function(err, result){
                    if(err) {
                        console.log(err);
                    } else {
                        var obj = JSON.parse(result.receipt);
                        // Save receipt to the database
                        r.table(RECEIPT_TABLE).get(row.new_val.id).update({'blockchain_receipt': obj}).run().then(function(updateResult) {
                            // Publish the object so the verification page is updated with the blockchain information
                            pubnub.publish({
                                    channel : row.new_val.hash,
                                    message : Object.assign({id: row.new_val.id}, obj),
                                    storeInHistory: false
                                },
                                function(status, response) {
                                    console.log(status, response);
                                }
                            );

                            // Publish the object so the list of last confirmed files on the main page is update
                            var formattedTimestamp = new Date(obj.header.timestamp * 1000)
                                                        .toISOString()
                                                        .replace(/T/, ' ')
                                                        .replace(/.000Z/, ' UTC');
                            pubnub.publish({
                                    channel : config.pubnub.confirmed_channel,
                                    message : {
                                        hash: row.new_val.hash,
                                        timestamp:formattedTimestamp
                                    },
                                }, 
                                function(status, response) {
                                    console.log(status, response);
                                }
                            );
                        }).error(function(error) {
                            console.log(error);
                        });
                    }
                });
            }
        });  
    });
}
```

If you're curious, this is how a receipt object from Tierion looks like:
```javascript
{  
   "receipt":{  
      "header":{  
         "chainpoint_version":"1.1",
         "hash_type":"SHA-256",
         "merkle_root":"9bace4b674c4adbfbf713cf263d105f3f3631bfe4cff43269eba8d3ac0474e97",
         "tx_id":"2c7e607de19e37bbb14d6a31a2ac77f473db7fa66228afcb26e114b7afb282b2",
         "timestamp":1469507399
      },
      "target":{  
         "target_hash":"789b2976442d94ed2c24c57e0ac1214572574e11e5f40102913e254e4dd752a1",
         "target_proof":[  
            {  
               "parent":"9bace4b674c4adbfbf713cf263d105f3f3631bfe4cff43269eba8d3ac0474e97",
               "left":"789b2976442d94ed2c24c57e0ac1214572574e11e5f40102913e254e4dd752a1",
               "right":"d030c83ea13049e271633efb45ae80bb084553e26e56867a2e91318276f0834c"
            }
         ]
      }
   }
}
```

Now that we have this function, let's update the last part of `server.js` so it can be called upon server initialization:
```javascript
var subscription = require('./models/subscription');
var receipt = require('./models/receipt');

subscription.setup(function(hashClient) {
    var routes = require('./routes/app')(app, hashClient);

    app.listen(config.port, function() {
        console.log('Server up and listening on port %d', config.port);
        receipt.setup(hashClient);
    });
});
```

Back to `receipt.js`, this changefeed is triggered when `sent_to_blockchain` is set to `true` (and the record doesn't have a `blockchain_receipt`). This is done by the `markReceiptsAsSent` function, called when Tierion's callback is received:
```javascript
module.exports.markReceiptsAsSent = function(startTimestamp, endTimestamp) {
    r.table(RECEIPT_TABLE).between(
            r.ISO8601(startTimestamp),
            r.ISO8601(endTimestamp), {index: 'timestamp'})
        .update({'sent_to_blockchain': true}).run();
}
```

Using [RethinkDB's time and date features](http://rethinkdb.com/docs/dates-and-times/javascript/), we mark all the records between the timestamps of processed hashes provided by Tierion.

To save a receipt object, we first clone the original object with the [Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) function, so we can safely change the timestamp property to use RethinkDB's time type, insert it into the database, and publish an update to the registered document channel (more on this in the next section):
```javascript
module.exports.saveReceipt = function (receipt, callback) {
    var formattedTimestamp = new Date(receipt.timestamp * 1000)
                                        .toISOString()
                                        .replace(/T/, ' ')
                                        .replace(/.000Z/, ' UTC')
    var clonedReceipt = Object.assign({}, receipt);
    clonedReceipt.timestamp = r.epochTime(receipt.timestamp);

    r.table(RECEIPT_TABLE).insert(clonedReceipt).run().then(function(results) {

        pubnub.publish({
                channel : config.pubnub.registered_channel,
                message : {
                    hash: receipt.hash,
                    timestamp:formattedTimestamp
                }
            }, 
            function(status, response) {
                console.log(status, response);
                callback(true, clonedReceipt);
            }
        );
        
    }).error(function(error) {
        console.log(error);
        callback(false, error);
    });
}
```

The other two methods on this file just get a receipt object by its hash and ID respectively:
```javascript
module.exports.getReceiptByHash = function(hash, callback) {
    r.table(RECEIPT_TABLE).filter(r.row('hash').eq(hash)).run().then(function(result) {
        if(result.length > 0) {
            callback(true, result[0]);
        } else {
            callback(false, result);
        }
    }).error(function(error) {
        console.log(error);
        callback(false, error);
    });
}

module.exports.getReceipt = function(id, callback) {
    r.table(RECEIPT_TABLE).get(id).run().then(function(result) {
        callback(true, result);
    }).error(function(error) {
        console.log(error);
        callback(false, error);
    });
}
```

Now let's review the code of the HTML pages.


# The index page
Here's the complete code for the index page template:
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Tierion/RethinkDB/Twilio Sync Demo</title>
	<meta name="viewport" content="width=device-width,initial-scale=1" />
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Roboto:300,300italic,400" />
	<link rel="stylesheet" href="css/index.css" />
</head>

<body>

<div class="container" role="main">

	<nav role="navigation">
		<a id="create-link" href="#" class="is-selected">Create</a>
		<a id="verify-link" href="#">Verify</a>
	</nav>

	<div id="drag-zone" class="box upload">

		
		<div class="box-input">
			<img class="icon" src="/img/cloud-upload.png" />
			<input type="file" name="files" id="file" class="file" />
			<label for="file"><strong>Choose a file</strong></label> or drag it here.
		</div>

		
		<div class="uploading">Reading&hellip;</div>
		<div class="success">
			Done!<br/>The hash of the file is:
			<span id="hash"></span>
			<a id="submit-link" href="#">Submit</a> or <a id="read-another-link" href="#">read another file</a>.
		</div>
		<div class="error">Error. Try again!</div>
	</div>



	<footer>
		<h2>Last documents registered:</h2>
		<table class="table" id="latest-registered">
			<tr>
				<th>File Hash</th><th>Timestamp</th>
			</tr>
		</table>
		
		
		<h2>Last documents confirmed:</h2>
		<table class="table" id="latest-confirmed">
			<tr>
				<th>File Hash</th><th>Timestamp</th>
			</tr>
		</table>
	</footer>

</div>

<script id="item-template" type="text/x-jsrender">
  <tr><td><a href="verify/{{:hash}}">{{:hash}}</a></td><td>{{:timestamp}}</td></tr>
</script>

<script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery.blockUI/2.70/jquery.blockUI.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jsrender/0.9.80/jsrender.min.js"></script>
<script src="https://cdn.pubnub.com/sdk/javascript/pubnub.4.0.0.js"></script>
<script>
	var REGISTERED_CHANNEL = '<%= config.pubnub.registered_channel %>';
	var CONFIRMED_CHANNEL = '<%= config.pubnub.confirmed_channel %>';

	var pubnubConfig = {
		subscribeKey: '<%= config.pubnub.subscribe_key %>'
	};
</script>
<script src="js/index.js"></script>
<script src="js/pubnub-index.js"></script>
</body>

</html>
```

It's a simple HTML page. Before the `<script>` tags, we define a template that contains the HTML to add to the lists of the latest registered and confirmed documents:
```html
<script id="item-template" type="text/x-jsrender">
  <tr><td><a href="verify/{{:hash}}">{{:hash}}</a></td><td>{{:timestamp}}</td></tr>
</script>
```

About the Javascript files, we're going to focus only on the essential parts. You can see the complete code for the Javascript (and CSS files) [on the Github repository for this application](https://github.com/eh3rrera/blockchain-proof-existence).

For the `public/js/index.js` file, the important part is where the hash of the file is generated. We're going to use the [FileReader](http://www.javascripture.com/FileReader) and [cryptographic](https://developer.mozilla.org/en-US/docs/Web/API/Crypto) libraries of Javascript.

First, we test if the browser supports them. If that's not the case, an alert is presented:
```javascript
if ( ! (window.File && window.FileReader && window.crypto && window.crypto.subtle && window.crypto.subtle.digest) ) {
	alert('No File API support. Please use the latest vesion of Firefox or Chrome.');
}
```

To read the file when it's dragged or chosen, we have the following code:
```javascript
dragZone.on( 'drop', function(evt)
{
	var files = evt.originalEvent.dataTransfer.files;

	handleFile(files);
});

inputFile.on('change', function(evt) {
	var files = evt.originalEvent.target.files;

	handleFile(files);
});

function handleFile(files) {
	if (!files.length) {
		return;
	}
	var file = files[0];
	
	var reader = new FileReader();
	reader.readAsArrayBuffer(file);
	
	reader.onload = function(e) {
		var data = e.target.result;
		
		...
	};
```

The limitation with this approach is that it won't work with large files (in the hundreds of MBs range, depending on your computer memory). A more robust implementation would read the file in blocks using workers (but that's a little complicated and out of the scope of this tutorial).

Then, to calculate the hash (as a SHA-256 digest), we [use the digest function](https://github.com/diafygi/webcrypto-examples/#sha-256---digest), which returns the hash as an [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer), and then we converted to its hexadecimal representation:
```javascript
reader.onload = function(e) {
	var data = e.target.result;

	window.crypto.subtle.digest({name: 'SHA-256'}, data).then(function(hash) {
		var hexString = '';
		var bytes = new Uint8Array(hash);

		for (var i = 0; i < bytes.length; i++) {
			var hex_i = bytes[i].toString(16);
			hexString += hex_i.length === 1 ? '0' + hex_i : hex_i;
		}

		$('#hash').text(hexString);
        calculatedHash = hexString;
		
		...
	}).catch(function(e) {
		showError(e);
	});
	};
```

The next section will cover how to use PubNub's Storage and Playback functionality to get the list of the latest registered and confirmed documents.


# PubNub's Storage and Playback and Multiplexing features

PubNub's Storage and Playback feature allows you to store messages so they can be retrieved at a later time.

In the `setup` and `save` functions of the receive module, we saw how you can publish to channels using version 4 of PubNub SDK.

Notice how you can indicate not to storage a message with `storeInHistory: false` (when the storage add-on is enabled, by default, all messages in all channels are stored).
`setup` function:
```javascript
...
pubnub.publish({
		channel : row.new_val.hash,
		message : Object.assign({id: row.new_val.id}, obj),
		storeInHistory: false
	},
	function(status, response) {
		console.log(status, response);
	}
);
...
pubnub.publish({
		channel : config.pubnub.confirmed_channel,
		message : {
			hash: row.new_val.hash,
			timestamp:formattedTimestamp
		},
	}, 
	function(status, response) {
		console.log(status, response);
	}
);
...
```

`save` function:
```javascript
...
pubnub.publish({
		channel : config.pubnub.registered_channel,
		message : {
			hash: receipt.hash,
			timestamp:formattedTimestamp
		}
	}, 
	function(status, response) {
		console.log(status, response);
		callback(true, clonedReceipt);
	}
);
...
```

This way, on the `public/js/pubnub-index.js` file, we create some variables and the PubNub object:
```javascript
$(document).ready(function() {
    var DOCS_TO_SHOW = 5;
    var listRegistered = '#latest-registered';
    var listConfirmed = '#latest-confirmed';
    var itemTemplate = $.templates('#item-template');

    var pubnub = new PubNub(pubnubConfig);
}
```

And using multiplexing, we subscribe to the channel for registered and confirmed documents:
```javascript
pubnub.addListener({
	message: function(data) {
		console.log(data);

		var el = listRegistered;
		if(data.subscribedChannel === CONFIRMED_CHANNEL) {
			el = listConfirmed;
		}

		$(itemTemplate.render(data.message)).hide().insertAfter($(el + ' tr:first-child')).fadeIn();

		var elements = $(el + ' tr').length - 1;
		if(elements > DOCS_TO_SHOW) {
			$(el + ' tr').last().remove();
		}
	}
});

pubnub.subscribe({
	channels: [REGISTERED_CHANNEL, CONFIRMED_CHANNEL] 
});
```

Multiplexing is the feature of listening to multiple data streams on a single connection. Notice than `addListener` must come before the `subscribe` function.

Inside `addListener`, we use the `subscribedChannel` property to know from which channel the message comes from. The actual message is in the `message` property.

This is for listening to new documents after the page is fully loaded. For the initial list when the page just loads, we call the `history` function on both channels to retrieve the last five new documents on the channels (using `reverse` and `count`):
```javascript
pubnub.history(
	{
		channel: REGISTERED_CHANNEL,
		reverse: false,
		count: DOCS_TO_SHOW, 
	},
	function (status, response) {
		console.log(response);

		response.messages.forEach(function(item) {
			$(listRegistered + ' tr:first-child').after(itemTemplate.render(item.entry));
		});
	}
);

pubnub.history(
	{
		channel: CONFIRMED_CHANNEL,
		reverse: false,
		count: DOCS_TO_SHOW, 
	},
	function (status, response) {
		console.log(response);

		response.messages.forEach(function(item) {
			$(listConfirmed + ' tr:first-child').after(itemTemplate.render(item.entry));
		});
	}
);
```

Next, it's the turn of the verification page.


# The verification page
The verification page is simple, also:
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="utf-8">
	<title>Tierion/RethinkDB/Twilio Sync Demo</title>
	<meta name="viewport" content="width=device-width,initial-scale=1" />
	<link rel="stylesheet" href="https://fonts.googleapis.com/css?family=Open+Sans:400,700" />
	<link rel="stylesheet" href="/css/verify.css" />
</head>
<body>
<div class="container">

	<div id="notice">
		<span class="title">The hash hasn't been submitted to <br/>the Blockchain yet.</span>
		
		<p>Keep this page open and wait 10 minutes at most, it will be updated automatically<br/>OR<br/> Bookmark it an check back later.</p>
	</div>
	
	<div id="info">
		<p>
			<span class="subtitle">Hash:</span>
			<small><%= data.hash %></small>
		</p>
		
		<p>
			<span class="subtitle">Created at:</span>
			<small><%= data.created_at %></small>
		</p>
        
	</div>

    <div id="links"></div>
</div>

<script id="message-template" type="text/x-jsrender">
  <span class="title">The hash was submitted to <br/>the Blockchain.</span>
</script>

<script id="blockchain-data-template" type="text/x-jsrender">
	<p>
        <span class="subtitle">Transaction ID:</span>
        <small>{{:tx_id}}</small>
    </p>

	<p>
        <span class="subtitle">Merkle tree:</span>
        <small>{{:merkle_root}}</small>
    </p>

    <p>
        <span class="subtitle">Submitted to the Blockchain at:</span>
        <small>{{:timestamp}}</small>
    </p>
</script>

<script id="links-template" type="text/x-jsrender">
	<div class="verify">
		<p>Verify the blockchain info at:<br/>
		<small><a href="https://blockexplorer.com/tx/{{:tx_id}}" target="_blank">https://blockexplorer.com/tx/{{:tx_id}}</a></small></p>
	</div>
	
	<div id="pdf-link">
		<p><a href="/pdf/{{:recepit_id}}" target="_blank">View PDF receipt</a></p>
	</div>
</script>

<script src="https://code.jquery.com/jquery-2.2.4.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jsrender/0.9.80/jsrender.min.js"></script>
<script src="https://cdn.pubnub.com/sdk/javascript/pubnub.4.0.0.js"></script>
<script src="/js/pubnub-verify.js"></script>
<script>
    $(document).ready(function() {

        var pubnubConfig = {
            subscribeKey: '<%= config.pubnub.subscribe_key %>'
        };

        var addBlockchainInfo = function(params1, params2, params3) {
            var notice = $('#notice');
            var info = $('#info');
            var links = $('#links');

            var template = $.templates('#message-template');
            notice.html(template.render(params1));

            template = $.templates('#blockchain-data-template');
            info.append(template.render(params2));

            template = $.templates('#links-template');
            links.append(template.render(params3));
        }

        <% if (data.sent_to_blockchain) { %>
        var params1 = {};

        var params2 = { 
            tx_id: '<%= data.blockchain_receipt.header.tx_id %>',
            merkle_root: '<%= data.blockchain_receipt.header.merkle_root %>',
            timestamp: <%= data.blockchain_receipt.header.timestamp %>
        };

        var params3 = { 
            tx_id: '<%= data.blockchain_receipt.header.tx_id %>',
            recepit_id: '<%= data.id %>'
        };

        addBlockchainInfo(params1, params2, params3);
        <% } else { %>

        setupPubNub(pubnubConfig, '<%= data.hash %>', addBlockchainInfo);

        <% } %>
    });
</script>

</body>

</html>
```

After the `.container` `div`, you can find the templates to add the blockchain information. If a user comes to this page after the receipt is generated, the information is added on page load. Otherwise, PubNub is set up with the function `setupPubNub` so it can listen for updates.

The code of this function can be found on `public/js/pubnub-verify.js`:
```javascript
var setupPubNub = function(pubnubConfig, hash, callback) {

     var pubnub = new PubNub(pubnubConfig);

     pubnub.addListener({
        message: function(data) {
            console.log(data);

            var params1 = {};

            var params2 = { 
                tx_id: data.message.header.tx_id,
                merkle_root: data.message.header.merkle_root,
                timestamp: data.message.header.timestamp
            };

            var params3 = { 
                tx_id: data.message.header.tx_id,
                recepit_id: data.message.id
            };

            var container = $('.container')
            
            container.css('opacity', '0')

            callback(params1, params2, params3);

            container.animate({
                opacity: 1
            }, 1500);
        }
    });

     pubnub.subscribe({
        channels: [hash],
    });

}
```

Here, we're just subscribing to the channel with name equals to the hash we're verifying to add the blockchain information when a message is received.

This way, the page of a verified hash looks like the following:

![Verification page](https://raw.githubusercontent.com/pluralsight/guides/master/images/23b03329-b17c-4e61-8d83-fa638eccf536.png)

And the receipt in PDF format:

![PDF receipt](https://raw.githubusercontent.com/pluralsight/guides/master/images/1c8c8326-37a8-4008-9e21-13b09098d9bd.png)

The template for the PDF receipt is similar to this page, the only significant difference is the QR code generated with [QRCode.js](https://davidshimjs.github.io/qrcodejs/) in this way:
```javascript
<div id="qrcode"></div>

<script src="js/qrcode.min.js" type="text/javascript"></script>
<script type="text/javascript">
var qrcode = new QRCode(document.getElementById("qrcode"), {
    text: "https://blockexplorer.com/tx/:{{r.blockchain_receipt.header.tx_id}}",
    width: 128,
    height: 128,
    colorDark : "#000000",
    colorLight : "#ffffff",
    correctLevel : QRCode.CorrectLevel.H
});
</script>
```

And that's it, the whole application. Remember that [the entire code is on Github](https://github.com/eh3rrera/blockchain-proof-existence) in case you want to review the files not covered here or if you missed something.

# But how do I verify the existence in the Blockchain?
I guess you may be thinking, *but what is this merkle_root, transaction id, and target proof that Tierion is returning? and how can all this act as a proof of existence?*.

Well, first of all, the file's SHA256 digest is unique. You need to have the exact same file to generate the same hash (try checking the hash of a file and then adding a space to see how the new hash is completely different). This proves that the file existed at least as early as the time the transaction was confirmed, otherwise, the hash would have been generated. 

About Tierion's receipt:
- The `target_hash` is the hash of the Record you wish to verify.
- This and other recorded hashes are combined into a [Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree).
- With the Merkle Tree, a `merkle_root` is calculated and inserted into the blockchain.

With a Merkle tree, you can prove that something belongs to a set, without having to store the whole set. 

To validate a receipt, you must confirm that the target hash is part of a Merkle Tree, and the tree's Merkle root has been published to a Bitcoin transaction.

If your `target_hash` was the only element in the Merkle Tree, it would be equal to `merkle_root`.

Otherwise, the SHA-256 hash of the `left` value concatenated to the `right` value of the `target_proof` should equal to the parent value, and we should continue moving down the list of nodes until the `parent` equals the `merkle_root`.

All this is better explained on this [Tierion whitepaper](https://tierion.com/chainpoint).

For example, to verify the hash of this receipt (e55bde19b226181fb630a2148b3969cac1c3fd6a44ebe464ca5ae6e408694227):
```javascript
{  
   "header":{  
      "chainpoint_version":"1.1",
      "hash_type":"SHA-256",
      "merkle_root":"9ddd8925d70371010b58797bbac404524039bdf0d5043308691e77656a44ed85",
      "timestamp":1470181200,
      "tx_id":"9cf0c93c28c1502cfb3e4bad9ea7fcce048455da2df619c95d097824de04a6d4"
   },
   "target":{  
      "target_hash":"e55bde19b226181fb630a2148b3969cac1c3fd6a44ebe464ca5ae6e408694227",
      "target_proof":[  
         {  
            "left":"e55bde19b226181fb630a2148b3969cac1c3fd6a44ebe464ca5ae6e408694227",
            "parent":"9ddd8925d70371010b58797bbac404524039bdf0d5043308691e77656a44ed85",
            "right":"55731848aa53bd31760f7c2fc15241f5099b460bfd5f645d3db24034522f4ce8"
         }
      ]
   }
}
```

We concatenate the `left` value with the `right` value of the first node of `target_proof`:
```
e55bde19b226181fb630a2148b3969cac1c3fd6a44ebe464ca5ae6e40869422755731848aa53bd31760f7c2fc15241f5099b460bfd5f645d3db24034522f4ce8
```

Then, you can use an [online SHA-256 calculator](http://www.xorbin.com/tools/sha256-hash-calculator) to verify that the hash of this string is equal to the `parent` value:
```
9ddd8925d70371010b58797bbac404524039bdf0d5043308691e77656a44ed85
```

Since this is the only node of `target_proof`, we can see that the above value is equal to the `merkle_root`. Otherwise, we'd have to repeat the process (the above value would be the `left` or `right` value of the next node).

Now that you've proven the value of the `merkle_root`, take the Bitcoin transaction ID (`tx_id`) to review the transaction on an online site like [https://blockchain.info/](https://blockchain.info/) or [https://blockexplorer.com/](https://blockexplorer.com/).

The transaction must include the `merkle_root` value in the `OP_RETURN` output. If this is true, the receipt is valid:

![Bitcoin Transaction](https://raw.githubusercontent.com/pluralsight/guides/master/images/b1162f85-5894-4df5-a59d-ea69ba65b114.png)

# Conclusion
As we have seen, using Tierion we can access the blockchain in an easy way. In addition, using RethinkDB and PubNub we can benefit from their real-time features to improve the user experience. This proves once again that combining the right APIs, you can build simple but powerful applications.

I hope you've enjoyed this tutorial, thanks for reading.
