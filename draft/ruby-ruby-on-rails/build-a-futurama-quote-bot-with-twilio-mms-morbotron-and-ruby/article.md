Back in August I wrote a post for the hack.guides() Tutorial Contest about [building a Simpsons quote-bot with Twilio MMS, Frinkiac, and Python](http://tutorials.pluralsight.com/interesting-apis/build-a-simpsons-quote-bot-with-twilio-mms-frinkiac-and-python).
I enjoyed writing that post so much that I decided to rewrite it in a language less familiar to me: [Ruby](https://www.ruby-lang.org/). I've also swapped out Frinkiac for [Morbotron](https://morbotron.com/), a similar database made by the same team.


In this tutorial, we are going to use Twilio along with Morbotron, the Futurama quote and screencap database, to create a Ruby app that will automatically send us a Futurama screencap and quote every day via [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service). **We are going to accomplish this in exactly 50 lines of Ruby.** Yup, no cron jobs, no servers, just Ruby.

If you don't want to follow along and just want to see the finished code, check out the Github [repository.](https://github.com/Brodan/MorbotronMMSBot).

# Getting Started


Before we can jump into code, we need to get our environment set up. 

### Ruby and gem


### Ruby libraries

Next we need to install a few Ruby gems. We are going to use bundler to take care of this. Run the following command which will automatically download and install all the external libraries our script will need:

https://rvm.io/rvm/install

```
$ rvm install 2.3

$ gem install bundler
 you make be asked for your password depending on user priveleges


```
if you're new to ruby and would like enable tab completeion: Basically, just create a file in your home directory called .irbrc. Add the following two lines and save it:

require 'irb/completion'

http://bundler.io/

https://github.com/jmettraux/rufus-scheduler#first_at-first_in-first-first_time

Let's break down the command we just used. 
['rufus-scheduler'](https://github.com/jmettraux/rufus-scheduler)
['unirest'](http://unirest.io/ruby.html)
['twilio-ruby'](https://github.com/twilio/twilio-ruby)
['http'](https://github.com/httprb/http)


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

If you run into any errors with Twilio, you will see a number of helpful tips printed to the terminal about how to resolve your issue.

Otherwise, check your phone and you should expect to see an MMS with a random Futurama screencap and caption! Here's an example:

![S10E23](https://frinkiac.com/img/S11E02/921960.jpg)

*Okay, so you say your son is towheaded, button nose, mischievous smile, and may be armed with a slingshot?*

# Wrapping Up


Special thanks to [this](https://www.twilio.com/blog/2015/10/4-ways-to-parse-a-json-api-with-ruby.html) Twilio article for help on parsing JSON with Ruby.


Congratulations! You've just built a Twilio-powered MMS Futurama quote-bot using nothing more than a few lines of Ruby. The tools in this guide -- using the `requests`, `twilio-ruby`, and `schedule` libraries, and more -- can be used in numerous ways to create applications with even more functionality. Try combining new APIs and libraries with some of Twilio's other features like [Voice](https://www.twilio.com/voice) or [IP Messaging](https://www.twilio.com/ip-messaging), and see what you can come up with!

If you enjoyed this post be sure to check out the [Twilio Blog](https://www.twilio.com/blog/) for an endless string of interesting articles and fun projects. Also, feel free to check out my [personal blog](https://brodan.biz/blog/) for both technical and non-technical articles, and follow me on Twitter [@brodan_](https://twitter.com/Brodan_) to see when I publish new posts!


# Optional Steps

* An alternative approach to configuring your `TwilioRestClient` object is to use environment variables. **With environment variables you won't have to worry about making your API keys visible to the public.** To do this, run the following commands in your terminal, _replacing the values with your actual account SID and auth token_:

```

```
Then open `morbotron.rb` and add `import os` to the top of the file and replace `account_sid = 'XXXXXXX'` and `auth_token  = 'YYYYYYYY'` with the lines below:
```
 Twilio.configure do |config|
   config.account_sid = account_sid
   config.auth_token = auth_token
 end

 and then you can create a new client without parameters
@client = Twilio::REST::Client.new
```

* As mentioned earlier, using `schedule` is very intuitive. Some alternative schedules you could use in your app are: https://github.com/jmettraux/rufus-scheduler#scheduling
* 
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

