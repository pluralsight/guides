# Build a multiplayer VR app that works in realtime - in less than 10 minutes.

#### No prior knowledge of VR necessary!

Okay so I'm done making the title sound fancy, now let's build a fancy VR app that will serve as your beginning step for building Virtual Reality apps which also work in realtime.




I made this app for a talk at [Front-end connect](http://frontend-con.io/) conference but I later realized that it would serve as a great getting-started guide as well. So here we go!

#### What will we build?

<img src="https://raw.githubusercontent.com/pluralsight/guides/master/images/aed2ea6f-915d-40a5-9f99-cdd457572b29.jpg-large" width="750" height="500" />

By the end of this tutorial, you'll have an app similar to the one seen in the above picture. It will have a basic VR scene where multiple users can connect to your app from their mobile phones by simply hitting a URL on their mobile browsers. We'll use only open sourced frameworks for implementing this application, thus at no point you have to pay.

For every user who joins your application, a new avatar will appear in your VR scene which will rotate/move in realtime according to the movement of the user's phone in real life.

You might wanna go through a basic A-Frame tutorial to understand it better since this tutorial will skip over a few basic details.

#### What will we use?

We'll use [WebVR](https://webvr.info/), which allows us to build Virtual Reality applications that are accessible directly on the web, thus completely eliminating heavy downloads and installs.

<img src="https://raw.githubusercontent.com/pluralsight/guides/master/images/24413ad4-4f09-4b66-a592-689bdc49c428.png" width="350" height="350" />

However, building applications directly in WebVR is a bit complex since it requires knowledge of WebGL, etc. Hence, to make it simpler for web developers to be able to build VR apps, Mozilla's VR team built a framework on top of WebVR, called [A-Frame](aframe.io).

<img src="https://raw.githubusercontent.com/pluralsight/guides/master/images/78806784-3ab1-46d1-8fe9-b4020821c78b.png" width="400" height="250" />

Next, we'll use [deepstream](https://deepstreamhub.com/) to implement all the realtime functionality. It provides an open realtime server that provides features like PubSub and data-sync. Out of it's five major features - records, RPCs, events, presence and listening- we shall extensively make use presence and records in this tutorial. More on this further down the tutorial.

![deepstream logo](https://raw.githubusercontent.com/pluralsight/guides/master/images/b72cd656-c2f8-488b-b8c6-4916e69a3f91.png)

#### Getting started

So, let's start by building the basic VR setup for the application. We'll do this in a HTML file and make use of A-Frame's entity-component system. 

In your HTML file, add the following: 

```html
<html>
<head>
<script src="https://cdnjs.cloudflare.com/ajax/libs/deepstream.io-client-js/2.1.1/deepstream.js"></script>
<script src="https://aframe.io/releases/0.5.0/aframe.min.js"></script>
<script type="text/javascript" src="main.js"></script>
<script src="https://unpkg.com/aframe-text-geometry-component@0.5.1/dist/aframe-text-geometry-component.min.js"></script>
</head>
<body>
<a-scene id="scene">
.
.
.
.
</a-scene>
</body>
</html>
```

Let's understand the above snippet.

We have added references to the following:

1. deepstream's Javascript client
2. A-Frame source file
3. Our local JS file which we'll use next to add the logic into the app
4. A-frame's community contributed - text component to conveniently add text to our VR scene.

All the objects we wish to include in our VR scene must go within the `a-scene` tag in our HTML file, as scene above (pun intended)!



