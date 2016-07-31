### Important information about how Alexa works. - WORK IN PROGRESS!

Alexa is based on a question->response similar to Siri and Google Now services.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8908f30c-1e20-4de2-add1-8efa12f06544.png)

To create a skill we will need to **Create an Alexa skill** to capture the request like "Alexa, Ask HackGuides what time is it?"

This will then get translated from the Alexa Skill API into a **method call into the Amazon AWS Lambda Services** that is is then sent back to the user through the Alexa Skill.

![Alexa Question Response](https://developer.amazon.com/public/binaries/content/gallery/developerportalpublic/solutions/alexa/alexa-voice-service/images/avs_getting_started_1.png)



# Lets get started

To create a skill you will need to create for [developer.amazon.com:](https://developer.amazon.com/edw/home.html#/skills/lis): 

Now that you have an account you can begin to [Add a new Skill](https://developer.amazon.com/edw/home.html#/skills/list): 

![Alexa Skill Kit](https://raw.githubusercontent.com/pluralsight/guides/master/images/cc72e1e7-057d-48fa-9609-63fbe3f8da87.png)

This is the list of all your skills and you can click the top right button to 
![Add New SKill](https://raw.githubusercontent.com/pluralsight/guides/master/images/f7e4d0bc-ae2d-428d-8622-931bd22f693b.png)


#### Skill Information

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/598b8c4b-a0c9-4c6d-bc4d-563797d8c97a.png)
- Update the Name with "Hack Guides SKill Demo"
- Update the Invocation Name with "code hack" (so it's easy to pronounce)

#### Interaction Model


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f35c7195-4064-408d-9c4a-e62ab30d54f1.png)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/0f18ac5d-5c73-4e74-a902-0673a5cb7d5b.png)


#### Back to "Configuration" in Alexa SKill Kit
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/71d28c9b-84ca-4588-88bf-5232bd2d80b0.png)

#### Test



```
{
  "intents": [
    {
      "intent": "GetChoresIntent",
      "slots": [
        {
          "name": "firstName",
          "type" : "AMAZON.US_FIRST_NAME"
        }
      ]
    },
    {
      "intent": "AMAZON.HelpIntent"
    },
    {
      "intent": "AMAZON.StopIntent"
    },
    {
      "intent": "AMAZON.CancelIntent"
    }
  ]
}
```
 
```
GetChoresIntent what chore is for {firstName}
GetChoresIntent {firstName} needs a chore
GetChoresIntent give me a chore for {firstName}
```

#### Configuration

- First we need to create a Lambda Expression before filling out this part
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/59909fc3-4afb-49a6-99f6-1725b16ae559.png)

Navigate to this website and create the necessary account [Lambda Expression Website](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions)

You can see your list of existing functions or ![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/201da99d-b3ef-4a5d-b639-ac813ac31c3c.png)

#### Select Blueprint
Search for "Alexa" and click on the title of the "alexa-skills-kit-color-expert"![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f49ec59a-74e5-4cdf-b92b-ea7fe32354a0.png)

#### Configure event sources

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/2c034e09-f330-4b49-9cba-e99e89a1bdd2.png)


#### Configure Function

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c1862f5f-032d-4549-a970-c5d4a06d05ee.png)

You will probably need to create a new Role so click to "Create new role".

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4cf7caf9-e354-4770-9f97-af10491e91e7.png)

#### Review

Finally you are read with your lambda expression and click: 
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/a4549162-5ab2-4e9f-98ed-05081cf36538.png)

In the top right corner of your Lambda Function you will now have a ARN that provides the connction for Alexa Skill. 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4f765305-f849-4e49-8811-a2a8e30f1f42.png)



