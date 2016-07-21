[Google Chrome extensions](https://developer.chrome.com/extensions) are an awesome way to improve the way people can browse the internet. Many Chrome extensions require a way to store data long-term, but selecting and configuring the right database can be difficult.

This blog post is going to cover how to build a basic Chrome extension and connect it to a [RethinkDB](https://rethinkdb.com/) database. Our app will open 

If you've never built a Chrome exntension before or you've never used a NoSQL database like RethinkDB then this is a great tutorial for you.

# Requirements
Let's take care of all of our setup first so that afterwards we can code without any roadblocks. This section will step through installing each tool we need and why we need them.

* Chrome - In order to build a Chrome extension we obviously need to have Google Chrome installed. You can download it from [here](https://www.google.com/chrome/browser/desktop/) if you don't already have it.

* Python/pip - Next we need to make sure you have [Python](https://www.python.org/) and [pip](https://pip.pypa.io/en/stable/#). Python comes pre-installed on many UNIX/Linux distributions now, but if you don't have it you can download it from [here](https://www.python.org/downloads/). pip comes packaged with Python versions >=2.7.9 or >=3.4, otherwise you can download it from [here](https://pip.pypa.io/en/stable/installing/).

* Flask - For our backend we are using [Flask](http://flask.pocoo.org/), a micro framework for Python. Since we've already installed pip, all we need to do to install Flask is run the following command:
```
$ pip install Flask
```

* RethinkDB - https://rethinkdb.com/docs/install/

* ngrok - We are going to use an incredible tool called [ngrok](https://ngrok.com/) to reach our Flask application. ngrok allows us to create publically accesible tunnels to our local machine so that we don't need to worry about hosting and configuring a server. To install ngrok, download it from [here](https://ngrok.com/download) and unzip it. If you want to be able to run ngrok anywhere on your command line, move the executable file you just unzipped using the following command (you may need to use 'sudo'):
```
$ mv ngrok /usr/local/bin
```


Once you have all everything installed we can finally start coding!

# Building our Chrome Extension
https://developer.chrome.com/extensions/xhr


setting up ngrok

send data to ngrok

query data from chrome extension

If you've never built a Chrome extension before and/or never worked with RethinkDB, then this is a guide for you.

# Setting Up Our Flask Server


# Running ngrok
You probably noticed the output from starting our Flask server looked something like this: 
``` 
* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
This means that we need to expose port 5000 in order to access this server. ngrok makes this an astronishingly easy task. All you need to run is the following:
```
$ ngrok http 5000
```



# Wrapping Up/Future Steps
If you'd like to take this tutorial a step further and host your application and database on its own server, Digital Ocean wrote an awesome tutorial on [serving Flask applications with gunicorn and nginx](https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-gunicorn-and-nginx-on-ubuntu-16-04).

Thanks for reading this post! Hopefully it made for a helpful introduction to RethinkDB, Chrome extensions, and/or Flask.