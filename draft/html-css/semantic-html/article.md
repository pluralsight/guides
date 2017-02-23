# Semantic HTML


There are many websites on the Internet. Most of them are made a long time ago and they are not updated with the latest HTML features. For that reason, they are not search engine friendly and they are not acessible for blind and visually impaired persons.

One of the most important features of the [HTML5](https://en.wikipedia.org/wiki/HTML5) is its **semantics**. It adds aditional information to basic html elements and makes a web page more informative and adaptable to search engines and [screen readers](https://en.wikipedia.org/wiki/Screen_reader).

## Basic HTML Layout

Let's consider a basic HTML layout, written in the basic (non-semantic) HTML:
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
As you can notice, we have replaced *div* tags with 3 new tags: *header*, *main* and *footer*. Those tags are **semantic** and they are used to represent different sections on a HTML page. There are much more descriptive than *div* tags, since there is no doubt what parts of the HTML page are header and footer and where is the main content on the page.

Let's add a navigation menu in the header. In the HTML5, there is the *nav* tag, so we will use it.

``` HTML
<header>
    <nav>
        <a href="index.html">Home</a>
        <a href="services.html">Services</a>
        <a href="contact.html">Contact</a>
        <a href="about.html">About Us</a>            
    </nav> 
</header>
```
We will add some content into the *main* section, as well. We will use new HTML5 tags for that purpose, *article* and *section*, so the main content will, for instance, have the following structure:

``` HTML
<main>
    <article>
        <h1> News </h1>
        <section>
            <h2>Headline 1</h2>
            <p>Content 1</p>
        </section>
        <section>
            <h2>Headline 2</h2>
            <p>Content 2</p>
        </section>
        <section>
            <h2>Headline 3</h2>
            <p>Content 3</p>
        </section>        
    </article>
</main>
```
If you are unsure which semantic tag to use in a particular case, you can always follow this great flowchart made by authors of the [HTML5Doctor](http://html5doctor.com) website.

![HTML5 Sectioning Flowchart](http://html5doctor.com/downloads/h5d-sectioning-flowchart.png)


## Browser Support

[http://caniuse.com/#feat=html5semantic](http://caniuse.com/#feat=html5semantic)

[https://github.com/aFarkas/html5shiv](https://github.com/aFarkas/html5shiv)

[https://modernizr.com](https://modernizr.com)