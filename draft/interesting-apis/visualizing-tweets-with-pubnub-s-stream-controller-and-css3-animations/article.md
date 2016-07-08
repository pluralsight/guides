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