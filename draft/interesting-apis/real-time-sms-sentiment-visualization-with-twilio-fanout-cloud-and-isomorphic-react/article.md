In this tutorial, we're going to build an application to visualize in real-time SMS messages sent to a Twilio phone number along with their sentiment analysis provided by the Marchex Sentiment Analysis plugin.

We'll be using Node.js with Express for the web server, Ngrok to expose our local server publicly, Fanout Cloud for the real-time functionality, and React for the view.

When a SMS is sent to the Twilio Number, information about the SMS (along with its sentiment analysis) is sent to our Node.js server.

Then, we extract the relevant information and publish it to a Fanout Cloud's channel. This is received by the clients in a web page that shows the information in this way:

![App demo](https://raw.githubusercontent.com/pluralsight/guides/master/images/17a85122-2db2-4e84-865e-7c58df8ed809.gif)


Notice how the background changes depending on the sentiment. Furthermore, there are transitions when new messages appear. The design was inspired by this [pen](http://codepen.io/bundleio/pen/regPeX).

The gauge comes from [Epoch](http://epochjs.github.io/epoch/), a real-time charting library that uses jQuery. Some would say React and jQuery should never be used together (because of the way React create the components), and we could have chosen another [gauge component with better React integration](http://michigan-com.github.io/react-gauge), but the truth is that React can play well with libraries like jQuery and sometimes, you have no option but to use a jQuery plugin in a React application. Besides, Epoch charts are very good looking.

Finally, with a couple of modifications, we'll make this application isomorphic (universal).

The source code of the final version of the application is available on [Github](https://github.com/eh3rrera/sms-sentiment).

Let's start by setting up everything that is required for this application.

# Requirements
### Twilio
We'll need [a trial account from Twilio](https://www.twilio.com/help/faq/twilio-basics/how-does-twilios-free-trial-work). Go to [https://www.twilio.com/try-twilio](https://www.twilio.com/try-twilio) to open one. 

When your personal phone number has been verified, you can get a Twilio phone number [here](https://www.twilio.com/console/phone-numbers/getting-started):

![Get phone number](https://raw.githubusercontent.com/pluralsight/guides/master/images/39d2308e-c05d-4a10-a3ad-63b581110d4b.png)


Make sure to choose a phone number with at least, SMS capabilities:

![Choose phone number](https://raw.githubusercontent.com/pluralsight/guides/master/images/b53d98e8-e422-408e-9af4-3d8db32f4b2f.png)


Now go to [Messaging - Programmable SMS - Add-ons](https://www.twilio.com/console/sms/add-ons):

![Add-ons](https://raw.githubusercontent.com/pluralsight/guides/master/images/eddfbbd2-397e-472e-ad2a-2b26765c3fc0.png)


Choose Marchex Sentiment Analysis for SMS:

![Choose add-on](https://raw.githubusercontent.com/pluralsight/guides/master/images/dfacd190-ad1c-40ab-ae03-92b01feb91d5.png)


The Marchex Sentiment Analysis add-on takes an SMS message and computes a sentiment score between `0.0` and `1.0` where `0.0` is most negative sentiment and `1.0` is the most positive sentiment:

![Marchex Sentiment Analysis add-on](https://raw.githubusercontent.com/pluralsight/guides/master/images/b45dcac5-5e84-4457-a684-5a43ccc9a5a5.png)


Press the *Install* button and agree to the terms of the service. You'll see the following screen as confirmation:

![Install add-on](https://raw.githubusercontent.com/pluralsight/guides/master/images/f615602c-cf12-43ed-b098-b32c282b3f64.png)


Check the *Use In Incoming SMS Message* option and press the *Save* button.

Finally, go to your [console](https://www.twilio.com/console) and copy your Auth Token, you may need it later:

![Twilio Console](https://raw.githubusercontent.com/pluralsight/guides/master/images/fbb92d2f-65e5-4d27-8d87-2d694e5ea08a.png)

### Ngrok
When an SMS is sent to our Twilio number, a webhook will be triggered (think of it as a callback). This means an HTTP request will be made to our server, so we'll need to deploy our application on the cloud or keep it locally and use a service like [ngrok](https://ngrok.com/).

Ngrok proxies external requests to your local machine by creating a secure tunnel and giving you a public URL.

ngrok is a Go program, distributed as a single executable file (no additional dependencies). So for now, just download it from [https://ngrok.com/download](https://ngrok.com/download) and unzip the compressed file.


### Fanout Cloud

We'll need a Fanout account, [sign up here](https://fanout.io/account/signup/).

When the account is activated, go to your dashboard:

![Fanout Dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/8bcde085-ddf6-4d11-b1c5-eede0fa00b84.png)


Copy your Realm ID and Realm Key since we'll need them later.

### Node.js

You'll also need version 4.4 or higher of Node.js and npm installed. You can download an installer for your platform [here](https://nodejs.org/en/download/).


Now that we have all we need, let's create the app.


# Setting up the application

Create a new directory and `cd` into it:
```
mkdir sms-sentiment && cd sms-sentiment
```

We're going to use ECMAScript 2015 so let's set up [Babel](https://babeljs.io/) to transform this syntax to one most browsers can understand by creating the configuration file `.babelrc`:
```
echo '{ "presets": ["react", "es2015", "stage-0"] }' > .babelrc
```

Babel 6.x does not ship with any transformations enabled, so you need to explicitly tell it what transformations to run by using a preset.

The first two are self-descriptive. The stage-x presets are changes to the language that haven’t been approved to be part of a release of Javascript.

The [TC39](https://github.com/tc39) categorizes proposals into 4 stages:

- stage-0 - Strawman: just an idea, possible Babel plugin. 
- stage-1 - Proposal: this is worth working on.
- stage-2 - Draft: initial spec.
- stage-3 - Candidate: complete spec and initial browser implementations.
- stage-4 - Finished: will be added to the next yearly release.

`stage-0` includes all plugins from presets of all levels. `stage-1` includes all plugins from presets 1 to 4 and so on.

To execute Babel and bundle our scripts with their dependencies, we'll use [Webpack](https://webpack.github.io) and npm. Let's install Webpack (globally) with:
```
npm install -g webpack
```

Don't forget to add `sudo` if you're working on Linux.

Then, add a `package.json` configuration file with:
```
npm init
```

Alternatively, if you want to accept all the defaults:
```
npm init -y
```

Next, install the Babel dependencies and presets (and a polyfill to emulate a full ES2015 environment) with:
```
npm install --save-dev babel-core babel-loader
npm install --save-dev babel-preset-es2015 babel-preset-react babel-preset-stage-0 babel-polyfill
```

Do the same with Webpack:
```
npm install --save-dev webpack
```

We're using `--save-dev` because these dependencies are only required to develop the application, so they can be listed under the `devDependencies` section of `package.json`. 

For running the application, we'll need Express, React, Twilio, Fanout, and other dependencies, let's add them with:
```
npm install --save express ejs body-parser path react react-dom twilio fanoutpub faye
```

Now, create a `webpack.config.js` file with the following content:
```javascript
var path = require('path');

module.exports = {
    entry: ['./src/app.js'],

    output: {
        filename: 'public/js/bundle.js',
        sourceMapFilename: 'public/js/bundle.map'
    },

    devtool: '#source-map',
    
    module: {
        loaders: [
            {
                loader: 'babel',
                exclude: /node_modules/
            }
        ]
    }
}
```

This way, Webpack will create a `public/js/bundle.js` file with all the JavaScript code of the application (from the script `/src/app.js`).

Next, create a `config.js` file with the following content:
```javascript
module.exports = {
	twilio: {
	    validate   :  false
    },

    fanout: {
        realm_id   : '<YOUR_FANOUT_REALM_ID>',
        realm_key  : '<YOUR_FANOUT_REALM_KEY>'
    },

    channel: 'sms',

	port: process.env.APP_PORT || 3000
}
```

Finally, add to `package.json` the following `start` script that packs our application and starts the Node.js server:
```javascript
{
  ...
  "scripts": {
    "start": "webpack && node server.js"
  },
  ...
}
```

We'll review the code of `server.js` in the next section.

# Setting up Express

Let's start by setting up a simple Express web application. 

Create the `server.js` file and write the following `require` statements:
```javascript
var express = require('express');
var ejs = require('ejs');
var bodyParser = require('body-parser');
var path = require('path');
var config = require('./config');
```

Now create the express object:
```javascript
var app = express();
```

Add the configuration to capture both the body and URL parameters:
```javascript
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
```

And the configuration of the public directory (to server CSS files, for example):
```javascript
app.use(express.static(path.join(__dirname, 'public')));
```

We'll be using [EJS](http://ejs.co/) as the template library, so let's define `views` as the directory to store our templates and configure Express to use EJS as the view engine:
```javascript
app.set('views', path.join(__dirname, 'views'));
app.set('view engine', 'ejs');
```

Next, let's add the following code to respond to the `/` route with an `index` template:
```javascript
app.get('/', function (req, res) {
	res.render('index', {});
});
```

And create the corresponding `views/index.ejs` file with some content, for example:
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>SMS Sentiment</title>
</head>
<body>
    <h1>Hello</h1>
</body>
</html>
```

Back to the `server.js` file, add the code to start the server:
```javascript
app.listen(config.port, function() {
    console.log('Server up and listening on port %d', config.port);
});
```

If you execute the command `node server.js` and go to `http://localhost:3000` (or whatever your host and port are), you should see something like this:

![Hello World](https://raw.githubusercontent.com/pluralsight/guides/master/images/3ed08101-23fc-4e11-9793-0b7ea1a8794d.png)


In the next section, we'll expose our server to the Internet with ngrok.



# Setting up Ngrok
In a new terminal window, navigate to the directory where you unzipped ngrok. 

We'll start ngrok by telling it which port we want to expose to the Internet, in our example, the port `3000`:
```
./ngrok http 3000
```

Alternatively, if you're on Windows:
```
ngrok http 3000
```

Now, you should see something like this:

![Ngrok Window](https://raw.githubusercontent.com/pluralsight/guides/master/images/d58ac65f-7849-498b-9beb-e6920a65802b.png)

See that URL in the *Forwarding* row(s) with the `ngrok.io` domain? That's your public URL. Your's will be different, ngrok generates a random URL every time you run it.

If you open in a browser `http://[YOUR_GENERATED_SUBDOMAIN].ngrok.io`, you should see the same page found on `http://localhost:3000`:

![Hello World with Ngrok](https://raw.githubusercontent.com/pluralsight/guides/master/images/37017d14-e65e-4cee-8468-d4f4fa31e40b.png)


Open this ngrok URL in another computer if you want. Once again, you should see the same page. Our local server is now available publicly.

The only disadvantage is that this URL is not permanent. If you restart ngrok, it will give you another URL.

You can specify a subdomain, for example, to get the URL  `http://sms.ngrok.io` use the command:
```
ngrok http -subdomain=sms 3000
```

However, this requires a paid plan. You can get more info in this [page](https://ngrok.com/product).

Nevertheless, as long as you don't stop or restart ngrok, the URL won't change, so forget about that terminal window and let's leave it running for now.

In the next section, we'll review the Twilio API for receiving/sending SMS messages.


# Receiving SMS messages

Let's configure our Twilio number to communicate to our server when it receives an SMS message.

Go to your [Phone Numbers Console](https://www.twilio.com/console/phone-numbers/incoming) and select your phone number. The following screen will be shown:

![Phone Numbers console](https://raw.githubusercontent.com/pluralsight/guides/master/images/c731fc4b-86f0-4e10-8e7d-163428a66c3c.png)

Let's say that Twilio will call the `/sms` route in our server, so enter your ngrok URL in the corresponding field of the *Messaging* section:

![Twilio Phone Number Webhook](https://raw.githubusercontent.com/pluralsight/guides/master/images/6a8aaddd-59b4-4061-ad99-bccfcb561afc.png)


Also, make sure the HTTP request type is set to *POST* and save your changes.

Now, we'll add the code to receive the message on the `/sms` route.

We'll be using the Twilio API for Node.js, which is documented on [http://twilio.github.io/twilio-node/](http://twilio.github.io/twilio-node/).

At the beginning of the `server.js` file, import the Twilio module:
```javascript
var Twilio = require('twilio');
```
And before (or after) the definition of the `/` route, add the following code:
```javascript
app.post('/sms', Twilio.webhook(config.twilio),function(req, res) {

});
```

This will define the `/sms` POST route and the [Twilio.webhook](http://twilio.github.io/twilio-node/#webhook) function, which acts as a [Express middleware function](http://expressjs.com/en/guide/using-middleware.html) that determines if the request was sent by Twilio, and makes the Express response object aware of TwimlResponse objects.

However, the `config.twilio` object we specify defines the `validate` option as `false`, which disables this validation. If the validation is enabled (which you'll need to do in a production application if you don't want to receive requests from anyone), this function will look to the environment variable TWILIO_AUTH_TOKEN for your Twilio auth token to validate the request. 

When Twilio sends an HTTP request to your server, it includes a value in the header that is signed with your auth token.

So, if you enable this option, make sure to export this variable before running the server. On Linux, for example, do it this way:
```
export TWILIO_AUTH_TOKEN=xxx0xxx00xx0xxxxxxx0xxxx0xxxx00xxx0
```

Moving on, when an SMS is received on your Twilio number, your server will get an object like the following:
```
{
 ToCountry: 'US',
  ToState: 'NJ',
  SmsMessageSid: 'SMc1cc1b8b81b28dceb80b42a925d37777',
  NumMedia: '0',
  ToCity: 'NEWTON',
  FromZip: '',
  SmsSid: 'SMc1cc1b8b81b28dceb80b42a925d37777',
  FromState: 'Df.',
  SmsStatus: 'received',
  FromCity: 'Mexico',
  Body: 'This is a test',
  FromCountry: 'MX',
  To: '+1201555555',
  ToZip: '07860',
  AddOns: '{
				"status":"successful",
				"message":null,
				"code":null,
				"results":{
					"marchex_sentiment":{
						"request_sid":"XR8b665c4741eabb7c3382d814be2d225r",
						"status":"successful",
						"message":null,
						"code":null,
						"result":{
							"result":0.1545030027627945
						}
					}
				}
			}',
  NumSegments: '1',
  MessageSid: 'SMc7cc1b8b81b28dceb80b42a925d37089',
  AccountSid: 'AC1127a09cf75ed2265926304233f3a2e7',
  From: '+52555555555',
  ApiVersion: '2010-04-01'
}
```

As you can see, there's a lot of information on this object. For this application, we'll only use the text of the message, the number, and the country from which it was sent, and the sentiment result.

So let's extract this information with the following code:
```javascript
app.post('/sms', Twilio.webhook(config.twilio),function(req, res) {
	var  addOn = JSON.parse(req.body.AddOns);

	// Create a data object with the properties you want to send
    var msg = {
        text: req.body.Body,
        from: req.body.From,
        sentiment: addOn.results.marchex_sentiment.result.result,
        country: req.body.FromCountry
    };
});
```

Additionally, let's respond to this message to acknowledge it was received. To do this, we'll need to send back a [TwiML](https://www.twilio.com/docs/api/twiml) response:
```javascript
app.post('/sms', Twilio.webhook(config.twilio),function(req, res) {
    var  addOn = JSON.parse(req.body.AddOns);

	// Create a data object with the properties you want to send
    var msg = {
        text: req.body.Body,
        from: req.body.From,
        sentiment: addOn.results.marchex_sentiment.result.result,
        country: req.body.FromCountry
    };

    // Create a TwiML response
    var twiml = new Twilio.TwimlResponse();
    twiml.message('Message received');
    
    // Send the TwiML response as XML
    res.send(twiml.toString());
});
```

If we restart the server with these changes and then send an SMS message to our Twilio number, we should receive an SMS with the following text:
```
Sent from your Twilio trial account:
Message received
```

Ngrok also keeps a log of all the traffic going through it. Go to http://localhost:4040 and you should see something like this:

![Ngrok Request](https://raw.githubusercontent.com/pluralsight/guides/master/images/8e3d1e58-564c-4171-8bb9-1031dbd99f7b.png)

You can even replay a request with the *Replay* button in the right top of the window when you select one request from the list on the left.

Also, at the bottom of the same window, you can see the TwiML response:
```xml
<?xml version="1.0" encoding="UTF-8"?><Response><Message>Message received</Message></Response>
```

And if you select your Twilio number on your [Phone Numbers Console](https://www.twilio.com/console/phone-numbers/incoming) and then select the *View Messages Inbound* option, you'll see the message you sent:

![Twilio incoming SMS messages](https://raw.githubusercontent.com/pluralsight/guides/master/images/ab494749-6399-481e-987d-eb6dc5f900ba.png)


The same applies to the messages sent by Twilio with the *View Messages Outbound* option.

Now let's talk about Fanout Cloud.

# Publishing a message with Fanout Cloud