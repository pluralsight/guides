### control your fish tank from anywhere in the world with Alexa voice control 

#### Overview
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d2cf9635-3fed-4415-b396-b20d2ec9223d.jpg)

I have a fish tank and I want to go on a vacation and need to feed the fish or sometimes I keep forgetting to feed them regularly or I 'm just too lazy to feed the fish but i don't want to see dead skeletons in my fish tank :D so from time to time i have to interrupt whatever i am doing and go to feed the fish twice a day everyday ...... why they just don't feed themselves :D so i came up with this idea i will make them feed themselves and guess want i won't touch the fish tank unless once a week to fill the food .

Magical , isn't it :D ? also i can control the fish tank lights remotely .
## How it Works
Thanks to this guy in the picture i call him "Mr Feeder"
and yes he 's a plastic cup with some decorations i made for him to look like a fish (i assume) :D .

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ab36f4a9-76d8-47b0-aba9-a40897d97350.jpg)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/383af858-996e-4d73-89e0-abe1cfb68eb8.jpg)

i can command alexa echo device saying "Alexa, ask fish tank to feed the fish" and mr feeder will thankfully do the rest of the job and by connecting it aws iot i can ask to alexa to feed the fish from anywhere in the world

the idea is that alexa send the command which is "feed the fish" to a raspberry pi through aws iot and raspberry pi controls the servo attached to "Mr feeder" and a relay module that turn lights on/off

and here is the result :

* [Final Fish Tank Demo on Youtube](https://youtu.be/1Uk0BqyBn34)

## Hardware used
- Alexa Amazon Echo
- Raspberry Pi 3 Model B
- Servo	
- Relay

## Online Sevices Used
	
- Alexa Skills Kit
- Amazon Web Services AWS Lambda
- Amazon Web Services AWS IoT

## Steps to Achieve this
### Step 1: Raspberry pi connection to servo motor and relay module

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d019e13a-04d4-4d76-b8b0-818b799e35d8.png)

first of all this is the Gpio pins for raspberry pi 3 we will need it to know how to connect the servo and relay module

1. the servo is attached to the plastic cup "Mr feeder" to rotate it with an opening that allows food to be dropped while rotation and the code makes 3 rotation for each command to be sure that enough food is dropped

the connection is as follow or look at the schematics below:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/655570d9-e60a-4887-96f1-dfcde23d60f6.jpg)

3 wires pwr , gnd , pulse of servo connected to pin 11

this is important because the code use this pin to control the servo this part of the code is
```javascript
import time;
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BOARD)
GPIO.setup(11,GPIO.OUT)
p =GPIO.PWM(11,50)
p.start(12.5)
count=0
try:
	while count<=3:
		p.ChangeDutyCycle(12.5)
		time.sleep(1)
		p.ChangeDutyCycle(2.5)
		time.sleep(1)
		count+=1	
except KeyboardInterrupt:
		p.stop()
		GPIO.cleanup()
```
testing the code

https://youtu.be/Ynqmf9tUODg

2. relay module code
```javascript
import time;
import RPi.GPIO as GPIO
GPIO.setmode(GPIO.BOARD)
GPIO.setup(11,GPIO.OUT)
GPIO.output(11,GPIO.HIGH)
time.sleep(2);
GPIO.output(11,GPIO.LOW)
time.sleep(2);
GPIO.cleanup()
```
testing relay

Note ( in testing same pin 11 was used but in the final code we use pin 13 as 11 was used by the servo)

https://youtu.be/lmbaIy4TDV4

connect the relay output to the lights wire it should be very easy , wire it all up as the schematic below and move to the next step

## Step 2 : AWS Lambda function
login to AWS or create an account and choose lambda service

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f3daf535-ec2f-463f-b323-a8f8c00bf3a7.png)

create a function

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/086a0564-f53e-4d54-b912-4528a32d88c5.png)

skip blueprints and click next


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/30b11605-c2ef-478b-b951-0a769df4db7c.42)

add alexa skill kit



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7e3f1a55-cee5-4f8b-ab44-fceeca47b000.43)

add function name and upload zip file from source code attached below and click next after configuring lambda role with basic execution role



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/075abe52-b04e-4fe8-aa05-cd1f8f48edfe.45)

review it and click create function


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b6b7977d-d8c6-4c37-8515-e3060cd5d824.48)


remember this section or take a note of ARN this is the endpoint to be used in next step



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/879f11df-f828-4c1f-a600-78d8676ea9fe.jpg)

