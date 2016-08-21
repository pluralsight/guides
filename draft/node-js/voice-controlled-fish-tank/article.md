# control your fish tank from anywhere in the world with voice control

### Overview
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
