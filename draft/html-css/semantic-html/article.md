# Semantic HTML


There are many websites on the Internet. Many of them are made a long time ago and they are not updated with the latest HTML features. For that reason, they are not search engine friendly and they are not acessible for blind and visually impaired persons.

One of the most important features of the [HTML5](https://en.wikipedia.org/wiki/HTML5) is its **semantics**. It adds aditional information to basic html elements and makes a web page more informative and adaptable to search engines and [screen readers](https://en.wikipedia.org/wiki/Screen_reader).

## Basic HTML Page Structure

Let's consider a basic HTML page structure, written in the basic (non-semantic) HTML:
``` HTML
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <div>
            Here goes logo, navigation, etc. 
        </div>
        <div>
            A place for website's main content 
        </div>
        <div>
            Footer information, links, etc.
        </div>
    </body>
</html>
```
In the semantic HTML, the code should look like this:
``` HTML
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <header>
            Here goes logo, navigation, etc. 
        </header>
        <main>
            A place for website's main content 
        </main>
        <footer>
            Footer information, links, etc.
        </footer>
    </body>
</html
```
As you can notice, we have replaced *div* tags with 3 new tags: *header*, *main* and *footer*. Those tags are **semantic** and they are used to represent different sections on a HTML page. There are much more meaningful than *div* tags, since there is no doubt what parts of the HTML page are header and footer and where is the main content on the page.

## HTML Page Layout




## Browser Support

[http://caniuse.com/#feat=html5semantic](http://caniuse.com/#feat=html5semantic)

[https://github.com/aFarkas/html5shiv](https://github.com/aFarkas/html5shiv)

[https://modernizr.com](https://modernizr.com)