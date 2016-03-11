## Purpose
This guide will give you a jumping off point to start testing your web app with Selenium Webdriver. Webdriver allows you to automate interactions with a website through a browser. The goal is to create a simple script that will navigate through a website and take a screen shot when something goes wrong.

## Setup
We will be using the selenium-webdriver and node filesystem modules in this guide. You can use npm to acquire selenium-webdriver.

`$ npm install selenium-webdriver`

We need to require the webdriver and filesystem module.

```
var webdriver = require('selenium-webdriver');
var fs = require('fs');
```

## Create a Driver
```
function GetDriver(name)
{
    switch(name){
        case "FireFox":
            return new webdriver.Builder().forBrowser('firefox').build();
        case "Chrome":
            return new webdriver.Builder().forBrowser('chrome').build();
    }
}
```

## Write a Test
```
function test(name,fileName) {
    return new Promise(function(resolve) {
        var driver = GetDriver(name);
        //add testing
    }
}
```

## Take a Picture
```
function TakePicture(driver,fileName,driverName){
    driver.takeScreenshot().then(function(data){
        var base64Data = data.replace(/^data:image\/png;base64,/,"");
        fs.writeFile(driverName + " : " + fileName+".png", base64Data, 'base64', function(err) {
            if(err) console.log(err);
        });
    });
}
```

## Put it Together
