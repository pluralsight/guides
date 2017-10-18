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

The references refer to the following:

1. deepstream's Javascript client
2. A-Frame source file
3. Our local JS file which we'll use next to add the logic into the app
4. A-frame's community contributed - text component to conveniently add text to our VR scene.

All the objects we wish to include in our VR scene must go within the `a-scene` tag in our HTML file, as scene above (pun intended)!

Next, let's add all the assets we'll use within the `a-assets` tag. Adding all the resources under this tag makes sure that all your assets are pre-loaded before your app shows up, thus preventing a shitty first look due to slow loading of resources.

```html
  <!--assets at one place for better loading-->
  <a-assets> 
    <img id="pink" src="https://img.gs/bbdkhfbzkk/stretch/http://i.imgur.com/1hyyIUi.jpg" crossorigin="anonymous" />
    <img src="https://img.gs/bbdkhfbzkk/stretch/https://i.imgur.com/25P1geh.png" id="grid" crossorigin="anonymous">
    <img src="https://img.gs/bbdkhfbzkk/2048x1024,stretch/http://i.imgur.com/WMNH2OF.jpg" id="chrome" crossorigin="anonymous">
    <img id="sky" src="https://img.gs/bbdkhfbzkk/2048x2048,stretch/http://i.imgur.com/WqlqEkq.jpg" crossorigin="anonymous" />
    
    <a-asset-item id="dawningFont" src="https://cdn.glitch.com/c719c986-c0c5-48b8-967c-3cd8b8aa17f3%2FdawningOfANewDayRegular.typeface.json?1490305922844"></a-asset-item>
    <a-asset-item id="exoFont" src="https://cdn.glitch.com/c719c986-c0c5-48b8-967c-3cd8b8aa17f3%2Fexo2Black.typeface.json?1490305922150"></a-asset-item>
    <a-asset-item id="exoItalicFont" src="https://cdn.glitch.com/c719c986-c0c5-48b8-967c-3cd8b8aa17f3%2Fexo2BlackItalic.typeface.json?1490305922725"></a-asset-item>
    
    <a-mixin id="eye" geometry="primitive: sphere; radius: 0.2" material="shader: flat; side: double; color: #FFF"></a-mixin>
    <a-mixin id="pupil" geometry="primitive: sphere; radius: 0.05" material="shader: flat; side: double; color: #222"></a-mixin>
    <a-mixin id="arm" geometry="primitive: box; depth: 0.2; height: 1.5; width: 0.2" material="color: #222; shader: flat"></a-mixin>
  </a-assets>
```
Feel free to use your own resources to give the app a customized feel.

There are two new things in the above code snippet, let's crack them down:

`a-asset-item` - invokes the three.js file loader. You can use this to load all file types.

`a-mixin` - is a very useful tag which allows code reuse by allowing you to specify a set of properties to be applied to a single entity. You can give it an id and reference it multiple times as we'll see. We will have three mixins, each specifying certain attributes of the avatar that we intend to create- the eye, pupil and arms.

Now, let's add all the static visual elements in our VR scene.

