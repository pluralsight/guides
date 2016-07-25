# Wait, what is WebRTC? 
Let me ask you a question. Does it pain you to have to download a third-party application or plugin to do simple things like have a video or audio chat? (I'm looking at you, Skype and Google Hangouts). Worse, imagine building an application that utilizes video or audio chat and having to write codecs for native apps. **Web Real Time Communication**, known among friends as WebRTC, to the rescue! 

## The 3 reasons:
1. It is free.
2. It is well supported.
3. It is simple.

# Free
WebRTC is an [open source project](https://webrtc.org/) with many spin-offs in various languages. The only part of WebRTC that is not built-in is its signaling mechanism or protocol, which is required to coordinate communication between the browsers involved in the connection. Messing with the signaling (mainly, STUN and TURN servers) manually can get hairy quickly so the easiest option is to use already-hosted public servers or to use a service that takes care of that for you, such as PubNub, Twilio, XirSys, and so on. 

If you're curious about STUN/TURN servers, check out this [link](https://www.twilio.com/docs/api/stun-turn/faq).

# Supported
WebRTC is compatible with Firefox, Opera, and Chrome: desktop and mobile. There's even a cool browser that is built on top of WebRTC; it's called [Bowser](http://www.openwebrtc.org/bowser/) and was developed by Ericsson Research.

- ![Lots of stars!](https://raw.githubusercontent.com/pluralsight/guides/master/images/74b0733b-47a2-4fd4-85b2-46d815a0c065.24)

- ![Build passing!](https://raw.githubusercontent.com/pluralsight/guides/master/images/adc73fd4-643e-4368-a52a-cde90d8b6594.25)

- ![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0c38196c-3612-4a1b-8edf-dc265cc9963b.29)


I've personally used it on Chrome, iOS, and Android and it works well.

# Simple
Considering that my team and I were able to build an entire healthcare application using WebRTC for our company hackathon in a day or so, it isn't hard to use, assuming you've got the signaling nailed down. 

It has three simple APIs. Let's explore two of them now. The code below is purely for demonstration purposes (it is bare-bones and lacks fancy styling, error handling, and tracing) and it has been adapted from Google Codelabs to make WebRTC even simpler to understand.

Let's say this is your HTML file:

/index.html
```
<body>
  My first WebRTC app!
  <video autoplay></video>
  <script src="js/main.js"></script>
</body>
```

And, let's say this is your JS file:

/js/main.js
```
var video = document.querySelector('video');
navigator.getUserMedia = navigator.webkitGetUserMedia;
navigator.getUserMedia({video: true}, (stream) => { video.src = window.URL.createObjectURL(stream) }, () => { console.log("No stream") });

```

That's it! That's all the code you need to start using WebRTC's first API, getUserMedia(). The code above gets a stream from your webcam and sets it as the video source using a bit of ECMAScript 6 syntax as well. 

Try replicating the above by following these steps (here's a [JSFiddle](https://jsfiddle.net/pveb37xs/) for reference): 
- create a folder on your desktop labeled 'webrtc,' 
- place the index.html file inside the webrtc folder 
- create a folder labeled 'js' within the webrtc folder 
- place the main.js file inside the js folder 
- open a terminal and navigate to the webrtc folder
- run 'python -m SimpleHTTPServer' on the terminal
- go to localhost:8000 on your Chrome browser (important as the code in main.js is specific to Chrome)
- you should be seeing a live stream of your beautiful self!

The second API that WebRTC uses is RTCPeerConnection, which is what's responsible for exchanging data between peers. Let's go ahead and add a second video element to our HTML file and add IDs to distinguish between the two elements.

/index.html
```
<body>
  My first WebRTC app!
  <video autoplay id="one"></video>
  <video autoplay id="two"></video>
  <button id="connect"> Start peer connection</button>
  <script src="js/main.js"></script>
  <script src="js/adapter.js"></script>
</body>
```

Notice the adapter.js file, which you can get from here: https://github.com/webrtc/adapter. The adapter file is just a shim, that acts to make sure the API calls are compatible in different environments. 

Let's add some peer connection code in the js file.

/js/main.js
```
// initialize vars
var videoOne = document.getElementById('one');
var videoTwo = document.getElementById('two');
var connectButton = document.getElementById('connect');
var videoStream;
var peerConnOne;
var peerConnTwo;
var offerOptions = { offerToReceiveAudio: 1, offerToReceiveVideo: 1};

//set up click handler
connectButton.onclick = startConnection;

//get video for peer one
navigator.getUserMedia = navigator.webkitGetUserMedia;
navigator.getUserMedia({video: true}, videoOneSource, function(){ console.log("No stream")});

//supporting functions
function videoOneSource(stream) {
  videoStream = stream;
  videoOne.src = window.URL.createObjectURL(stream);
}

function startConnection() {
  var servers = null; // this is where we could use STUN/TURN servers

  peerConnOne = new RTCPeerConnection(servers);
  peerConnOne.onicecandidate = function(e) { onIceCandidate(peerConnOne, e);};

  peerConnTwo = new RTCPeerConnection(servers);
  peerConnTwo.onicecandidate = function(e) { onIceCandidate(peerConnTwo, e);};

  peerConnTwo.onaddstream = function (data) { videoTwo.srcObject = data.stream; }

  peerConnOne.addStream(videoStream);

  peerConnOne.createOffer(offerOptions)
    .then(onCreateOfferSuccess);
}

function onIceCandidate(pc, event) {
  var peer = (pc === peerConnOne) ? peerConnTwo : peerConnOne;
  if (event.candidate) {
    peer.addIceCandidate(new RTCIceCandidate(event.candidate));
  }
}

function onCreateOfferSuccess(desc) {
  peerConnOne.setLocalDescription(desc);
  peerConnTwo.setRemoteDescription(desc);
  peerConnTwo.createAnswer()
    .then(onCreateAnswerSuccess);
}

function onCreateAnswerSuccess(desc) {
  peerConnTwo.setLocalDescription(desc);
  peerConnOne.setRemoteDescription(desc);
}
```

There's a lot going on here. Are you still with me? Here we see RTCPeerConnection started for each peer. The first peer registers [ICE candidates](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API/WebRTC_basics#ICECandidate), when they become available, through a callback. Then, the first peer transmits each ICE candidate's data to the second peer. Lastly, peers exchange information about the codecs/media configuration via offers and answers to set their internal descriptions. 

Try this out in your browser following the same steps for the first API! Here's a [JSFiddle](https://jsfiddle.net/c22djhw4/) for reference. If you're successful, you should be able to see your twin on the same screen (just kidding, it's two streams of just you!).

The third API of WebRTC is RTCDataChannel. We won't cover it in this tutorial but know that it's available if you want to transmit data other than just video.

# Demo time

In the code above, we simulated peer connections on the same screen so there was no need for the STUN/TURN signaling servers I had mentioned in the beginning of this tutorial. Plus, we were running this locally so there was no need to transmit data across different peers. But, we want to deploy this application on the Interwebz so that any two people can utilize the power of WebRTC.

As I had mentioned earlier in this tutorial, companies like PubNub offer services that do the signaling dirty work for you. So, let's take advantage of that to build and deploy a revolutionary new app for any doctor in the world to log on and video chat with a patient in the browser with no additional downloads needed. We'll call it DocTalk because I'm a poet and I know it.

PubNub makes it astonishingly simple to utilize WebRTC with calls (no pun intended) that have abstracted and simplified the APIs you learned about above. Ugh, then why didn't we just learn this first? Because we've got to know how it works under the hood!

Here are the two API calls:
- phone.dial
- phone.receive

Simple, right?! Check out PubNub's documentation for additional APIs: https://github.com/stephenlb/webrtc-sdk

Let's create and deploy the DocTalk WebRTC app using [PubNub's WebRTC demo](https://www.pubnub.com/blog/2015-08-25-webrtc-video-chat-app-in-20-lines-of-javascript/) and [Surge](https://surge.sh/), respectively.

Similar to before, set up the index.html and js files as shown below (note that these files are in the folder 'doctalk').

doctalk/index.html
```
<body>
  <h1>&hearts; DocTalk &#9877;</h1>
  <div id="video"></div>
  <form onsubmit="return selectUser(this);">
      <input type="text" name="username" placeholder="Pick a username!" />
      <input type="submit" value="Select">
  </form>
  <form onsubmit="return makeCall(this);">
  	<input type="text" name="number" placeholder="Enter user to dial!" />
  	<input type="submit" value="Call"/>
  </form>
  <script src="js/main.js"></script>
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
  <!-- below are PubNub's signaling code and abstracted APIs that make WebRTC even simpler -->
  <script src="https://cdn.pubnub.com/pubnub-3.7.14.min.js"></script>
  <script src="https://cdn.pubnub.com/webrtc/webrtc.js"></script>
</body>
```

doctalk/main/main.js
```
var video = document.getElementById("video");

function selectUser(form) {
	// set up the phone
	var phone = window.phone = PHONE({
	    number        : form.username.value || "Anonymous",
	    publish_key   : 'insert your publish_key here',
	    subscribe_key : 'insert your subscribe_key here',
	    ssl : (('https:' == document.location.protocol) ? true : false)
	});
	// denote it's ready
	phone.ready(function(){ form.username.style.background="#55ff5b"; });
	// set up callbacks that execute upon start or finish of session
	phone.receive(function(session){
	    session.connected(function(session) { video.appendChild(session.video); });
	    session.ended(function(session) { video.innerHTML=''; });
	});
	return false;
}

function makeCall(form){
	// dial a phone number; this is initiating the peer connection over STUN/TURN servers
	phone.dial(form.number.value);
	return false;
}

```

Let's take a look at what is going on above. 

First, we grab the video element from the DOM. Then, we establish the functions to select a username and call someone else. The phone is established by providing your PubNub keys, which you can acquire by creating a free account on PubNub and going to your settings page. Then, callbacks are attached to the phone so that it can append a video feed when it receives a call and remove it once the call is ended. Making the call itself is simply done by dialing with the username to call. Pretty cool! 

We can deploy this static site with Surge by following the steps below (this assumes you have [npm](https://www.npmjs.com/) installed; it comes bundled with node.js so if you don't have it, please install npm or node first):
- open a terminal and navigate to the doctalk folder
- run 'npm install --global surge' on the terminal
- run 'surge' on the terminal, make sure the path is correct, change the domain name of the app to doctalk.surge.sh and hit enter  
- run 'surge --domain https://doctalk.surge.sh' and voila! 

It's deployed! Here's my link: https://doctalk.surge.sh/. Select a username, tell a friend your usename, and have your friend call you! 

If you've been following along with the code, deploy your version and test it out!

What use-cases can you think of for WebRTC? Post them below, along with any comments, feedback, questions, or trouble that you may have regarding this tutorial!
