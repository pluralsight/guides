## The 3 reasons:
1. It is free.
2. It is simple.
3. It is well supported.

# Wait, that is WebRTC? 
Before I justify my reasons, let me answer the aforementioned question with another question. Does it pain you to have to download a third-party application or plugin to do simple things like have a video or audio chat (*cough* Skype, Google Hangouts, etc. *cough*)? Worse, imagine building an application that utilizes video or audio chat and having to write codecs for native apps... Web Real Time Communication, known among friends as WebRTC, to the rescue! 

# Free
WebRTC is an [open source project](https://webrtc.org/) with many spin-offs in various languages. The only part of WebRTC that is not built-in is the signaling mechanism or protocol, which is required to coordinate communication between the browsers involved in the connection. Messing with STUN/TURN servers manually can get hairy quickly so the easiest option is to user an already hosted public server or use a service that takes care of that for you: PubNub, Twilio, Xirsys, etc. 
https://www.twilio.com/docs/api/stun-turn/faq

# Simple
Considering that my Kareo team and I were able to build an entire healthcare application using WebRTC for our company hackathon in a day or so, it isn't hard to use, assuming you've got the signaling nailed down. It has three simple APIs. Let's explore them now.

Let's say this is your HTML file.
```
<body>

  <video autoplay></video>

  <script src="js/main.js"></script>

</body>

```

And, let's say this is your JS file.
```javascript
var video = document.querySelector('video');

navigator.getUserMedia = navigator.getUserMedia;

function successCallback(stream) {
  window.stream = stream;
  if (window.URL) {
    video.src = window.URL.createObjectURL(stream);
  } else {
    video.src = stream;
  }
}

function errorCallback(error) {
  console.log('navigator.getUserMedia error: ', error);
}

navigator.getUserMedia(constraints, successCallback, errorCallback);
```

That's it! That's all the code you need to start using WebRTC's first API: getUserMedia(). The code above gets a stream from your webcam. The second API that WebRTC uses is RTCPeerConnection, which is what's responsible for exchanging data between peers. The last API of WebRTC is RTCDataChannel, which allows peers to transmit data other than just video. 

# Supported
WebRTC is compatible with Firefox, Opera, and Chrome: desktop and mobile. There's even a cool browser that is built on top of WebRTC; it's called [Bowser](http://www.openwebrtc.org/bowser/) and was developed by Ericsson Research.
I've personally used it on Chrome, iOS, and Android and it works well.

![Lots of stars!](https://raw.githubusercontent.com/pluralsight/guides/master/images/74b0733b-47a2-4fd4-85b2-46d815a0c065.24)

![Build passing!](https://raw.githubusercontent.com/pluralsight/guides/master/images/adc73fd4-643e-4368-a52a-cde90d8b6594.25)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0c38196c-3612-4a1b-8edf-dc265cc9963b.29)

# Demo time

As I mentioned earlier, PubNub offers a We