```
// Route the incoming request based on type (LaunchRequest, IntentRequest,
// etc.) The JSON body of the request is provided in the event parameter.
exports.handler = function (event, context) {
    try {
        console.log("event.session.application.applicationId=" + event.session.application.applicationId);

        /**
         * Uncomment this if statement and populate with your skill's application ID to
         * prevent someone else from configuring a skill that sends requests to this function.
         */
        /*
        if (event.session.application.applicationId !== "amzn1.echo-sdk-ams.app.[unique-value-here]") {
             context.fail("Invalid Application ID");
        }
        */

        if (event.session.new) {
            onSessionStarted({requestId: event.request.requestId}, event.session);
        }

        if (event.request.type === "LaunchRequest") {
            onLaunch(event.request,
                event.session,
                function callback(sessionAttributes, speechletResponse) {
                    context.succeed(buildResponse(sessionAttributes, speechletResponse));
                });
        } else if (event.request.type === "IntentRequest") {
            onIntent(event.request,
                event.session,
                function callback(sessionAttributes, speechletResponse) {
                    context.succeed(buildResponse(sessionAttributes, speechletResponse));
                });
        } else if (event.request.type === "SessionEndedRequest") {
            onSessionEnded(event.request, event.session);
            context.succeed();
        }
    } catch (e) {
        context.fail("Exception: " + e);
    }
};

/**
 * Called when the session starts.
 */
function onSessionStarted(sessionStartedRequest, session) {
    console.log("onSessionStarted requestId=" + sessionStartedRequest.requestId +
        ", sessionId=" + session.sessionId);
}

/**
 * Called when the user launches the skill without specifying what they want.
 */
function onLaunch(launchRequest, session, callback) {
    console.log("onLaunch requestId=" + launchRequest.requestId +
        ", sessionId=" + session.sessionId);

    // Dispatch to your skill's launch.
    getWelcomeResponse(callback);
}

/**
 * Called when the user specifies an intent for this skill.
 */
function onIntent(intentRequest, session, callback) {
    console.log("onIntent requestId=" + intentRequest.requestId +
        ", sessionId=" + session.sessionId);

    var intent = intentRequest.intent,
        intentName = intentRequest.intent.name;

    // Dispatch to your skill's intent handlers
    if ("GetChoresIntent" === intentName) {
        getChoreResponse(intent, session, callback);
    } else if ("AMAZON.HelpIntent" === intentName) {
        getWelcomeResponse(callback);
    } else if ("AMAZON.StopIntent" === intentName || "AMAZON.CancelIntent" === intentName) {
        handleSessionEndRequest(callback);
    } else {
        throw "Invalid intent";
    }
}

/**
 * Called when the user ends the session.
 * Is not called when the skill returns shouldEndSession=true.
 */
function onSessionEnded(sessionEndedRequest, session) {
    console.log("onSessionEnded requestId=" + sessionEndedRequest.requestId +
        ", sessionId=" + session.sessionId);
    // Add cleanup logic here
}

// --------------- Functions that control the skill's behavior -----------------------

function getWelcomeResponse(callback) {
    // If we wanted to initialize the session to have some attributes we could add those here.
    var sessionAttributes = {};
    var cardTitle = "Welcome";
    var speechOutput = "Welcome to the Hack Guides demo. " +
        "Please ask for a chore by saying: give me a chore for name";
    var repromptText = "Please ask for a chore by saying: give me a chore for name";
    var shouldEndSession = false;

    callback(sessionAttributes,
        buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession));
}

function handleSessionEndRequest(callback) {
    var cardTitle = "Session Ended";
    var speechOutput = "Thank you for trying the Hack Guides demo. Have an Amazon day!";
    // Setting this to true ends the session and exits the skill.
    var shouldEndSession = true;

    callback({}, buildSpeechletResponse(cardTitle, speechOutput, null, shouldEndSession));
}

/**
 * Sets the color in the session and prepares the speech to reply to the user.
 */
function getChoreResponse(intent, session, callback) {
    var cardTitle = intent.name;
    var fistNameSlot = intent.slots.firstName;
    var repromptText = "";
    var sessionAttributes = {};
    var shouldEndSession = false;
    var speechOutput = "";

    if (fistNameSlot) {
        
        var miscResponses = [
    'I think <break time="1s"/> {firstNameSlot} should work on Sweeping and moping!',
    'I think {firstNameSlot} should work on <break time="1s"/> dishes!',
    '{firstNameSlot}! should work on making their bed!',
    'Maybe, nope definitely {firstNameSlot} should work on vacuuming the house',
    'I think this time {firstNameSlot} should work on cleaning the toilets!',
    '{firstNameSlot} should <break time="1s"/> work on clearing the table'
    ];
        
        var firstNameValue = fistNameSlot.value;
        var randomInt = getRandomInt(0, miscResponses.length - 1);
        console.log(randomInt);
        speechOutput = miscResponses[randomInt].replace(/{firstNameSlot}/g, firstNameValue);
        repromptText = "Please ask for a chore by saying: give me a chore for name";
    } else {
        speechOutput = "I didn't catch that name you tell me again";
        repromptText = "Please ask for a chore by saying: give me a chore for name";
    }

    callback(sessionAttributes,
         buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession));
}

function getRandomInt(min, max) {
    console.log("max:" + max + " min:" + min);
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

// --------------- Helpers that build all of the responses -----------------------

function buildSpeechletResponse(title, output, repromptText, shouldEndSession) {
    return {
        outputSpeech: {
            type: "PlainText",
            text: output
        },
        card: {
            type: "Simple",
            title: "SessionSpeechlet - " + title,
            content: "SessionSpeechlet - " + output
        },
        reprompt: {
            outputSpeech: {
                type: "PlainText",
                text: repromptText
            }
        },
        shouldEndSession: shouldEndSession
    };
}

function buildResponse(sessionAttributes, speechletResponse) {
    return {
        version: "1.0",
        sessionAttributes: sessionAttributes,
        response: speechletResponse
    };
}
```



#### Test





Alexa has been made popular by the [Amazon Echo](https://www.amazon.com/Amazon-Echo-Bluetooth-Speaker-with-WiFi-Alexa/dp/B00X4WHP5E), [Amazon Tap](https://www.amazon.com/dp/B01BH83OOM), [Amazon Dot](https://www.amazon.com/b/?node=14047587011), and now [Pebble Core](https://blog.getpebble.com/2016/06/02/ks3u03/)



QUICK URLS and ACCOUNTS NECCESSARY: 

https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit#why-build-skills ** Reasons to build skills**
https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit # **Developer Portal**
https://developer.amazon.com/edw/home.html#/skills/list **List of your skills**
https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions **Lambda functions to support Alexa**
