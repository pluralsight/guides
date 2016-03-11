__This article is incomplete and is not ready for review__

__I am a noob programmer, so please be gentle__

<h1>The problem</h1>
I commute everyday when I go to work. I also rarely look at my phone's weather app. Now that El Nino has hit Los Angeles, we get random rains. Combine all these and what do you get? A very wet and late-for-work programmer.

I got (synonym of surprised) by the rain several times because I am lazy to check the current weather. And my mind is now telling me, oooohhh, we have a problem, and I think we can create a simple solution for this (need to rephrase).

<h1>The proposed solution</h1>(need to pick a better subtitle)
Yes, I hear you screaming at me "get a car dude!". However, I am not ready to give up the extra minutes that I could snooze at the passenger seat. I mean, come on! There has got to be a way other than buying a car!

Actually there is, 3 letters. No, not C-A-R, but A-P-I. Or better yet, cloud services that you can engineer to do your bidding.

<h1>The requirements</h1>
Upon writing this tutorial, I only know C# (like how a 5th grade knows Physics). But I researched, and I ended up using the following services:

- Heroku, as a Platform As A Service
- Node.js and Express.js
- OpenWeatherAPI
- Twilio
(need to add links)

Why did I choose these services? Heroku, because I think Orion Henry is great when he talked in the 2016 Hack.Summit(). Node.js and Express.js, because I want to learn back end javascript-ing. OpenWeatherAPI because there is nothing else out there that is free (need to rephrase). Twilio, because everytime I google "Free SMS API", Twilio keeps popping even if it is not actually free and the fact that it has a Node.js API. Of course, you have all the freedom to choose your API. Whatever floats your boat, man. On this regard, I will also avoid certain specifics on the services and instead focus on how I built my own wet-programmer avoiding system.

<h2>Step 1</h2>
I set up Heroku. I had a bit of difficulty, but I was able to set up a local system that pushes whatever javascript code I created 