```html
  <!--adds the torusKont-->
  <a-entity scale="2 2 2" geometry="primitive: torusKnot" position="0 6 -10" material="color: magenta; metalness:1; roughness: 0.1; sphericalEnvMap: #sky;">
        <a-animation easing="linear" attribute="rotation" dur="10000" to="0 0 360" repeat="indefinite"></a-animation>
  </a-entity>

  <!--adds the text with custom font-->
  <a-entity position="-3 1 -6" rotation="5 0 0">
    <a-entity
          rotation="0 0 5"
          position="0 2 0.2"
          text-geometry="value: Frontend-Connect; font: #dawningFont; bevelEnabled: true; bevelSize: 0.05; bevelThickness: 0.05; curveSegments: 12; size: 1; height: 0;"
          material="color:lavenderblush; metalness:1; roughness: 0; sphericalEnvMap: #pink;">
    </a-entity>

    <!--adds the text with custom font-->
    <a-entity position="-0.5 0.5 -0.5" scale="0.6 1.2 1" text-geometry="value: VR on the web; font: #exoFont; bevelEnabled: true; bevelSize: 0.1; bevelThickness: 0.1; curveSegments: 1; size: 1.5; height: 0.5;" material="color:pink; metalness:0.9; roughness: 0.05; sphericalEnvMap: #chrome;"></a-entity>

    <!--adds the text with custom font-->
    <a-entity position="1 0 0.3" text-geometry="value: 2017; font: #exoItalicFont; style: italic; size: 0.8; weight: bold; height: 0;"
                  material="shader: flat; color: white">
    </a-entity>
    <!--adds the text with custom font-->
    <a-entity position="1 0 0.3" text-geometry="value: 2017; font: #exoItalicFont; style: italic; size: 0.8; weight: bold; height: 0; bevelEnabled: true; bevelSize: 0.04; bevelThickness: 0.04; curveSegments: 1"
                  material="shader: flat; color: white; transparent: true; opacity: 0.4">
    </a-entity>
  </a-entity>

  <!--adds the grid to give a perspective of distance-->
  <a-entity
        geometry="primitive: plane; width: 10000; height: 10000;" rotation="-90 0 0"
        material="src: #grid; repeat: 10000 10000; transparent: true;metalness:0.6; roughness: 0.4; sphericalEnvMap: #sky;">
  </a-entity>

  <!--adds the lighting to make the scene more realistic-->
  <a-entity light="color: #ccccff; intensity: 1; type: ambient;" visible=""></a-entity>
  <a-entity light="color: ffaaff; intensity: 1.5" position="5 5 5"></a-entity>
  <a-entity light="color: white; intensity: 0.5" position="-5 5 15"></a-entity>
  <a-entity light="color: white; type: ambient;"></a-entity>

  <!--adds the sky-->
  <a-sky src="#sky" rotation="0 -90 0"></a-sky>
  <a-entity position="0 -5 0"
        geometry="primitive: plane; width: 10000; height: 10000;" rotation="-90 0 0"
        material="src: #grid; repeat: 10000 10000; transparent: true;metalness:0.6; roughness: 0.4; sphericalEnvMap: #sky;">
  </a-entity>
```

I've added easy to follow comments within the code that explain what each of the entity-component set implements in our app. For the VR newbies, a sky is like a layer that covers your 360 deg sphere inside which you assume yourself to be standing while you are experiencing a VR app. It's generally like the sky in real-life which can be seen on the top as well appears to fold down near the horizon. We use `a-sky` in A-Frame to add a sky and the resource to be used can be either a 360 deg image or just a solid colour.

Now comes the interesting part. We add a camera entity. This is a special entity offered by A-Frame that intrinsically grabs the continously changing position as well as rotation of your mobile phone while you are using an A-Frame powered VR app in the browser. It takes advantage of the various gyro sensors in your phone to achieve this under-the-hood.

Let's add the camera-entity as shown below. We can optionally give it a shape and animation, which helps us to track the movement by serving as a cursor.

```html
 <!--camera entity-->
  <a-entity>
    <a-entity camera look-controls wasd-controls id="user-cam">
      <a-entity position="0 0 -3" scale="0.2 0.2 0.2" geometry="primitive: ring; radiusOuter: 0.20; radiusInner: 0.13;" material="color: #ADD8E6; shader: flat" cursor="maxDistance: 30; fuse: true">
        <a-animation begin="click" easing="ease-in" attribute="scale" fill="backwards" from="0.1 0.1 0.1" to="1 1 1" dur="150"></a-animation>
        <a-animation begin="fusing" easing="ease-in" attribute="scale" fill="forwards" from="1 1 1" to="0.2 0.2 0.2" dur="1500"></a-animation>
      </a-entity>
    </a-entity>
  </a-entity>
```

By the end of this section, your VR app should ideally look like this: (Of course, if you haven't switched the resources with your custom ones!)

![first look](https://raw.githubusercontent.com/pluralsight/guides/master/images/83779eef-aa4b-4cd3-968e-1417afc4fd0b.51)

We just finished setting up the basic scene, let's now add some funtionality to it that will make the avatars appear and disappear as the users log in and out of your application.

#### Now let's make it realtime

So, let's now add the logic within the main.js file that we linked above. As I mentioned earlier, we'll add realtime functionality using deepstream's open realtime server.

Start by connecting to deepstream:

```javascript
var client = deepstream('<YOUR APP URL>')
client.login({}, function (success,data) {
	console.log("logged in", success)
	if(success){
		startApp(data)
	}else{
	    //handle login failed
	}
  
})
```

We can connect to deepstream's JS client as shown above. Navigate to deepstream's [dashboard](dashboard.deepstreamhub.com) and create an account for free. Next, add a new application give it a name and 