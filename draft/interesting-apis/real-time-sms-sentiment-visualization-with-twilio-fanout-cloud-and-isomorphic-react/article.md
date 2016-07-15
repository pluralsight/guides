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
