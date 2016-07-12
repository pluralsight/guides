## The 3 reasons:
1. It is free.
2. It is well supported.
3. It is simple.

# Wait, that is WebRTC? 
Before I justify my reasons, let me answer the aforementioned question with another question. Does it pain you to have to download a third-party application or plugin to do simple things like have a video or audio chat (*cough* Skype, Google Hangouts, etc. *cough*)? Worse, imagine building an application that utilizes video or audio chat and having to write codecs for native apps... Web Real Time Communication, known among friends as WebRTC, to the rescue! 

# Free
WebRTC is an [open source project](https://webrtc.org/) with many spin-offs in various languages. The only part of WebRTC that is not built-in is the signaling mechanism or protocol, which is required to coordinate communication between the browsers involved in the connection. Messing with STUN/TURN servers manually can get hairy quickly so the easiest option is to user an already hosted public server or use a service that takes care of that for you: PubNub, Twilio, Xirsys, etc. For the curious George: https://www.twilio.com/docs/api/stun-turn/faq

# Supported
WebRTC is compatible with Firefox, Opera, and Chrome: desktop and mobile. There's even a cool browser that is built on top of WebRTC; it's called [Bowser](http://www.openwebrtc.org/bowser/) and was developed by Ericsson Research.

- ![Lots of stars!](https://raw.githubusercontent.com/pluralsight/guides/master/images/74b0733b-47a2-4fd4-85b2-46d815a0c065.24)

- ![Build passing!](https://raw.githubusercontent.com/pluralsight/guides/master/images/adc73fd4-643e-4368-a52a-cde90d8b6594.25)

- ![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0c38196c-3612-4a1b-8edf-dc265cc9963b.29)


I've personally used it on Chrome, iOS, and Android and it works well.

# Simple
Considering that my team and I were able to build an entire healthcare application using WebRTC for our company hackathon in a day or so, it isn't hard to use, assuming you've got the signaling nailed down. It has three simple APIs. Let's explore two of them now. The code below is purely for demonstration purposes: it is bare-bones and lacks fancy styling, error handling, and tracing.

Let's say this is your HTML file.

/index.html
```
<body>

  My first WebRTC app!
  
  <video autoplay></video>
  
  <script src="js/main.js"></script>
  
</body>
```

And, let's say this is your JS file.

/js/main.js
```
var video = document.querySelector('video');

navigator.getUserMedia = navigator.webkitGetUserMedia;

navigator.getUserMedia({video: true}, (stream) => { video.src = window.URL.createObjectURL(stream) }, () => { console.log("No stream") });

```

That's it! That's all the code you need to start using WebRTC's first API: getUserMedia(). The code above gets a stream from your webcam and sets it as the video source using a bit of ES6 syntax as well. Try it out: you should be seeing a live stream of your beautiful self :)

The second API that WebRTC uses is RTCPeerConnection, which is what's responsible for exchanging data between peers. Let's go ahead and add a second video element to our HTML file and add ids to differentiate between the two.

/index.html
```
<body>

  My first WebRTC app!

  <video autoplay id="one"></video>
  <video autoplay id="two"></video>

  <button id="connect"> Start peer connection </button>

  <script src="js/main.js"></script>
  <script src="js/adapter.js"></script>

</body>
```

Notice the adapter.js file, which you can get from here: https://github.com/webrtc/adapter.

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
  var servers = null;

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

Are you still with me??! Here we see RTCPeerConnection started for each peer. The first peer registers ICE candidates, when they become available, through a callback. Then, the first peer transmits this ICE candidate data to the second peer. Lastly, peers exchange information about the codecs/media configuration via offers and answers to set their internal descriptions. Try this out in your browser! If it's successful, you should be able to see your twin on the same screen (just kidding, it's two of you!).

The third API of WebRTC is RTCDataChannel. We won't cover that in this tutorial but know that it's available if you want to transmit data other than just video.

# Demo time

In the code above, we simulated peer connections on the same screen so there was no need for the STUN/TURN signaling servers I had mentioned in the beginning of this tutorial. But, as I also mentioned earlier, companies like PubNub offer services that do that dirty work for you. So, let's take advantage of that to build and deploy a revolutionary new app for any doctor in the world to log on and video chat with a patient in the browser with no additional downloads needed. We'll call it DocToc because I'm a poet and I know it.

PubNub makes it astonishingly simple to utilize WebRTC with the following calls (no pun intended):
- phone.dial
- phone.receive

Similar to before, let's set up our files.

/index.html
```
<body>
  <h1>DocTalk</h1>

  <div id="video"></div>

  <form name="loginForm" id="login" action="#" onsubmit="return selectUser(this);">
      <input type="text" name="username" id="username" placeholder="Pick a username!" />
      <input type="submit" name="login_submit" value="Select">
  </form>

  <form name="callForm" id="call" action="#" onsubmit="return makeCall(this);">
  	<input type="text" name="number" placeholder="Enter user to dial!" />
  	<input type="submit" value="Call"/>
  </form>

  <script src="js/main.js"></script>
</body>
```

/main/main.js
```
var video = document.getElementById("video");

function selectUser(form) {
	var phone = window.phone = PHONE({
	    number        : form.username.value || "Anonymous",
	    publish_key   : 'pub-c-a7775e85-ea67-4ecf-84d1-09f306ed9939',
	    subscribe_key : 'sub-c-df5175dc-2dcb-11e6-8bc8-0619f8945a4f',
	});

	phone.ready(function(){ form.username.style.background="#55ff5b"; });

	phone.receive(function(session){
	    session.connected(function(session) { video.appendChild(session.video); });
	    session.ended(function(session) { video.innerHTML=''; });
	});
	return false;
}

function makeCall(form){
	phone.dial(form.number.value);
	return false;
}
```

What's happening here? First, we grab the video element from the DOM. Then, we establish the functions to select a username and call someone else. The phone is established by providing your PubNub keys. Then, callbacks are attached to the phone so that it can append a video feed when it receives a call and remove it once the call is ended. Making the call itself is simply done by dialing with the username to call. Pretty cool! Deploy and test it out with friends.

What use-case can you think of for WebRTC? Post below!
