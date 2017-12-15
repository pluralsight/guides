

## Why Web Scraping










A lot of people scrape a site or multiple sites to see if prices on items have gone up or down or for stats or data that changes frequently.  A lot of sites have searchable API's, but, some don't, so we need ways to find the infomation we are looking for quickly. 



Say you have a website like one of the two below.  Shown is Amazon.com and Destinytracker.com.  

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/881c2e2b-359f-4bc6-8271-53fccaecba11.png)

If you want to simply run a program that tell's you the price of the item pictured, or, to see if your stat's have gone up or down without having to go to the webpage over and over, then using [Python](https://www.python.org) is the way to go.
<br>

<b>The flow is pretty simple:</b> <br>`1` Search a Webpage<br>`2` Inspect it<br>`3` Pythonic Magic<br>`4` Store your Results. 
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/11fcf0ee-8e70-46e4-8850-bd19e36e98e1.png)


You will need to know some basic `HTML` and `CSS`, or atleast be able to read it and understand what you are looking at.
Essentually, what you'll be doing is pulling up a webpage and peaking behind the curtain.  

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


## Tools
mm








