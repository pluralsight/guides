Back in August I wrote a post for the hack.guides() Tutorial Contest about [building a Simpsons quote-bot with Twilio MMS, Frinkiac, and Python](http://tutorials.pluralsight.com/interesting-apis/build-a-simpsons-quote-bot-with-twilio-mms-frinkiac-and-python).
I enjoyed writing that post so much that I decided to rewrite it in a language less familiar to me: [Ruby](https://www.ruby-lang.org/). I've also swapped out Frinkiac for [Morbotron](https://morbotron.com/), a similar database made by the same team.


In this tutorial, we are going to use Twilio along with Morbotron, the Futurama quote and screencap database, to create a Ruby app that will automatically send us a Futurama screencap and quote every day via [MMS](https://en.wikipedia.org/wiki/Multimedia_Messaging_Service). **We are going to accomplish this in less than 50 lines of Ruby.** Yup, no cron jobs, no servers, just Ruby.

If you don't want to follow along and just want to see the finished code, check out the Github [repository.](https://github.com/Brodan/MorbotronMMSBot).

# Getting Started


Before we can jump into code, we need to get our environment set up. 

### Ruby and RVM
For this project I'm going to be using [RVM](https://rvm.io/), the Ruby Version Manager, to install Ruby version 2.3.0. I recommend following RVM's [quick guided install](https://rvm.io/rvm/install#quick-guided-install) to set it up. Once installed, run the following commands to install the necessary Ruby version and set our 
```
$ source ~/.rvm/scripts/rvm
$ rvm install 2.3.0
$ rvm use 2.3.0
```

If you see the message `RVM is not a function, selecting rubies with 'rvm use ...' will not work.` check out [this](http://stackoverflow.com/questions/9336596/rvm-installation-not-working-rvm-is-not-a-function) StackOverflow post to correct it. 

### Ruby libraries

Next we need to install a few Ruby libraries (also called gems). It's a good idea to use [bundler](http://bundler.io/) to handle this. The following command will install bundler which we can use to download and install all the external libraries we will need:

```
$ gem install bundler

```
Then we need to make a [gemfile](http://bundler.io/gemfile.html), which bundler will use to install our libraries. Create a new file in your project direcotry called `Gemfile` and add the following:
```
source 'https://rubygems.org'
gem 'httparty'
gem 'rufus-scheduler'
gem 'twilio-ruby'
```
Save it, and we can now install all of the gems we need by running the following in the same directory:
```
$ bundle install
```

Let's break down what we just installed. 
* ['rufus-scheduler'](https://github.com/jmettraux/rufus-scheduler) - A popular job scheduler for Ruby that will allow us to execute a method at specific time intervals.
* ['httparty'](https://github.com/jnunemaker/httparty) - A gem that simplifies HTTP requests, which we will use for making GET requests to the Morbotron API since there is no official API wrapper.
* ['twilio-ruby'](https://github.com/twilio/twilio-ruby) - The official Ruby gem for using the Twilio REST API. 

### Twilio account

Lastly, make sure you have a Twilio account. If you don't, you can [sign up for free](https://www.twilio.com/try-twilio). You'll need a [Twilio phone number](https://www.twilio.com/help/faq/phone-numbers) with SMS and MMS capabilities. You can check the capabilities of numbers on the [Phone Numbers Dashboard](https://www.twilio.com/console/phone-numbers/dashboard). Once that's all set up, you're ready to start building the quote-bot.

# Building Our App

It's time to start building our app. We only need one file, so navigate to a directory of your choosing and open a new file called `morbotron.rb` in your preferred editor.

At the top of this file add the following lines:

```
require 'twilio-ruby'
require 'rufus-scheduler'
require 'httparty'

# Set up a client to talk to the Twilio REST API.
account_sid = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX' # Your Account SID from www.twilio.com/console
auth_token = 'YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY' # Your Auth Token from www.twilio.com/console
@client = Twilio::REST::Client.new account_sid, auth_token

# Set up our scheduler.
scheduler = Rufus::Scheduler.new
```

The first three lines in the code above are simply importing all of the libraries we just installed. The three lines after our imports configure and create a `Twilio::REST::Client` object that will let us make calls to the [Twilio REST API](https://www.twilio.com/docs/api/rest). Make sure you replace the values for `account_sid` and `auth_token` with your actual account SID and auth token. You can find these values in your [Twilio account dashboard](https://www.twilio.com/console). The final line is creating our `scheduler` which we will configure at the end of our app.

**Important note**: Never push code with your API credentials to a public repository. See the "Optional Steps" section at the bottom of this post for an alternative approach to using your Twilio API keys.

Next add the following method:

```
def get_quote
  r = HTTParty.get('https://morbotron.com/api/random')
  # Check if our request had a valid response.
  if r.code == 200
      json = r.parsed_response
      # Extract the episode number and timestamp from the API response.
      _, episode, timestamp = json["Frame"].values

      # Build a proper URL 
      image_url = "https://morbotron.com/meme/" + episode + "/" + timestamp.to_s
      
      # Combine each line of subtitles into one string, seperated by newlines.
      caption = json["Subtitles"].map{|subtitle| subtitle["Content"]}.join("\n")

      return image_url, caption
  end
end
```

The method we just added uses `httparty` to send a GET request to Morbotron and retreive data about a random Futurama moment. Although Morbotron isn't *actually* an API, the entire site is react-based and fetches resources via HTTP. As such, we can use the site in the same way that we would use an API. Next, we convert our retrieved data into [JSON](http://www.json.org/), extract the `timestamp` and `episode` code, and convert these two components into string format. `timestamp` and `episode` are used to create the URL that points to a screencap of the random Futurama moment. Finally, we grab the contents from each line of subtitles in our JSON and join them together to form our screencap's `caption`.

Now add the only other method that we need:
```
def send_MMS
  media, body = get_quote
  begin
    @client.messages.create(
      body: body,
      media_url: media,
      to: '+12345678901',  # Replace with your phone number
      from: '+12345678901' # Replace with your Twilio number
    )
    puts "Message sent!"
  rescue TwilioRestException
    # If an error occurs, print it out.
    $stderr.print "Message failed to send: " + $!
  end
end
```
This method starts by calling the `get_quote` method we created in the previous step and by storing its return values. The `begin`/`rescue` block from above was adapted from the [Twilio Ruby Helper Library ](https://www.twilio.com/docs/libraries/ruby#exceptions) documentation. These lines are simply taking in a number of parameters and turning them into a call to the Twilio REST API. 

_Replace the `to` and `from` parameters with your phone real phone number and your Twilio phone number, respectively._
If an error occurs during the API call it will be printed to the terminal.


Now at the bottom of our file, below the two methods we just added, insert the following three lines:
```
scheduler.every '24h' do
  send_MMS
end
scheduler.join
```
`rufus-scheduler` allows you to set how often a method runs in a very intuitive way. Our app will now behave according to the scheduler, which means that `send_MMS` will be called **once every 24 hours** indefinitely or until you exit the app.

# Testing Our App
For the purpose of testing the application, it's a good idea to change the schedule we added above to run more frequently. For example, ```scheduler.every '30s' do``` would call the `send_MMS` method every 30 seconds. (This way you won't need to wait 24 hours to know if your application is working.)

After you've changed that, make sure you save `morbotron.rb`. Go back to your terminal and run the following command:

```
$ ruby morbotron.rb
```
**Your terminal will look like its frozen, but that's because your app is running.** After 30 seconds, you should see a line printed to your terminal that says `Message sent!`. See the "Optional Steps" section for instructions on running your program as a background process.

If you run into any errors with Twilio, you will see a number of helpful tips printed to the terminal about how to resolve your issue.

Otherwise, check your phone and you should expect to see an MMS with a random Futurama screencap and caption! Here's an example:

![S07E12](https://morbotron.com/img/S07E12/644185.jpg)

*"It's a priceless quantum-force gemerald."*

# Wrapping Up


Congratulations! You've just built a Twilio-powered MMS Futurama quote-bot using nothing more than a few lines of Ruby. The tools in this guide -- using the `httparty`, `twilio-ruby`, and `rufus-scheduler` libraries, and more -- can be used in numerous ways to create applications with even more functionality. Try combining new APIs and libraries with some of Twilio's other features like [Voice](https://www.twilio.com/voice) or [IP Messaging](https://www.twilio.com/ip-messaging), and see what you can come up with!

Special thanks to [this](https://www.twilio.com/blog/2015/10/4-ways-to-parse-a-json-api-with-ruby.html) Twilio article written by [Greg Baugues](https://twitter.com/greggyb) for help on parsing JSON with Ruby.

If you enjoyed this post be sure to check out the [Twilio Blog](https://www.twilio.com/blog/) for an endless string of interesting articles and fun projects. Also, feel free to check out my [personal blog](https://brodan.biz/blog/) for both technical and non-technical articles, and follow me on Twitter [@brodan_](https://twitter.com/Brodan_) to see when I publish new posts!


# Optional Steps

* An alternative approach to configuring your `Twilio::REST::Client` object is to use environment variables. **With environment variables you won't have to worry about making your API keys visible to the public.** To do this, run the following commands in your terminal, _replacing the values with your actual account SID and auth token_:

```
$ export TWILIO_ACCOUNT_SID='XXXXX'
$ export TWILIO_AUTH_TOKEN='YYYYY'
```
Then open `morbotron.rb` and replace `account_sid = 'XXXXXXX'` and `auth_token  = 'YYYYYYYY'` with the lines below:
```
account_sid = ENV["TWILIO_ACCOUNT_SID"]
auth_token = ENV["TWILIO_AUTH_TOKEN"]
```

* As mentioned earlier, using `rufus-scheduler` is very intuitive. Some alternative schedules you could use in your app are:
```
scheduler.every '15m' do
scheduler.every '2d', :first_at => Time.now do
scheduler.at '2016/12/24 2000' do
```
See the [repository](https://github.com/jmettraux/rufus-scheduler#scheduling) for more examples.

* If you'd like to run your application as a background process, simply add `&` to the end of your `ruby` command:
```
$ ruby morbotron.rb &
[1] 1872
```
This command will return a process ID (PID) number (`1872` in this example) that you can append to the `kill` command in order to terminate your program (e.g. `$ kill 1872`). It might be a good idea to remove any lines that print to the terminal if you decide to do this.

___
Thank you for reading my tutorial on creating a Futurama-based Quote-Bot using Twilio MMS, Morbotron, and (some) Ruby. I hope that you found this guide informative and entertaining. Please leave your comments and feedback in the discussion section below, and please favorite this article as well!

