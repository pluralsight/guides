I'm a huge fan of The Simpsons. In fact, every other sentance that comes out of my mouth is usually a Simpson's reference. As a result, I love social media pages like [@Simpsons_tweets](https://twitter.com/Simpsons_tweets), [The Best Simpsons Faces](https://www.facebook.com/TheBestSimpsonsFaces/) and especially [@SimpsonsQOTD](https://twitter.com/SimpsonsQOTD?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor).

But what am I to do if I don't have access to a computer or WiFi? How will I get my fill of Simpson's humor?

Enter [Twilio](https://www.twilio.com), the API for text, VoIP, and voice in the cloud. In this blog post we are going to use Twilio along with [Frinkiac](https://frinkiac.com/), the Simpons quote and screencap database, to create a Python script that will automatically send us a Simpson's screencap and quote every day via [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service). We are going to do all of this in less than 40 lines of Python.


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
[twilio]() is obviously  [requests](http://docs.python-requests.org/en/master/) makes it very easy to send HTTP requests which will help us get data from Frinkiac. [schedule](https://pypi.python.org/pypi/schedule
) makes the painful task of job scheduling trivial and easily configurable. 



https://www.twilio.com/docs/libraries/python
$ pip install twilio

This code is taken from https://www.twilio.com/docs/quickstart/python/sms/sending-via-rest

MMS messages can only be sent and received by numbers having MMS capability. You can check the capabilities of numbers in the account portal or query the Available Phone Numbers resource to search for Twilio numbers that are MMS enabled.

$ export TWILIO_AUTH_TOKEN='YYYYYY'
$ export TWILIO_ACCOUNT_SID='XXXXX'

import os
os.environ.get('TWILIO_AUTH_TOKEN')

```
schedule.every(15).seconds.do(send_MMS)
# schedule.every().day.at("12:00").do(job)

# schedule.every().hour.do(job)
# schedule.every().day.at("10:30").do(job)
# schedule.every().monday.do(job)
# schedule.every().wednesday.at("13:15").do(job)
```

https://frinkiac.com/meme/S03E08/1219993.jpg




https://twitter.com/Simpsons_tweets

http://docs.python-requests.org/en/master/



http://docs.python-requests.org/en/master/community/faq/#what-are-hostname-doesn-t-match-errors

# Building Our App

Thanks to schedule, we don't have to deal with setting up cron jobs.
Although Frinkiac doesn't have an official API, the entire site is react-based and fetches everything via HTTP API. We can use this to our advantage.

# Wrapping Up



