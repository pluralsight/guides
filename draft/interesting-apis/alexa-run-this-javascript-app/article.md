So there are two types of Alexa skill tutorials, the ones with lots of screenshots showing you how to create an AWS lambda function, configure your skill and deploy some arbitary code...and then there are the ones that show you how to develop a skill, deploy it, test it and get it certified. There were too many in the former category (I recommend [this one](https://developer.amazon.com/public/community/post/Tx3DVGG0K0TPUGQ/New-Alexa-Skills-Kit-Template:-Step-by-Step-Guide-to-Build-a-Fact-Skill) and [this one](http://tutorials.pluralsight.com/node-js/amazon-alexa-skill-tutorial)), so I'm going for the latter. In fact it's going to be less tutorial and more opinionated rambling with code snippets. Generally I'll make the assumption you have a passing notion of an Alexa skill, and if not, read those two links first and come back...please. Oh yeah, it's all about the JavaScript too (in case you had a penchant for Python).

# The current landscape

You can be forgiven for finding it hard when starting to develop with Alexa. While there is a plethora of documentation...there's almost nothing on how to actually build a skill. If you're anything like me, you keep going round in circles and ending up [here](https://developer.amazon.com/ask) when trying to understand what the skills kit is...when actually it's just a big umbrella term. Then you might end up at their [node kit](https://github.com/amzn/alexa-skills-kit-js), or at least what you think is their Node SDK but sort of isn't...although it's still actively maintained. As you look at the samples you'll notice some code that you thought died with ES2015, and if you use a framework, perhaps haven't written for a good few years:

```javascript
Fact.prototype = Object.create(AlexaSkill.prototype);
Fact.prototype.constructor = Fact;
```

...but help is at hand because you stumble across [this guy](https://github.com/matt-kruse/alexa-app) and you think, ah cool, this API is a bit nicer, then start attempting to write your skill with it...although you're still not entirely sure how to go about it. However, there was this [new SDK announced](https://developer.amazon.com/public/community/post/Tx213D2XQIYH864/Announcing-the-Alexa-Skills-Kit-for-Node-js) that you'll be hard pressed to find anywhere on the site, so the repo is [here](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs). This is the guy we'll be working with in the following tutorial.

# Starter for 10

I've created a start-kit thing based on my experience creating my skill, [Arithlistic](https://github.com/craigbilner/arithlistic), [here](https://github.com/craigbilner/alexa-starter-kit). A lot of the code samples you'll see will be very focussed on getting setup quickly with some intents, which is understandable. This article will be an *intentsive* look at how to put together a robust app that can you scale in complexity with confidence. It will be modular, use TDD, be linted, have a CI pipeline and an easy deployment step using [Gulp](http://gulpjs.com/).

## Structure

Probably best if you read this section with a split screen of the tutorial and the starter-kit.

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
* your secret `deploy.env.json` (which you'll create yourself obviously) which will look a little something like this:

```json
{
  "ROLE": "arn:aws:iam::[123456789012]:role/[the AWS lambda role you defined]",
  "ACCESS_KEY_ID": "ABCDEFGHIJKLMNOPQRST",
  "SECRET_ACCESS_KEY": "abcdefghijklmnopqrst/abcdefghijklmnopqrs"
}
```

this allows us to deploy our code with a gulp task

* our `enums.js` which will help us manage the state of our skill, just to give us some semblance of type safety in JavaScript land
* a `gulpfile.js` that details our deployment tasks. It removes our old `dist` stuff, grabs all the stuff we care about and puts it in `dist`, zips it up because this is how AWS lambda takes multiple files and uses `node-aws-lambda` to upload our file using the `lambda-config` that has had it's values replaced with our secret ones from above
* `index.js` - the entry to our skill - it sets the environment path to help our AWS lambda traverse our files, registers our `appId` and `handlers` (which we'll come to) and kicks the whole thing off
* the `lambda-config.js` which will look very familiar to the screen in your [AWS lambda console](https://console.aws.amazon.com/lambda/home?region=us-east-1). Set these values as desired
* `package.json` which - I'm going to assume you know. The only real points to note are the scripts.
 * Finding a good way to deploy an Alexa skill is a bit of a PITA, this is [not a deployment workflow](https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/deploying-a-sample-skill-to-aws-lambda#preparing-a-nodejs-sample-to-deploy-in-lambda). I found these two examples ([Rick's](https://github.com/rickwargo/alexa-skill-template) and [Thoughtworks'](https://github.com/ThoughtWorksStudios/node-aws-lambda)) which I borrowed heavily from for my own `gulpfile`. However, they both like a `PROD` install whereas I've gone for a slightly different approach to avoid doing an install every time I deploy. When you `npm i` for your skill, it'll stick the `PROD` dependencies straight into `dist`.
 * I've also stuck `iron-node` in there, which I've had mixed experiences with, but is actually invaluable when you want to debug something easily in Chrome's dev tools. Please feel free to debug your node code your preferred way though, such as through your IDE.
* `responses.js` should be where you put anything Alexa says. I would do them as functions to have a consistent API where you can pass in values to be dynamically built. In fact I refactored these out to call the emit methods in order to store the last known reponses and states to help us continue after a "pause". Having all the responses in one place makes testing easier (as you can change the phrases without breaking your tests) and when i18n comes in, you can swap this file out rather than have hard coded text littered over your code base.

#### handlers

These are the boys who receive the `intents` (what Alexa thinks the user said based upon your `speech-assets`), perform some kind of business logic, perhaps store some stuff in session state (or dynamo db) and return a response (something for Alexa to say back). The state stuff is necessary because the skill is very dumb, it takes some JSON in, it sends some JSON out. It's because of this pure nature, TDD (and testing in general) is so easy. When we get to the `event-samples` later you'll see what that JSON looks like. It's a good idea to keep your handlers as skinny as possible and move the business logic into the `modules` folder.

I've gone for a rather rigid approach of commenting each handler's intent to ensure it follows the same pattern when edited. The pattern being, setup, state changes, response. As a member of the church of functional programming, it would have been preferable to get an immutable model in the intent, perform your logic, update that thing, then return it with your response. However, the SDK has gone for a more OO approach with heavy use of `this` to fire events and manage state...which is less than ideal and led to some wonky workarounds.

I advise having as many small handlers as you can manage, this will make handling ambiguous intents such as "yes" and "no" much easier and helps to organise the code and test.

In the starter-kit we only have three handlers:

* `core.handlers.js` - we'll get to "certification" at the end but for now you just need to know these are required, so we'll mix them in to all our other handlers. `AMAZON.StopIntent` is basically pause and `AMAZON.CancelIntent` is "get me out of here". The `SessionEndedRequest` is just a nice to have for logging purposes
* `new-session.handlers.js` - this is the skill entry point (as oppposed to `index.js` which is the code entry point), we'll set some state here (which decides which handler to go to) and tell our skill which intent to run. There's also the random "catch-all" which will log out something really useless...
* `stopped.handlers.js` - because `StopIntent` and `CancelIntent` are generally required, ideally we want one place to manage being "paused". Here we set the state back to the previous state if the user wishes to continue and exit if they...don't want to continue

#### modules

This is where your business logic should live. As mentioned above, all the heavy lifting should be in here. We don't want the `intents` to get too busy and dirty because it makes it hard to read and hard to maintain. I've included a small naïve mixin implementation in the `utils` module so we don't need to add the core intents over and over.

#### tests

None of the samples have tests. I don't believe you can actually develop and ship an Alexa skill without tests. There are so many paths and branches and intents that as soon as you add a modicum of complexity to your skill, you'll get bugs.

In the tests folder we have our fixtures known as `event-samples` and a giant test file. Now, you could probably split this up but it feels like a "pre-optimisation", I quite liked having all my skill test paths together. So we have some helpers at the top:

* `sanitise` just to make life easier if you've used a multi-line template string and want to assert on the speech output
* `getOutputSpeech` which just destructures the response, strips everything back and allows us to assert on what we care about, what Alexa said
* `getAttribute` which again is a nice to have little abstraction to pick out our session attributes to test on
* `runIntent` finally, but by no means least, this is the thing that will make all your testing a dream. It uses `aws-lambda-mock-context` to stub out all the context stuff we don't really care about and gives us a fluent API to invoke our intent with ease

I've gone for a heavily nested approach to give it the feel of a conversation when you read the output. So each `describe` is what the user would say, then the test name is what you think your skill should have done. As you can see we can now nicely destructure and assert in quite a clean manner. Also using fat arrows means less curly braces for each `it` for the win.

I would split the `event-samples` by handler state. This just makes them more manageable. This is an example `event-sample` which would be in the `game` directory:

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

You can see our game state comes in as `attributes`, along with the request which has a name and slot values. This is why it's a beauty for TDD purposes, we take this JSON in, we test the next state and the response, done.

A small note, you'll need Node 5.x.x to get the benefits of ES2015 but AWS lambdas only support Node 4.3.2 which has limited ES2015 support. In order to have a nice dev experience but still be able to test the skill locally, I use [nvm](https://github.com/creationix/nvm) to easily switch from one to t'other.


## Writing the thing

* [lambda](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions/Arithlistic?tab=code)
* [alexa console](https://developer.amazon.com/edw/home.html#/skills/list)
* [echoism](https://echosim.io/)
* [alexa app](http://alexa.amazon.com/spa/index.html#settings/dialogs)

Code stuff

First test

`responses.js`

```javascript
module.exports.gamePrelude = () =>
  'This is a dangerous game of cat and mouse in the even more dangerous forest, do you want to play?'
```

The flow diagram may differ from the actual response as you test and play around with how Alexa says things. She talks rather rapidly so you'll need a large supply of commas and other punctuation to keep her in check.

`enums.js`

```
GAME_START: 'GAME_START',
```

test

```
describe('Alexa, start game', () => {
  it('Respond with game prelude and set state to GAME_START', () =>
    runIntent(sessionStartIntent)
      .then(({ outputSpeech, gameState }) => {
        assert.deepEqual(outputSpeech, sanitise(gamePrelude()));
        assert.deepEqual(gameState, GAME_STATES.GAME_START);
      }));
});
```

test result

```
1) Alexa, start game Respond with game prelude and set state to GAME_START:
     Error: timeout of 2000ms exceeded. Ensure the done() callback is being called in this test.
```

`new-sessions.handlers.js`

```
const setStateAndInvokeEntryIntent = function() {
  // updates
  this.handler.state = GAME_STATES.GAME_START;

  // response
  this.emitWithState('GameIntro');
};
```

add game-start.handlers

`index.js`

```
const gameStartHandlers = require('./handlers/game-start.handlers');
```

Start game failures tests

```
'AMAZON.NoIntent': function() {
    res.tell.call(this, res.goodbye());
  },
```

For linting write out the function call rather than do it as a computed. Slightly ugly call to make this explicit. As mentioned would have been nicer to update some immutable mode and return that with a response. Perhaps the API would have been nicer if I had spread and gather so I could pass arbitary args and not have to invoke responses in the intent itself.

```
assert(endOfSession);
```

Copy and paste tests - DRY in tests is a balancing act

Core handlers takes care of cancel and stop for us


Next bit - fill in the responses

Nest responses. When doing it for real, make sure all paths do something, usually via the prompt text or the unhandled "catch-all".

Add a stopped event-samples directory and put in the yes/no. Add test for no to help but because we didn't change state we are just re-using the game start's yes and no intents. If you wanted to switch state to help and do a whole help thing then you'd obviously create a directory and test for the state switches back and forth.

Add yes intent making clear split between state/session changes and reponse - will pay dividends when intents get a bit fatter.

Adding handlers, adding fixtures, adding tests, switching states, adding assets and configuring in Alexa console.

Fill in magic key values, npm run deploy, test it out

Receiving slot values, performing business logic and responding

# Going live

* Implement every handler
* Write a test for every fail