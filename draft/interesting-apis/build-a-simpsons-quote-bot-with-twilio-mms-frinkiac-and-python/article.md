I'm a huge fan of The Simpsons. In fact, every other sentance that comes out of my mouth is usually a Simpson's reference. As a result, I love social media pages like [@Simpsons_tweets](https://twitter.com/Simpsons_tweets), [The Best Simpsons Faces](https://www.facebook.com/TheBestSimpsonsFaces/) and especially [@SimpsonsQOTD](https://twitter.com/SimpsonsQOTD?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor).

But what am I to do if I don't have access to a computer or WiFi? How will I get my fill of Simpson's humor? Enter [Twilio](https://www.twilio.com), the API for text, VoIP, and voice in the cloud. 

In this blog post we are going to use Twilio along with [Frinkiac](https://frinkiac.com/), the Simpons quote and screencap database, to create a Python app that will automatically send us a Simpson's screencap and quote every day via [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service). We are going to accomplish this in less than 40 lines of Python. That's right: no cron jobs, no servers, pure Python.

If you don't want to follow along and just want to see the finished code, you can find it on my Github [here](https://github.com/Brodan/FrinkiacMMSBot).

# Getting Started

Before we can jump into code, we need to get our environment set up. 

Make sure you have [Python](https://www.python.org/) and [pip](https://pip.pypa.io/en/stable/#) installed on your machine.
Python comes pre-installed on many UNIX/Linux distributions, but if you don't have it you can download it from [here](https://www.python.org/downloads/). pip comes packaged with Python versions >=2.7.9 or >=3.4, otherwise you can download it from [here](https://pip.pypa.io/en/stable/installing/). You can test if Python and pip are both installed by running the following:
```
$ python -V && pip -V
```

Next we need to install a few Python libraries. pip makes this really easy for us. Run the following command which will automatically download and install all the external libraries our script will need:

```
$ pip install twilio requests schedule

```
[twilio](https://www.twilio.com/docs/libraries/python) is the official Python helper library that will help us interact with the Twilio API. [requests](http://docs.python-requests.org/en/master/) will make it very easy to send HTTP requests which will help us get data from Frinkiac. [schedule](https://pypi.python.org/pypi/schedule
) will make the painful task of job scheduling trivial and easily configurable. 

Lastly, make sure you have a Twilio account. If you don't, you can [sign up for free](https://www.twilio.com/try-twilio). You'll also need a [Twilio phone number](https://www.twilio.com/help/faq/phone-numbers) with SMS and MMS capabilities. You can check the capabilities of numbers on the [Phone Numbers Dashboard](https://www.twilio.com/console/phone-numbers/dashboard).

# Building Our App

Our setup is complete and we start building our app. We only need one file, so navigate to a directory of your choosing and open a new file called `frinkiac.py` in your preferred editor.

At the top of this file add the following lines:

```
import schedule
import requests
from twilio.rest import TwilioRestClient
from twilio import TwilioRestException

account_sid = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'
auth_token  = 'YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY'
client = TwilioRestClient(account_sid, auth_token)
```

```
$ export TWILIO_AUTH_TOKEN='YYYYYY'
$ export TWILIO_ACCOUNT_SID='XXXXX'

import os
os.environ.get('TWILIO_AUTH_TOKEN')
```


```
schedule.every(15).seconds.do(send_MMS)
# schedule.every().day.at("12:00").do(job)

# schedule.every().hour.do(job)
# schedule.every().day.at("10:30").do(job)
# schedule.every().monday.do(job)
# schedule.every().wednesday.at("13:15").do(job)
```
If your app crashes due to a `hostname doesn't match` error, it's because of an issue between the requests library and your Python version. You'll need to upgrade to Python >=2.7.9 or follow [this StackOverflow answer](https://stackoverflow.com/questions/18578439/using-requests-with-tls-doesnt-give-sni-support/18579484#18579484) to resolve this issue.


https://www.twilio.com/console


These lines of code were adapted from [Twilio's Python quickstart documentation](https://www.twilio.com/docs/quickstart/python/sms/sending-via-rest).

Thanks to schedule, we don't have to deal with setting up cron jobs.
Although Frinkiac doesn't have an official API, the entire site is react-based and fetches everything via HTTP API. We can use this to our advantage.

# Wrapping Up
Congratulations! You've just built a Twilio-powered MMS bot using nothing more than a few lines of Python. The tools in this post can be used in so many ways to create tons of awesome applications like this. Try combining new APIs and with some of Twilio's other features like [Voice](https://www.twilio.com/voice) or [IP Messaging](https://www.twilio.com/ip-messaging) to see what you can come up with!

If you enjoyed this post be sure to check out the [Twilio Blog](https://www.twilio.com/blog/) for an endless amount of articles like this. Also, feel free to check out my [personal blog](https://brodan.biz/blog/) for both technical and non-technical articles, or follow me on Twitter [@brodan_](https://twitter.com/Brodan_) to see when I publish new posts!


