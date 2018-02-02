
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/310d6edd-b569-408a-a61d-f6d9a9a9eb61.png)


## Let's Begin

Sometimes we will find ourselves needing information (data) from a website.  If the website does't contain an API or maybe we just want to check for changing information such as Prices on a ecommerce page or maybe you want to track the up's and downs of bitcoin without constently checking the website.  Well, we can write a program for that.  


<p align="center">
<b>The Flow Is Pretty Simple:</b> 
<br>
`1` Search a Webpage<br>
`2` Inspect it<br>
`3` Pythonic Magic<br>
`4` Store your Results. 
</p>

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/11fcf0ee-8e70-46e4-8850-bd19e36e98e1.png)


You'll need to know some the basic anatomy of `HTML`.

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

Essentually, you're loading a webpage and peaking behind the curtain.  We will write a script that grabs the `HTML` and then parses it into usable chunks.

The basic framework (_behind the curtain_) of a webpage is `HTML` & `CSS`.  Just think of `HTML` as a structure and `CSS` as the structure's style.  The layout of a webpage is super important as you will come to find later.  `HTML` is great because it's predictable and structured.  


## Toolbox


We will be using the following:  <br>
`1` [Python 3.6](https://www.python.org/downloads/)<br> 
`2` Terminal: <br>
`3` [Sublime Text 3](https://www.sublimetext.com/3)<br>

Python Packages:<br>
[urllib](https://docs.python.org/3/library/urllib.html)<br>
[BeautfulSoup](https://pypi.python.org/pypi/BeautifulSoup/3.2.1)<br>

If you do not have Pyhton, do not fear.  Simply open the Python link above, download it, and install it (_3.x and above_).
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
When you're finished type `exit()` to quit Python but remain in the terminal.
```
$ exit()
```


Now let's install our packages.  While in terminal type the following: `pip install`.  If you get errors, or it's installing to Python 2, please use `pip3 install` instead.
```
$ pip install beautifulsoup4
```
and
```
$ pip install urllib3
```


To make sure everything is working you can now simply type `python` - or -  if you have mutliple versions of python type in `python3` to run Python 3.x.  

```
$ Python

```

Next import your packages: BeautifulSoup and urllib.  

```
import bs4, urllib3
```


It should look something like this.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/640c0c20-aa57-41a7-9a64-2e80991724e9.gif)

```>>>``` Is a good sign!!!  If there was an error you would get a bunch of nasty literature...

Now that we have Python and our packages let's scrape.

 




## Python Time

You can complete this in the terminal line by line which is great for testing.  Meaning, you cannot move forward until your errors are resolved, or, you can use a text editor to create your script as a whole then run it.  Sometimes it's easier to use terminal until you know everything is working then copy it to your .py program.  The choice is yours.  

Please open your Terminal and type in the following.
```
import requests
from bs4 import BeautifulSoup
```
When importing, remember packages are case sensitive. If you type `beautifulsoup` you'll get an error, so make sure you type your packages correctly. 

Also, notice I wrote `BeautifulSoup as Soup` "as" assigns an allias.  You don't want to type "BeautifulSoup" over and over.  You can asign anything as the variable such as "Soup" or "BS" or "WowThisIsReallyAwesome" etc.  

If you get our favorite `>>>` then we can move forward. 

Let's grab a website.  For this example will use something simple.  

Here is the URL:  [Cow Mask](https://www.amazon.com/CreepyParty-Novelty-Halloween-Costume-Party/dp/B0199PV50K/ref=pd_sim_21_3?_encoding=UTF8&pd_rd_i=B0199PV50K&pd_rd_r=9CN0FB2X3B4YRY4JBSHW&pd_rd_w=yoPuL&pd_rd_wg=AsYde&psc=1&refRID=9CN0FB2X3B4YRY4JBSHW)

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/c55c896d-a5f7-4a85-ad53-7c78bf1d06b7.41)

Let's say this product is to expensive and we want to keep checking to see if it will come down.  

First, we need Python to connect to the URL and grab the HTML.

Type the following:
```
source = request.get("https://www.amazon.com/CreepyParty-Novelty-Halloween-Costume-Party/dp/B0199PV50K/ref=pd_sim_21_3?_encoding=UTF8&pd_rd_i=B0199PV50K&pd_rd_r=9CN0FB2X3B4YRY4JBSHW&pd_rd_w=yoPuL&pd_rd_wg=AsYde&psc=1&refRID=9CN0FB2X3B4YRY4JBSHW").text
```
We just grabbed our URL with requests and stored it into a variable called "source".  To simplify, we took infomation and placed it into a box called "source"  That way when we need it, we don't need to type "`urllib('https://www.......')`" over and over, we simply need to type "`source`".

Next type:

```
soup = BeautifulSoup(source.html, 'html.parser')
```

We are 