## Step 3 : Alexa skill
login to amazon developer console or create an account then choose Alexa



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/e4175e93-c4d0-46b1-94e4-f46841e0a634.00)

click Alexa Skill kit

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b87989ee-95db-4226-ae04-7c1969d4ecdd.02)

add a new skill


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/271438c3-a4ff-4a8b-8ec5-5bfb13774ccd.02)

i Have already submitted the skill to certification so no testing and data will be filled in the pictures

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b89afaf0-e728-4744-aad2-fc5290303a15.03)

here we are defining interaction model and 2 intents are defined "feedintent" and "lightsIntent" with their sample utterance



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/9ae98539-32ab-476a-82bd-e00b039c789e.05)

configuration add ARN from previous step i have blurred it here


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/da480e69-41c2-4a69-9f57-ca9c88d2f23f.07)

publishing info


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/e716a90f-a80d-4407-b219-23ebf96b3519.08)

icons ( Isn't it cute :D )


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/5d8686c9-4fd9-4e73-9263-3497810ceccf.png)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/49b1937f-248d-4b7e-b9a8-db9c1e3fbb05.09)

now submit your skill to certification




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c694a81e-23b3-4d26-859a-710d9984db8b.10)


## Step 4 : AWS Iot
return to AWS console again and choose AWS Iot


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ed230c74-bca3-431a-9d77-1c609842c306.39)

click on create a resource

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8b1481c6-2f7e-423d-a7be-321ae53f55dd.12)

we need 3 resource : a thing ,a policy, and a certificate


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/18909bdd-6752-422e-9156-60308dcc73fc.13)

first create a thing name it fish_tank and click create

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/744d4212-29d4-4c6a-9fb5-25b18bf226f6.15)

create a certificate , tick activate and click 1 click certificate create

and download the certificates we will need to add them to the raspberry pi in folder called "certs" on Desktop to be able to connect to iot

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/72eadf27-910f-496b-9d5f-af2788f2d7ed.17)

create a policy as follows




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/d595bbdc-791c-4d17-92f8-0d2822c1f3cd.23)

click add statement then click create

now we have 3 resources we should attach the thing and policy both to the certificate we created so after clicking on the certificate click actions and choose attach policy and enter it's name then add thing and enter it's name

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7bc1ed88-339c-4b6a-99e8-186f3844f7c5.19)

now put the code file fishtank.py attached below on your /home/pi/Desktop/ destination and create folder ins same destination called certs if you haven't created it with certificates downloaded inside it and change their names to be used as in the code

for rootCA.key you can download it from this link
```
https://www.symantec.com/content/en/us/enterprise/verisign/roots/VeriSign-Class%203-Public-Primary-Certification-Authority-G5.pem 

```
their names should be as follows :

```
root_cert = cert_path + "rootCA.key"
cert_file = cert_path + "cert.pem.crt"
key_file = cert_path + "private.pem.key"
```

#### finally all is done hope you enjoy it :D

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/7cf2d951-f0b9-4a3a-a4b1-d7983b3f3ee9.png)

## Schematics

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4caf20fc-62d1-4ef8-967a-86e5ecf8da69.PNG)


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b2a56d18-f7ae-4a62-92e7-2fb249be7729.jpg)

## source code files used :

