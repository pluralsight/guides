I'm a huge fan of The Simpsons. In fact, every other sentence that comes out of my mouth is usually a Simpson's reference. I also love social media pages like [@Simpsons_tweets](https://twitter.com/Simpsons_tweets), [The Best Simpsons Faces](https://www.facebook.com/TheBestSimpsonsFaces/), and especially [@SimpsonsQOTD](https://twitter.com/SimpsonsQOTD?ref_src=twsrc%5Egoogle%7Ctwcamp%5Eserp%7Ctwgr%5Eauthor).

But what am I to do if I don't have access to a computer or wi-fi? How will I get my fill of Simpson's humor? Enter [Twilio](https://www.twilio.com), the API for text, VoIP, and voice in the cloud. 

In this tutorial, we are going to use Twilio along with [Frinkiac](https://frinkiac.com/), the Simpons quote and screencap database, to create a Python app that will automatically send us a Simpson's screencap and quote every day via [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service). **We are going to accomplish this in less than 40 lines of Python.** Yup, no cron jobs, no servers, just pure Python.

If you don't want to follow along and just want to see the finished code, check out my Github [repository.](https://github.com/Brodan/FrinkiacMMSBot).

# Getting Started

![Press "Any" Key](https://media.giphy.com/media/3orif0rjs49gsPWg1y/giphy.gif)

Before we can jump into code, we need to get our environment set up. 

### Python and pip

Make sure you have [Python](https://www.python.org/) and [pip](https://pip.pypa.io/en/stable/#) installed on your machine. Python comes pre-installed on many UNIX/Linux distributions, but if you don't have it you can download it from [here](https://www.python.org/downloads/). pip comes packaged with Python versions >=2.7.9 or >=3.4. 
You can test if Python and pip are both installed by running the following:
```
$ python -V && pip -V
```
If you don't think you have pip on your system, [download it here](https://pip.pypa.io/en/stable/installing/). 

### Python libraries

Next we need to install a few Python libraries. pip makes this really easy for us. Run the following command which will automatically download and install all the external libraries our script will need:

```
$ pip install twilio requests schedule

```
Let's break down the command we just used. 

[`twilio`](https://www.twilio.com/docs/libraries/python) is the official Python helper library that will help us interact with the Twilio API. [`requests`](http://docs.python-requests.org/en/master/) will make it very easy to send HTTP requests, which will ultimately help us get data from Frinkiac. [`schedule`](https://pypi.python.org/pypi/schedule) will make the painful task of job scheduling easily configurable. 

### Twilio account

Lastly, make sure you have a Twilio account. If you don't, you can [sign up for free](https://www.twilio.com/try-twilio). You'll need a [Twilio phone number](https://www.twilio.com/help/faq/phone-numbers) with SMS and MMS capabilities. You can check the capabilities of numbers on the [Phone Numbers Dashboard](https://www.twilio.com/console/phone-numbers/dashboard). Once that's all set up, you're ready to start building the quote-bot.

# Building Our App

It's time to start building our app. We only need one file, so navigate to a directory of your choosing and open a new file called `frinkiac.py` in your preferred editor.

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

The first four lines in the code above are simply importing all of the libraries we just installed. The three lines after our imports configure and create a `TwilioRestClient` object that will let us make calls to the [Twilio REST API](https://www.twilio.com/docs/api/rest). Make sure you replace the values for `account_sid` and `auth_token` with your actual account SID and auth token. You can find these values in your [Twilio account dashboard](https://www.twilio.com/console).

**Important note**: Never push code with your API credentials to a public repository. See the "Optional Steps" section at the bottom of this post for an alternative approach to using your Twilio API keys.

Next add the following function to your app:

```
def get_quote():
    r = requests.get("https://frinkiac.com/api/random")
    if r.status_code == 200:
        json = r.json()
        # Extract the episode number and timestamp from the API response
        # and convert them both to strings.
        timestamp, episode, _ = map(str, json["Frame"].values())

        image_url = "https://frinkiac.com/meme/" + episode + "/" + timestamp
        # Combine each line of subtitles into one string.
        caption = "\n".join([subtitle["Content"] for subtitle in json["Subtitles"]])
        return image_url, caption
```

The function we just added uses `requests` to send a GET request to Frinkiac and retreive data about a random Simpsons moment. Although Frinkiac isn't *actually* an API, the entire site is react-based and fetches resources via HTTP. As such, we can use the site in the same way that we would use an API. Next, we convert our retrieved data into [JSON](http://www.json.org/), extract the `timestamp` and `episode` code, and convert these two components into string format. `timestamp` and `episode` are used to create the URL that points to a screencap of the random Simpsons moment. Finally, we grab the contents from each line of subtitles in our JSON and join them together to form the `caption`.

Now add the only other function that we need:
```
def send_MMS():
    media, body = get_quote()
    try:
        message = client.messages.create(
            body=body,
            media_url=media,
            to="+12345678901",    # Replace with your phone number
            from_="+12345678901") # Replace with your Twilio number
        print("Message sent!")
    # If an error occurs, print it out.
    except TwilioRestException as e:
        print(e)
```
This function starts by calling the `get_quote` function we created in the previous step and by storing its return values. The `try`/`except` block from above was adapted from [Twilio's Python quickstart documentation](https://www.twilio.com/docs/quickstart/python/sms/sending-via-rest). These lines are simply taking in a number of parameters and turning them into a call to the Twilio REST API.

_Replace the `to` and `from` parameters with your phone real phone number and your Twilio phone number, respectively._
If an error occurs during the API call it will be printed to the terminal.


Now at the bottom of our file, below the two functions we just added, insert the following three lines:
```
schedule.every().day.at("12:00").do(send_MMS)

while True:
    schedule.run_pending()
```
`schedule` allows you to set how often a function runs in a very readable way. The schedule will run inside a `while` loop that will continue looping indefinitely. Our app will now behave according to the schedule, which means that `send_MMS` will be called **every day at 12:00 p.m.** indefinitely or until you exit the app.

# Testing Our App
For the purpose of testing the application, it's a good idea to change the schedule we added above to run more frequently. For example, ```schedule.every(30).seconds.do(send_MMS)``` would call the `send_MMS` function every 30 seconds. (This way you won't need to wait until noon to know if your application is working.)

After you've changed that, make sure you save `frinkiac.py`. Go back to your terminal and run the following command:

```
$ python frinkiac.py
```
**Your terminal will look like its frozen, but that's because your app is running.** After 30 seconds, you should see a line printed to your terminal that says `Message sent!`. See the "Optional Steps" section for instructions on running your program as a background process.

If your app crashes due to a `hostname doesn't match` error, it's because of an issue between the `requests` library and your Python version. Upgrade to Python >=2.7.9 or follow [this StackOverflow answer](https://stackoverflow.com/questions/18578439/using-requests-with-tls-doesnt-give-sni-support/18579484#18579484) to resolve this issue. 

If you run into any errors with Twilio, you will see a number of helpful tips printed to the terminal about how to resolve your issue.

Otherwise, check your phone and you should expect to see an MMS with a random Simpsons screencap and caption! Here's an example:

![S10E23](https://frinkiac.com/img/S11E02/921960.jpg)

*Okay, so you say your son is towheaded, button nose, mischievous smile, and may be armed with a slingshot?*

# Wrapping Up

Congratulations! You've just built a Twilio-powered MMS Simpsons quote-bot using nothing more than a few lines of Python. The tools in this guide -- using the `requests`, `twilio`, and `schedule` libraries, and more -- can be used in numerous ways to create applications with even more functionality. Try combining new APIs and libraries with some of Twilio's other features like [Voice](https://www.twilio.com/voice) or [IP Messaging](https://www.twilio.com/ip-messaging), and see what you can come up with!

If you enjoyed this post be sure to check out the [Twilio Blog](https://www.twilio.com/blog/) for an endless string of interesting articles and fun projects. Also, feel free to check out my [personal blog](https://brodan.biz/blog/) for both technical and non-technical articles, and follow me on Twitter [@brodan_](https://twitter.com/Brodan_) to see when I publish new posts!


# Optional Steps

* An alternative approach to configuring your `TwilioRestClient` object is to use environment variables. **With environment variables you won't have to worry about making your API keys visible to the public.** To do this, run the following commands in your terminal, _replacing the values with your actual account SID and auth token_:
```
$ export TWILIO_ACCOUNT_SID='XXXXX'
$ export TWILIO_AUTH_TOKEN='YYYYYY'
```
Then open `frinkiac.py` and add `import os` to the top of the file and replace `account_sid = 'XXXXXXX'` and `auth_token  = 'YYYYYYYY'` with the lines below:
```
account_sid = os.environ.get('TWILIO_ACCOUNT_SID')
auth_token = os.environ.get('TWILIO_AUTH_TOKEN')
```

* As mentioned earlier, using `schedule` is very intuitive. Some alternative schedules you could use in your app are:
```
schedule.every().hour.do(send_MMS)
schedule.every().monday.do(send_MMS)
schedule.every().wednesday.at("16:00").do(send_MMS)
```

* If you'd like to run your application as a background process, simply add `&` to the end of your `python` command:
```
$ python frinkiac.py &
[1] 1872
```
This command will return a process ID (PID) number (`1872` in this case) that you can append to the `kill` command in order to terminate your program (e.g. `$ kill 1872`).

___
Thank you for reading my tutorial on creating a Simpsons-based Quote-Bot using Twilio MMS, Frinkiac, and (some) Python. I hope that you found this guide informative and entertaining. Please leave your comments and feedback in the discussion section below, and please favorite this article as well!

