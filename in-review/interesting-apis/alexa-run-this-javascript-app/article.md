So there are two types of Alexa skill tutorial, the ones with lots of screenshots showing you how to create an AWS lambda function, configure your skill and deploy some arbitary code...and then there are the ones that show you how to develop a skill, test it, deploy it, and get it certified. There were too many in the former category (I recommend [this one](https://developer.amazon.com/public/community/post/Tx3DVGG0K0TPUGQ/New-Alexa-Skills-Kit-Template:-Step-by-Step-Guide-to-Build-a-Fact-Skill) and [this one](http://tutorials.pluralsight.com/node-js/amazon-alexa-skill-tutorial)), so I'm going for the latter.

A lot of the Alexa code samples you'll see will be very focussed on getting setup quickly with some intents, which is understandable. This article will be an *intentsive* look at how to put together a robust app that you can scale in complexity with confidence. It will be modular, use TDD, be linted, have a CI pipeline and an easy deployment step using [Gulp](http://gulpjs.com/).

Generally I'll make the assumption you have a passing notion of an Alexa skill, and if not, read those two links first and come back...please. Oh yeah, it's all about the JavaScript too (in case you had a penchant for Python).

# The current landscape

You can be forgiven for finding it hard when starting to develop with Alexa. While there is a plethora of documentation...there's almost nothing on how to actually build a skill. If you're anything like me, you keep going round in circles and ending up [here](https://developer.amazon.com/ask) when trying to understand what the skills kit is...when actually it's just a big umbrella term. Then you might end up at their [node kit](https://github.com/amzn/alexa-skills-kit-js), or at least what you think is their Node SDK but sort of isn't...although it's still actively maintained. As you look at the samples you'll notice some code that you thought died with ES2015, and if you use a framework, perhaps haven't written for a good few years:

```javascript
Fact.prototype = Object.create(AlexaSkill.prototype);
Fact.prototype.constructor = Fact;
```

...but help is at hand because you stumble across [this guy](https://github.com/matt-kruse/alexa-app) and you think, ah cool, this API is a bit nicer, then start attempting to write your skill with it...although you're still not entirely sure how to go about it and it's not all Amazon official like.

However, there was this [new SDK announced](https://developer.amazon.com/public/community/post/Tx213D2XQIYH864/Announcing-the-Alexa-Skills-Kit-for-Node-js) (that you'll be hard pressed to find anywhere on the site, so the repo is [here](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs)). This is the guy we'll be working with in the following tutorial.

# Starter for 10

I've created a start-kit thing based on my experience creating my [Arithlistic](https://github.com/craigbilner/arithlistic) skill, [here](https://github.com/craigbilner/alexa-starter-kit). Probably best if you read this section with a split screen of the tutorial and the starter-kit.

## Structure

All the skills split the code between `src` and `speech-assets` - let's not buck the trend. Other than that I've stuck the logos in here (which are needed when you go live in the store) and my [circleci](https://circleci.com/) config. You can use any CI but as you can see from mine, you'll need to specify that all your code is in the nested `src` directory and because we're masochists who have written our tests in ES2015, we need Node 6.x.x.

### speech-assets

#### intent-schema.json

A config which basically explains to Alexa the "things people say" (intents) that she should expect, and the types of the slots (a variable in the intent). This will help her more easily work out how to interpret what the user is saying.

#### uterrances

Utterly mystifying beasts that attempt to tame the english language. It's basically another helping hand to give Alexa a fighting chance in interpreting what the user is saying. If they say this thing, they mean this. The words in curly braces represent the slots, whose types are defined in your json schema next door. Be careful with one word intents, for Arithlistic I use "pass" which gets triggered through random words a little too much when you expect it to only trigger when someone says "pass"...

### src

In the special `src` you cook your skill.

#### At the top level

We effectively have our dev assets and fixtures. Potentiallly you could split these into their own directory but it felt like overkill. So we have:

* eslint configs that have to be rather liberal due to the SDK we'll be working with
* your secret `deploy.env.json` (that you'll create yourself obviously) which will look a little something like this:

```json
{
  "ROLE": "arn:aws:iam::[123456789012]:role/[the AWS lambda role you defined]",
  "ACCESS_KEY_ID": "ABCDEFGHIJKLMNOPQRST",
  "SECRET_ACCESS_KEY": "abcdefghijklmnopqrst/abcdefghijklmnopqrs"
}
```

this allows us to deploy our code with a gulp task

* our `enums.js` which will help us manage the state of our skill, just to give us some semblance of type safety in JavaScript land
* a `gulpfile.js` that details our deployment tasks. It removes our old `dist` stuff, grabs all the stuff we care about and puts it in `dist`, zips it up because this is how AWS lambda takes multiple files and uses [node-aws-lambda](https://www.npmjs.com/package/node-aws-lambda) to upload our file using the `lambda-config` that has had it's values replaced with our secret ones from above
* `index.js` - the entry to our skill - it sets the environment path to help our AWS lambda traverse our files, registers our `appId` and `handlers` (which we'll come to) and kicks the whole thing off
* the `lambda-config.js` which will look very familiar to the screen in your [AWS lambda console](https://console.aws.amazon.com/lambda/home?region=us-east-1). Set these values as desired
* `package.json` which - I'm going to assume you know. The only real points to note are the scripts:
 * Finding a good way to deploy an Alexa skill is a bit of a PITA, this is [not a deployment workflow](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/deploying-a-sample-skill-to-aws-lambda#preparing-a-nodejs-sample-to-deploy-in-lambda). I found these two examples ([Rick's](https://github.com/rickwargo/alexa-skill-template) and [Thoughtworks'](https://github.com/ThoughtWorksStudios/node-aws-lambda)) which I borrowed heavily from for my own `gulpfile`. However, they both like a `PROD` install whereas I've gone for a slightly different approach to avoid doing an install every time I deploy. When you `npm i` for your skill, it'll stick the `PROD` dependencies straight into `dist`
 * I've also stuck `iron-node` in there, which I've had mixed experiences with, but is actually invaluable when you want to debug something easily in Chrome's dev tools. Please feel free to debug your node code your preferred way though, such as through your IDE
* `responses.js` should be where you put anything Alexa says. I would do them as functions to have a consistent API where you can pass in values to be dynamically built. In fact I refactored these out to call the emit methods in order to store the last known reponses and states to help us continue after a "pause". Having all the responses in one place makes testing easier (as you can change the phrases without breaking your tests) and when i18n comes in, you can swap this file out, rather than have hard coded text littered over your code base

#### handlers

These are the boys who receive the `intents` (what Alexa thinks the user said based upon your `speech-assets`), perform some kind of business logic, perhaps store some stuff in session state (or dynamo db) and return a response (something for Alexa to say back).

The state stuff is necessary because the skill is very dumb, it takes some JSON in, it sends some JSON out. It's because of this pure nature, TDD (and testing in general) is so easy. When we get to the `event-samples` later you'll see what that JSON looks like. It's a good idea to keep your handlers as skinny as possible and move the business logic into the `modules` folder.

I've gone for a rather rigid approach of commenting each handler's intent to ensure it follows the same pattern when edited. The pattern being, setup, state changes, response. As a member of the church of functional programming, it would have been preferable to get an immutable model in the intent, perform your logic, update that thing, then return it with your response. However, the SDK has gone for a more OO approach with heavy use of `this` to fire events and manage state...which is less than ideal and led to some wonky workarounds.

I advise having as many small handlers as you can manage, this will make handling ambiguous intents such as "yes" and "no" much easier and helps to organise the code and test.

In the starter-kit we only have three handlers:

* `core.handlers.js` - we'll get to "certification" at the end but for now you just need to know these are required, so we'll mix them in to all our other handlers. `AMAZON.StopIntent` is basically pause and `AMAZON.CancelIntent` is "get me out of here". The `SessionEndedRequest` is just a nice to have for logging purposes
* `new-session.handlers.js` - this is the skill entry point (as oppposed to `index.js` which is the code entry point), we'll set some state here (which decides which handler to go to) and tell our skill which intent to run. There's also the random "catch-all" which will log out something really useless...
* `stopped.handlers.js` - because `StopIntent` and `CancelIntent` are generally required, ideally we want one place to manage being "paused". Here we set the state back to the previous state if the user wishes to continue and exit if they...don't want to continue

#### modules

This is where your business logic should live. As mentioned above, all the heavy lifting should be in here. We don't want the `intents` to get too busy and dirty because it makes it hard to read and hard to maintain. I've included a small naïve mixin implementation in the `utils` module so we don't need to add the core intents over and over.

#### tests

None of the samples have tests. I don't believe you can actually develop and ship an Alexa skill without tests. There are so many paths, branches and intents, that as soon as you add a modicum of complexity to your skill, you'll get bugs.

In the tests folder we have our fixtures known as `event-samples` and a giant test file. Now, you could probably split this up but it feels like a "pre-optimisation". I quite liked having all my skill test paths together.

So we have some helpers at the top:

* `sanitise` just to make life easier if you've used a multi-line template string and want to assert on the speech output
* `getOutputSpeech` which just destructures the response, strips everything back (the ssml tags) and allows us to assert on what we care about, what Alexa said
* `getAttribute` which again is a nice to have abstraction to pick out our session attributes to test on
* `runIntent` finally, but by no means least, this is the thing that will make all your testing a dream. It uses [aws-lambda-mock-context](https://www.npmjs.com/package/aws-lambda-mock-context) to stub out all the context stuff we don't really care about and gives us a fluent API to invoke our intent with ease

I've gone for a heavily nested approach to give it the feel of a conversation when you read the output. So each `describe` is what the user would say, then the test name is what you think your skill should have done. As you can see we can now nicely destructure and `assert` in quite a clean manner. Also using fat arrows means less curly braces for each `it` for the win.

I would split the `event-samples` by handler state. This just makes them more manageable. This is an example `event-sample` which would be in the `playing` directory:

```json
{
  "session": {
    "sessionId": "sessionId",
    "application": {
      "applicationId": "appId"
    },
    "attributes": {
      "playerCount": 1,
      "activePlayer": 0,
      "players": [
        {
          "name": "craig",
          "score": 0,
          "correctAnswers": 0
        }
      ],
      "STATE": "PLAYING",
      "startTime": "2016-07-31T16:58:47Z",
      "timeOfLastQuestion": "2016-07-31T16:58:47Z",
      "currentAnswer": 6
    },
    "user": {
      "userId": "userId"
    },
    "new": false
  },
  "request": {
    "type": "IntentRequest",
    "requestId": "reqId",
    "locale": "en-US",
    "timestamp": "2016-07-31T16:59:00Z",
    "intent": {
      "name": "AnswerIntent",
      "slots": {
        "Answer": {
          "name": "Answer",
          "value": "5"
        }
      }
    }
  },
  "version": "1.0"
}
```

You can see our game state comes in as `attributes`, along with the request which has a name and the intent's slot values (which for some reason doubles up the name ¯\\\_(ツ)_/¯). This is why it's a beauty for TDD purposes, we take this JSON in, we test the next state and the response, done.

A small note, you'll need Node 5.x.x + to get the benefits of ES2015 but AWS lambdas only support Node 4.3.2 which has limited ES2015 support. In order to have a nice dev experience but still be able to test the skill locally, I use [nvm](https://github.com/creationix/nvm) to easily switch from one to t'other.


## Writing the thing

Please now switch to my [demo skill](https://github.com/craigbilner/alexa-demo-skill) which I'll use to talk through my dev process. I've used my starter-kit to scaffold it out, and will reference the commits so you can see the changes through the diffs.

### In the beginning

There is a suggestion that you create a flow diagram before creating your skill. This does seem like hard work, but I think it probably pays dividends for something with many paths. So that's where I started, [here](https://github.com/craigbilner/alexa-demo-skill/commit/43674dca5a4601243021852e3dc41fc6673c48ca). We'll be starting an incredibly adventurous adventure game called, the "Dangerous Forest".

First we want Alexa to say something to introduce the game when it's started, so we write that out:

`responses.js`

```javascript
module.exports.gamePrelude = () =>
  'This is a dangerous game of cat and mouse in the even more dangerous forest, do you want to play?'
```

The flow diagram may differ from the actual response as you test and play around with how Alexa says things. She talks rather rapidly so you'll need a large supply of commas and other punctuation to keep her in check. For Arithilistic I was also a bit of a pedant and changed the responses if they came after a pause or if multiple people were playing, basically try and keep it contextual and conversational, as it will make it feel much more polished.

We add our starting game state:

`enums.js`

```javascript
GAME_START: 'GAME_START',
```

And write a test:

```javascript
describe('Alexa, start game', () => {
  it('Respond with game prelude and set state to GAME_START', () =>
    runIntent(sessionStartIntent)
      .then(({ outputSpeech, gameState }) => {
        assert.deepEqual(outputSpeech, sanitise(gamePrelude()));
        assert.deepEqual(gameState, GAME_STATES.GAME_START);
      }));
});
```

Which will fail :-)

```javascript
1) Alexa, start game Respond with game prelude and set state to GAME_START:
     Error: timeout of 2000ms exceeded. Ensure the done() callback is being called in this test.
```

Then wire up our initial intents, by setting the beginning game state in `this.handler.state` and invoking our `GameIntro` intent, which will live in the `GAME_START` handlers. Essentially `emithWithState` says, call the intent that it's in the handler of the current state.

`new-sessions.handlers.js`

```javascript
const setStateAndInvokeEntryIntent = function() {
  // updates
  this.handler.state = GAME_STATES.GAME_START;

  // response
  this.emitWithState('GameIntro');
};
```

And add `game-start.handlers` to our index file and pass them into the registration function.

`index.js`

```javascript
const gameStartHandlers = require('./handlers/game-start.handlers');
```

The test now passes. See the [commit for details](https://github.com/craigbilner/alexa-demo-skill/commit/b3c83c207781f14a4e5f6e134d31b1401a50d5f2).

### Start game failures

So now we have the "start rhombus", let's fill in the various failure conditions. For example, a player may say "No". For something like this we can use one of the built in Amazon intents like so:

```javascript
'AMAZON.NoIntent': function() {
    res.tell.call(this, res.goodbye());
},
```

For linting, we write out the function call rather than do it as a computed. I do this rather ugly `res.tell.call` thing because I want to capture previous responses (or overload with a *continuation) and state, but explicity preserve `this`. It's also useful for strong typing again rather than `':ask'` and `':tell'` magic strings, a fix I've seen in common with other Alexa approaches on GitHub.

\* I created the concept of a continuation, so that you can respond with similar text after a pause but as someone would speak when continuing, rather than repeating parrot fashion. This optional string is set as the `previousResponse` in `responses.js`.

When using a "tell", the session will end because Alexa no longer requires the user to respond, she can be dismissive like that. When I expect the session to end I can test for it like this:

```javascript
assert(endOfSession);
```

You'll notice that while I have helpers at the top of my tests to make things easier, a lot of the tests end up looking the same or similar. I think DRY in tests is a balancing act where in general you can be quite wet.

We add a stopped `event-samples` directory, because we didn't change state, we can just copy and paste the game start's intent fixtures.

And that's it. You'll notice the `core.handlers` have taken care of "cancel" and "stop" for us which we can just write tests for.

When doing it for real, make sure all paths do something, usually via the prompt text or the unhandled "catch-all". Also make sure you implement an `AMAZON.HelpIntent` for each handler. If you wanted to switch state to `HELP` and do a whole help thing, then you'd obviously create a directory and test for the state switches back and forth.

Code can be found [here](https://github.com/craigbilner/alexa-demo-skill/commit/0594aaa8de5305206eba5993fa7d30147c326a08).

### Fill in the responses

So we've sort of done the top part of our flow-diagram. We'll now go down and make the game do something. It's important to remember to cover all the "tedious" paths above each time to ensure the user can't get stuck (and that we get certified).

We're going to go to three different "places" in the forest, so let's create three new states in our `enums.js`.

```javascript
PLAYING: 'PLAYING',
EVIL_PIG: 'EVIL_PIG',
MYSTERIOUS_WITCH: 'MYSTERIOUS_WITCH',
```

`PLAYING` will be when we've opted into the game and the other two are left and right on our flow-diagram.

Then we add a `yes.intent.json` fixture to our `event-samples/game-start` directory and `left.intent.json` and `right.intent.json` fixtures to our `event-samples/playing` directory. Pull them into our tests and asserts things we expect such as:

```javasacript
describe('I would like to go left', () => {
    it('Responds with evil pig intro and sets state to EVIL_PIG', () =>
        runIntent(playingLeftIntent)
            .then(({ outputSpeech, gameState }) => {
                assert.deepEqual(outputSpeech, sanitise(evilGoatPig()));
                assert.deepEqual(gameState, GAME_STATES.EVIL_PIG);
            })
    );
});
```

These will now fail because we don't have any of the logic. Let's wire in the responses such as:

`responses.js`

```javascript
module.exports.evilGoatPig = () =>
    'There\'s an evil goat pig who offers you 30 pieces of silver, do you take it?';
```

Great, so we have our tests, we have our states that we want to get to, and we have what Alexa is going to say. So to get out of `GAME_START` and into `PLAYING` we add the `AMAZON.YesIntent` to our `GAME_START` handler.

```javascript
'AMAZON.YesIntent': function() {
    // updates
    this.handler.state = GAME_STATES.PLAYING;
 
    // response
    res.ask.call(this, res.enterForest());
},
```

This will flip our state, speak aboute entering the forest to the user and then when they respond, the intents in our `PLAYING` handler will be invoked. So let's add that, it's quite large so I won't put it here. You can see it follows the same pattern though, pull in the `alexa-sdk`, mixin our `core.handlers`, `CreateStateHandler` with our `PLAYING` state then implement `GoLeftIntent`, `GoRightIntent`, `AMAZON.HelpIntent` and `Unhandled`. In the left and right intents, we set the desired state to "jump" to our next handler and tell the user something.

I'll leave it as an exercise for the reader to create a truly surprising random game, but this is where you wouldn't hard code the state but pick a random one. Hint: use a seeded random number generator to make it testable.

We then do the same for the pig and witch handlers, wire them in to our `index.js` file and because Bob is our uncle, the tests pass.

Code is [here](https://github.com/craigbilner/alexa-demo-skill/commit/747f8d25e6c16f3c0dbf8fffee5797fdb5f3c1e5).

### The assets

Even though we've created our game logic and all our tests are passing, Alexa won't actually be able to use it yet. Take a look at the commit [here](https://github.com/craigbilner/alexa-demo-skill/commit/7fca9e2e95aaf95456c5440257a481cf79bb4ee3). In our `intent-schema.json` we have to tell her what type of intents she should expect and for our final intent, the type of the slot.

Then in `utterances.txt` we need to be able to start the game #obvs, and tell her what a person needs to say in order to invoke our intents, using curly braces for the cards slot. These go into our source control for posterity but as you've seen from the other tutorials, you'll need to take these and put them in your skills config.

### Slotting one in

We only stubbed out the mysterious witch's handler, so we'll now finish off with how you can take parts of what the user says and perform some logic.

We create `odd.intent.json` and `even.intent.json` fixtures where we pass odd and even numbers in as our slot e.g.

```json
"intent": {
    "name": "NumberOfCardsIntent",
    "slots": {
        "NoOfCards": {
            "name": "NoOfCards",
            "value": 4
        }
    }
}
```

we're also super defensive so we create a `blablabla.intent.json` fixture with no value at all. Then we...write our tests.

This time our response will actually use a parameter like so:

```javascript
module.exports.cardConfirmation = goEven =>
    `You shall go down the ${goEven ? 'cobbled' : 'broken bridge'} path`;
```

Then in our `NumberOfCardsIntent` in `mysterious-witch.handlers.js` we implement the logic. You'll notice this rather lengthy thing:

```javascript
this.event.request.intent.slots.NoOfCards.value
```

where you literally reach in and pluck out the value, perhaps it would have been nicer to come in as an argument of the intent.

We can then put our rather trivial logic into our modules folder which parses the number and decides whether it's odd or even. This is just to emphasise the point that you'll want to keep your intents as skinny as possible. If someone does say "blablabla" then it will call the `Unhandled` intent in the same handler.

You can find the commit [here](https://github.com/craigbilner/alexa-demo-skill/commit/85f682cf3c46e994df64bf1f2ef1b6c791b314f6).

### Trying it out

You've now got some basic logic for your game and you know it works because you have a ton of tests, however you don't really have a feel for the game. You'll now want to start iterating, which means quickly deploying your code to your [lambda function](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/Arithlistic?tab=code) and checking it in the [Alexa console](https://developer.amazon.com/edw/home.html#/skills/list).

If you've filled out your `deploy.env.json` and `lambda-config.js` files correctly, you should be able to `npm run deploy` which will zip up the pertinent stuff and throw it on your lambda. Then go to the Alexa console, then the test tab on the left and type as if you would talk.

I recommend using this hack so you can read the requests and responses more easily rather than fiddling with the rather small boxes:

```javascript
document.head.getElementsByTagName('style')[0].sheet.insertRule('.AppManagementViewContainer { overflow: visible !important; }', 0);
document.head.getElementsByTagName('style')[0].sheet.insertRule('.Simulator-tab { display: flex; flex-flow: column; }', 1);
document.head.getElementsByTagName('style')[0].sheet.insertRule('.CodeMirror { width: 1500px !important; }', 2);
```

Be sure to type random things and the opposite of what you would expect to really kick the tyres. If it fails, grab the JSON, stick it in your `event-samples`, write a test etc. create a PR, push the code, merge if all good with your CI tools.

Click the play button to get a feel for how Alexa is saying your responses and punctuate as necessary.

When you think you're all done, head over to [echosim](https://echosim.io/) to try it out. You can then login to the [alexa app](http://alexa.amazon.com/spa/index.html#settings/dialogs) and see what you said and what actually happened. In fact you can play back the recording of what Alexa captured and her interpretation. For example, Alexa cannot understand me when I say "two players", so for now, you're going to need a US accent or use a different utterance.

# Going live

Finally, you've slaved over your project, it's all working, just hit the "certification" button and it'll be in the app store in a couple of days...unless you're like me and a bit of a rogue with the rules.

Please learn from my mistakes:

* Check your invocation name against the [rules](https://developer.amazon.com/appsandservices/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill). Then check it again. Then again. It might still fail, and they don't furnish you with the reason, so you may just need to have another go
* Handler all the things. Even if it doesn't make sense, make sure you've got a "stop", "cancel", "help", "repeat" etc. for every state. Especially "repeat"
* Add lots of different utterances to reduce the friction of your app, otherwise they'll have to keep saying very exact phrases just to get your skill to play ball

# Stuff I didn't cover because I've written lots of other words

* There are many more [built-in intents](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/implementing-the-built-in-intents) than I've mentioned - use the one that makes sense
* There are different [types of output](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interface-reference) including cards, which allows you to accompany your oral skill with visual content
* There is a lot more you can do with [slots](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/alexa-skills-kit-interaction-model-reference) including custom slots. N.B `AMAZON.Literal` is deprecated and shouldn't be used

So that's that. Tweet me @CraigBilner with any skill you make or any questions you have...or abuse for my mistakes, thanks.