1.fishtank.py 
```
#!/usr/bin/python

import time;
import paho.mqtt.client as mqtt
import json, time, sys, ssl
import RPi.GPIO as GPIO  

lights = 13
feeder = 11


GPIO.setmode(GPIO.BOARD)

GPIO.setup(feeder, GPIO.OUT)
GPIO.setup(lights, GPIO.OUT)

#GPIO.output(feeder, GPIO.HIGH)
#GPIO.output(lights, GPIO.HIGH)


cert_path = "/home/pi/Desktop/certs/"
host = "ENDPOINT.amazonaws.com"
topic = "$aws/things/fishtank/shadow/update"
root_cert = cert_path + "rootCA.key"
cert_file = cert_path + "cert.pem.crt"
key_file = cert_path + "private.pem.key"

globalmessage = ""  # to send status back to MQTT
isConnected = False

def feedcommand(data):  #   {"name":"FeedIntent","slots":{"Task":{"name":"Task","value":"feed the fish"}}}
    task = str(data["slots"]["Task"]["value"])
    global globalmessage

    if task == "feed the fish":
        globalmessage = "now the fish is no longer hungry"
        print globalmessage
        toggle(feeder)

def lightscommand(data):  #   {"name":"LightsIntent","slots":{"Task":{"name":"Task","value":"on"}}}
    try:
        state = str(data["slots"]["state"]["value"])
        global globalmessage

        globalmessage = "Turning lights " + state
        print globalmessage

        if state == "on":
         GPIO.output(lights, GPIO.HIGH)
        elif state == "off":
         GPIO.output(lights, GPIO.LOW)      
    except (ValueError):
        print "lights Error"
    
def toggle(gpio):
	
	p =GPIO.PWM(11,50)
	p.start(12.5)
	count=0
	try:
		while count<=3:
			p.ChangeDutyCycle(12.5)
			time.sleep(1)
			p.ChangeDutyCycle(2.5)
			time.sleep(1)
			count+=1	
	except KeyboardInterrupt:
					
			p.stop()
			GPIO.cleanup()	
  #  GPIO.output(gpio, cmd)   call servo motor
        
# The callback for when the client receives a CONNACK response from the server.
def on_connect(client, userdata, flags, rc):
    isConnected = True
    # Subscribing in on_connect() means that if we lose the connection and      
    # reconnect then subscriptions will be renewed.    
    client.subscribe(topic)

# The callback for when a PUBLISH message is received from the server.
def on_message(client, userdata, msg):
    data = json.loads(str(msg.payload))
    # INTENT
    if "name" in data:
        if data["name"] == "FeedIntent":
            feedcommand(data)
        elif data["name"] == "LightsIntent":
            lightscommand(data)
        


client = mqtt.Client(client_id="fishtank.py")
client.on_connect = on_connect
client.on_message = on_message
client.on_log = on_log
client.tls_set(root_cert,
               certfile = cert_file,
               keyfile = key_file,
               cert_reqs=ssl.CERT_REQUIRED,
               tls_version=ssl.PROTOCOL_TLSv1_2,
               ciphers=None)

client.connect(host, 8883, 60)

run = True

try:
    while run:
        client.loop()
        time.sleep(1)

        try:
            mypayload = '''{
                "StatusMessage": "%s"
            }''' % (globalmessage)

            if globalmessage != "":
                client.publish(topic, mypayload)
                globalmessage = ""

        except (TypeError):
            pass
    
except KeyboardInterrupt:
    print "Bye Bye!"
    GPIO.cleanup()    
```
2.alexa skill intent scheme
```
{
   "intents":[
      {
         "intent":"FeedIntent",
         "slots":[
            {
               "name":"Task",
               "type":"LITERAL"
            }
         ]
      },
      {
         "intent":"LightsIntent",
         "slots":[
            {
               "name":"state",
               "type":"LITERAL"
            }
         ]
      }
   ]
}
```

3.Alexa skill Sample utterance

```
FeedIntent {feed the fish | Task} 
LightsIntent turn lights {on|state} 
LightsIntent turn lights {off| state}
LightsIntent lights {on| state}
LightsIntent lights {off| state}
```

4.index.js (lambda function)

