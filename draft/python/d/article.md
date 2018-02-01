## Why Web Scraping





A lot of people scrape a site (or multiple sites) to see if prices of items have gone up or down or for stats or data that changes frequently.  A lot of sites have searchable API's, but, some don't, so we need ways to find the infomation we are looking for quickly. 



Say you have a website like one of the two below. If you would like to monitor multiple prices/values/data from one site, or multiple sites you'll need to create a program that scrapes that information.  

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/4ab40f70-5fb6-4ce4-9bae-b65661c3c605.png)


<p align="center">
<b>The Flow Is Pretty Simple:</b> 
<br>
`1` Search a Webpage<br>
`2` Inspect it<br>
`3` Pythonic Magic<br>
`4` Store your Results. 
</p>

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/11fcf0ee-8e70-46e4-8850-bd19e36e98e1.png)
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6223d0ce-eb2c-4685-8660-551e7adfcd4f.png)

You'll need to know some basic `HTML` and `CSS` or atleast be able to read it and understand what you are looking at.
Essentually, what you'll be doing is pulling up a webpage and peaking behind the curtain.  We will write a script to grab the `HTML` and then parse it into usable chunks.

The basic framework (_or behind the curtain_) of a webpage is `HTML` & `CSS`.  Just think of `HTML` as a structure and `CSS` as the structure's style.  The layout of a webpage is super important as you will come to find later.  

`HTML` is great because it's predictable and structured.  
```
<!DOCTYPE html>  
<html>  
    <head>
    </head>
    <body>
        <h1> Why Web Scraping </h1>
        <p> Scrape All The Things!! </p>
    <body>
</html>
```

## Toolbox


We will be using the following:  (_<b>Note:</b>  I am on a Mac, what's a PC anyways jk_)<br>
`1` [Python 3.6](https://www.python.org/downloads/)<br> 
`2` Terminal:  _Just use ⌘ + Space Bar and type 'Terminal'_ <br>
`3` [Sublime Text 3](https://www.sublimetext.com/3)<br>

Python Packages:<br>
[urllib](https://docs.python.org/3/library/urllib.html)<br>
[BeautfulSoup](https://pypi.python.org/pypi/BeautifulSoup/3.2.1)<br>

If you don't have Pyhton, don't fear.  Simply open the Python link above, download it, and install it.  
If you have Python and are unsure of what version you are using.  Open Terminal and type `python` and click enter.

```
$ Python
```
You'll see the version populate like below. 
```
Python 3.6.3 (v3.6.3:2c5fed86e0, Oct  3 2017, 00:32:08) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>
```
Now let's install our packages.  While in terminal type the following: `pip install`.  If you get errors, or it's installing to Python 2, please use `pip3 install` instead.
```
$ pip install beautifulsoup4
```
and
```
$ pip install urllib3
```
(_NOTE: For extra credit, install `pyqt5` w/ `pip3 install pyqt5`_) We will be using it later :)  I won't spoil the surprise!!  <br>

To make sure everything is working you can now simply type `python` - or -  if you have mutliple versions of python type in `python3`.  Next import your packages: BeautifulSoup and urllib.  

```
$ Python
import bs4, urllib3
```


Should look something like this.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/640c0c20-aa57-41a7-9a64-2e80991724e9.gif)

```>>>``` Is a good sign!!!  If there was an error you would get a bunch of nasty literature...

Now that we have Python and our packages let's scrape.

Wait, what's a package?  

Watch This Video:  




## Python Time

Please open your Terminal and type in the following.
```
from urllib.request import urlopen as alias
from bs4 import BeautifulSoup as Soup
```
_When importing, remember packages are case sensitive._  If you type `beautifulsoup` you'll get an error, so make sure you type in your packages correctly. 






























<br>








<br>



<br>



<br>



<br>



<br>











