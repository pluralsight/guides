This tutorial will show you how to use channel groups with PubNub's Stream Controller API by building a web app to visualize tweets with certain hashtags in real-time. 

This is the stack we're going to use:
- **Node.js** as the server-side language,
- **Express** as a web framework,
- **Handlebars** as the template library,
- **Twitter Streaming API** to get the tweets in real-time,
- **PubNub Stream Controller** with channel groups to control what tweets to show,
- **jQuery** for client-side interaction, and
- **CSS3 animations** to visualize the tweets.

The app will track tweets with the following six hashtags (they're configurable):
- **#news**
- **#family**
- **#quote**
- **#love**
- **#today**
- **#programming**

Each hashtag will have its own PubNub channel (where the tweets will be published), and the six channels will belong to a channel group.

The client will listen to messages on this channel group to present tweets as pulsating circles of different colors (each tag has it own color) like this:

![Main page](https://raw.githubusercontent.com/pluralsight/guides/master/images/9fcb4ad8-1a47-4261-a05e-8ef846c5832d.gif)

There's also an admin page that will "turn-on/turn-off" the listening of hashtags by adding/removing channels to the PubNub channel group:

![Admin page](https://raw.githubusercontent.com/pluralsight/guides/master/images/1cbaa490-f36f-47d2-8fe8-b5d95132a81f.png)


Changes made on the admin page are reflected on the client(s) in real time:

![Real-time changes](https://raw.githubusercontent.com/pluralsight/guides/master/images/2c832eda-394f-46e5-9f14-b2442ef884c1.gif)


For simplicity, the admin page (and their server-side routes) won't implement authentication.

The entire source code [is available on Github](https://github.com/eh3rrera/visual-tweets). 

# Requirements

### Twitter
Log in to your Twitter account and go to https://dev.twitter.com/apps/new to create a new application (you must add your mobile phone to your Twitter profile before creating an application. Please read https://support.twitter.com/articles/110250-adding-your-mobile-number-to-your-account-via-web for more information).

Enter the following information:
- **Name** (your application name, for example, *sample_eh3rrera*)
- **Description** (your application description)
- **Website** (your application's publicly accessible home page where users can go to  use it. We're not going to use it, so you can just enter *http://127.0.0.1*).

Agree to Twitter's developer agreement and create your application:

![Creating Twitter app](https://raw.githubusercontent.com/pluralsight/guides/master/images/a7e48adb-e0ad-48ef-8c74-84df27527c57.png)


Now go to the *Keys and Access Token* section and create you access token by clicking on the *Create my access token*:

![Twitter keys and tokens](https://raw.githubusercontent.com/pluralsight/guides/master/images/cf78b26f-d185-468e-8fe4-ff1e0edf386d.png)


Now save your consumer key, consumer secret, access token, and access token secret since we'll need it later.

### PubNub
Go to https://admin.pubnub.com/#/register and register a new account. When you log into your account, it should look like this:

![Pubnub dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/ec80305c-c419-45bd-b73b-f3ba81524ef4.png)


You can either create a new app or use the demo app already created. When you click on this app, you'll something like the following:

![Pubnub dashboard](https://raw.githubusercontent.com/pluralsight/guides/master/images/d6c81f7c-1d5b-4938-9d86-96e31c707a5b.png)


Save your publish and subscribe keys since we'll need it later. Now click on the keyset panel and go to the *Application add-ons* section, enable the *Stream Controller*, add-on and save the changes:

![Pubnub addons](https://raw.githubusercontent.com/pluralsight/guides/master/images/29501408-f15b-45ac-8ca6-7c7f279c1dd9.png)


### Node.js

We'll need to have the latest version of [Node.js](https://nodejs.org/en/download/) installed.


Now that we have all we'll need, let's get started.