```
/**
 * Control your fish tank with your voice, using Amazon Alexa, Lambda, IOT, MQTT.
 */

var awsIot = require('aws-iot-device-sdk');

var host = "EndPOINT.amazonaws.com";
var topic = "$aws/things/fishtank/shadow/update";
var app_id = "YOUR APP ID"
var deviceName = "FishTank";

var mqtt_config = {
    "keyPath": "./certs/private.pem.key",
    "certPath": "./certs/cert.pem.crt",
    "caPath": "./certs/rootCA.key",
    "host": host,
    "port": 8883,
    "clientId": deviceName, 
    "region":"us-east-1",
    "debug":true
};

var ctx = null;
var client = null;

// Route the incoming request based on type (LaunchRequest, IntentRequest, etc.) The JSON body of the request is provided in the event parameter.
exports.handler = function (event, context) {
    try {
        console.log("event.session.application.applicationId=" + event.session.application.applicationId);
        ctx = context;

        if (event.session.application.applicationId !== app_id) {
             ctx.fail("Invalid Application ID");
         }

        client = awsIot.device(mqtt_config);

        client.on("connect",function(){
            console.log("Connected to AWS IoT");
        });


        if (event.session.new) {
            onSessionStarted({requestId: event.request.requestId}, event.session);
        }

        if (event.request.type === "LaunchRequest") {
            onLaunch(event.request, event.session);
        }  else if (event.request.type === "IntentRequest") {
            onIntent(event.request, event.session);
        } else if (event.request.type === "SessionEndedRequest") {
            onSessionEnded(event.request, event.session);
            ctx.succeed();
        }
    } catch (e) {
        console.log("EXCEPTION in handler:  " + e);
        ctx.fail("Exception: " + e);
    }
};

/**
 * Called when the session starts.
 */
function onSessionStarted(sessionStartedRequest, session) {
    console.log("onSessionStarted requestId=" + sessionStartedRequest.requestId + ", sessionId=" + session.sessionId);
}


/**
 * Called when the user launches the skill without specifying what they want.
 */
function onLaunch(launchRequest, session, callback) {
    console.log("onLaunch requestId=" + launchRequest.requestId + ", sessionId=" + session.sessionId);

    // Dispatch to your skill's launch.
    getWelcomeResponse(callback);
}

/**
 * Called when the user specifies an intent for this skill.
 */
function onIntent(intentRequest, session ) {
    console.log("onIntent requestId=" + intentRequest.requestId + ", sessionId=" + session.sessionId);

    var intent = intentRequest.intent,
    intentName = intentRequest.intent.name;

    console.log("REQUEST to string =" + JSON.stringify(intentRequest));

    var callback = null;
    // Dispatch to your skill's intent handlers
    if ("FeedIntent" === intentName) {
        feedIntent(intent, session);
    }
    else if ("LightsIntent" === intentName) {
        ligthsIntent(intent, session);
    } else {
        throw "Invalid intent";
    }

}

/**
 * Called when the user ends the session.
 * Is not called when the skill returns shouldEndSession=true.
 */
function onSessionEnded(sessionEndedRequest, session) {
    console.log("onSessionEnded requestId=" + sessionEndedRequest.requestId + ", sessionId=" + session.sessionId);
    // Add cleanup logic here
}

// --------------- Functions that control the skill's behavior -----------------------

function getWelcomeResponse() {
    // If we wanted to initialize the session to have some attributes we could add those here.
    var sessionAttributes = {};
    var cardTitle = "Welcome";
    var speechOutput = "Welcome to the Fish Tank.";

    var repromptText = "Fish Tank  is ready for command.";
    var shouldEndSession = false;

    ctx.succeed(buildResponse(sessionAttributes, buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession)));
}


function feedIntent(intent, session, callback) {
    var repromptText = null;
    var sessionAttributes = {};
    var shouldEndSession = true;
    var speechOutput = "";

    repromptText = "Tell me what to do with the fish tank.";

    var task = intent.slots.Task.value;
    var validTasks = [ "feed the fish"];

    if (validTasks.indexOf(task) == -1)
    {
        speechOutput = "I couldn't understand the command.  ";
        ctx.succeed(buildResponse(sessionAttributes, buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession)));
    }
    else
    {
        var cardTitle = "Feeding the Fish"  ;
        speechOutput = "Feeding the Fish" ;
        mqttPublish(intent, sessionAttributes, cardTitle, speechOutput, repromptText, shouldEndSession);
    }
}

function ligthsIntent(intent, session, callback) {

    var repromptText = null;
    var sessionAttributes = {};
    var shouldEndSession = false;
    var speechOutput = "";

    repromptText = "Tell me how you want the fish tank lights.  ";

    var state = intent.slots.state.value;
    var validstates = [ "on", "off" ];

    if (validstates.indexOf(state) == -1)
    {
        speechOutput = "I couldn't understand the state of the lights. ";
        ctx.succeed(buildResponse(sessionAttributes, buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession)));
    }
    else
    {
        var cardTitle = "fish tank lights turning " + state;
        speechOutput = "Turning lights " + state;
        mqttPublish(intent, sessionAttributes, cardTitle, speechOutput, repromptText, shouldEndSession);
    }
}

function mqttPublish(intent, sessionAttributes, cardTitle, speechOutput, repromptText, shouldEndSession)
{
    var strIntent = JSON.stringify(intent);
    console.log("mqttPublish:  INTENT text = " + strIntent);
    
    client.publish(topic, strIntent, function() {
        client.end();
    });
    
    client.on("close", (function () {
        console.log("MQTT CLIENT CLOSE - thinks it's done, successfully. ");
        ctx.succeed(buildResponse(sessionAttributes, buildSpeechletResponse(cardTitle, speechOutput, repromptText, shouldEndSession)));
    }));

    client.on("error", (function (err, granted) {
        console.log("MQTT CLIENT ERROR!!  " + err);
    }));
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
            title: title,
            content: output
        },
        reprompt: {
            outputSpeech: {
                type: "PlainText",
                text: repromptText
            }
        },
        shouldEndSession: shouldEndSession
    }
}

function buildResponse(sessionAttributes, speechletResponse) {
    return {
        version: "1.0",
        sessionAttributes: sessionAttributes,
        response: speechletResponse
    }
}
```

