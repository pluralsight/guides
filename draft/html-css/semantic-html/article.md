# Semantic HTML

There are many websites on the Internet. Many of them are made a long time ago and they are not updated with the latest HTML features. For that reason, they are not ranked well in search engines.

One of the most important features of the [HTML5](https://en.wikipedia.org/wiki/HTML5) is its **semantics**. It adds aditional information to basic html elements and makes a web page more informative and adaptable to search engines and screen readers (a software used by blind or visually impaired persons).

For instance, let's see the following code snippet, written in the basic (non-semantic) HTML:
``` HTML
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <div id="header">
            Here goes logo, navigation, etc. 
        </div>
        <div id="mainContent">
            A place for website's main content 
        </div>
        <div id="footer">
            Footer information, links, etc.
        </div>
    </body>
</html>
```
In the semantic HTML, the code above should look like this:
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