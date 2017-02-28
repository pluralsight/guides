# Introduction

There are over 1 billion websites on the Internet. A large number of them are made a long time ago and they are not updated with the latest HTML features. For that reason, they are not search engine friendly and they are not acessible for blind and visually impaired persons.

One of the most important features of the [HTML5](https://en.wikipedia.org/wiki/HTML5) is its **semantics**. It adds aditional information to basic html elements and makes a web page more informative and adaptable to search engines and [screen readers](https://en.wikipedia.org/wiki/Screen_reader).

In this tutorial we will learn how to use new semantic HTML5 tags in a web page creation and how to add more information to its content.

# Semantic HTML Page Layout
## Basic HTML Page Structure
Let's first consider a basic HTML page structure, written in the basic (non-semantic) HTML:
``` HTML
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <div id="header">
            Here goes logo, navigation, etc. 
        </div>
        <div id="main-content">
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
As you can notice, we have replaced *div* tags with 3 new tags: *header*, *main* and *footer*. Those tags are **semantic** and they are used to represent different sections on a HTML page. There are much more descriptive than *div* tags, since there is no doubt what parts of the HTML page are header and footer and where is the main content on the page.

## Navigation

In the HTML5, there is the *nav* tag, so we can use it instead of the *div* tag to wrap links that make a navigation menu. For instance, the navigation menu can be put within the *header* section:
bann
``` HTML
<header>
    <img src="logo.png" />
    <nav>
        <a href="index.html">Home</a>
        <a href="services.html">Services</a>
        <a href="contact.html">Contact</a>
        <a href="about.html">About Us</a>            
    </nav> 
</header>
```
but, it can be put after the *header* section, as well:

``` HTML
<header>
    <img src="logo.png" />
</header>
<nav>
    <a href="index.html">Home</a>
    <a href="services.html">Services</a>
    <a href="contact.html">Contact</a>
    <a href="about.html">About Us</a>            
</nav> 

```
Generally, the navigation menu can be put anywhere on a page, it just needs to be wrap it with the *nav* tag. However, it shouldn't be inside of the *main* tab, unless the navigation is specific for that page, because the *main* tag is intended to contain a content that is specific for a particular page. 

## Main Content

In order to add some content into the *main* section, we need to use new HTML5 tags for that purpose, *article* and *section*, so the main content could, for instance, have the following structure:

``` HTML
<main>
    <article>
        <h1>JavaScript Basics</h1>
        <p>JavaScript is a rich and expressive language...</p>
        <section>
            <h2>Syntax Basics</h2>
            <p>Understanding statements, variable naming, whitespace...</p>
        </section>
        <section>
            <h2>Operators</h2>
            <p>Operators allow you to manipulate values...</p>
        </section>
        <section>
            <h2>Conditional Code</h2>
            <p>Sometimes you only want to run a block of code under certain conditions...</p>
        </section>        
    </article>
</main>
```
The *article* tag is used for wrapping an autonomous content on a page, i.e. content that can be removed from the page and put on some another page. It can contain several *section* tags, like in our example. The *section* tag is similar to the *div* tag, but it is more meaningful, since it is used to wrap a logical group of related content (e.g. a chapter of an article). The *section* tag can be also used to wrap the article, but it's much better to use the *article* tag for that purpose, since it is more appropriate and more descriptive. An *article* is actually an autonomous *section*.

## Additional Content

An additional content, that is not important for understanding an article, but it is related to the article, can be put inside the *aside* tag. For instance, it could be information about how many people read that article, who is the author of the article etc. In that case, the HTML code of the article could have the following structure:

``` HTML
<main>
    <article>
        <h1>News</h1>
        <h1>JavaScript Basics</h1>
        <p>JavaScript is a rich and expressive language...</p>
        <section>
            <h2>Syntax Basics</h2>
            <p>Understanding statements, variable naming, whitespace...</p>
        </section>
        <section>
            <h2>Operators</h2>
            <p>Operators allow you to manipulate values...</p>
        </section>
        <section>
            <h2>Conditional Code</h2>
            <p>Sometimes you only want to run a block of code under certain conditions...</p>
        </section>  
        <aside>
            <p>Viewed by 1503 people</p>
            <p>Author: John Smith</p>
        </aside>
    </article>
</main>
```

The *aside* tag can be also used to enclose an additional content that is related to a whole page, not just to a particular article. That content can be a sidebar, advertising, footnote etc.

The *figure* tag is used to mark up photos, code blocks, diagrams, charts, illustrations etc. Generally, it encloses a content that can be moved away into an appendix. Some images should be enclosed with the *figure* tag (e.g. a logo image), but banner ads shouldn't, since they are not related to the main content of the page. If a banner ad is in a sidebar, it shouldn't be inside of the *aside* tag, for the same reason. However, there is a way to mark up a banner ad and we will cover that in the Microdata section of this tutorial.

The *figcaption* tag represents a caption or legend for a figure. It's optional, so it can be omitted. Only one *figcaption* tag can be nested into a *figure*, so if a *figure* contains multiple images, it can be only one *figcaption* for all of them.

For instance, photos of people who liked the article could be enclosed with the *figure* tag. Since that information is not so important, it can be nested into the *aside* section:

``` HTML
<aside>
    <p>Viewed by 1503 people</p>
    <p>Author: John Smith</p>
    <figure>
        <img src="John Doe.png" />
        <img src="Maggie Smith.png" />
        <img src="Tom Hardy.png" />
        <figcaption>People who liked the article</figcaption>
    </figure>
</aside>
```
The logo in the *header* section should be enclosed with the *figure* tag, as well, so our *header* section will finally have the following code:

``` HTML
<header>
    <figure>
        <img src="logo.png" />
    </figure>
    <nav>
        <a href="index.html">Home</a>
        <a href="services.html">Services</a>
        <a href="contact.html">Contact</a>
        <a href="about.html">About Us</a>            
    </nav> 
</header>
```
## HTML5 Element Flowchart

If you are unsure which semantic tag to use in a particular case, you can always follow this great flowchart made by authors of the [HTML5Doctor](http://html5doctor.com) website.

![HTML5 Sectioning Flowchart](http://html5doctor.com/downloads/h5d-sectioning-flowchart.png)

# Microdata

Microdata provide additional information about content of a web page. They help search engines and screen readers to better understand it.

Microdata can be added as attributes to any HTML tag. For instance, let's add some data about the author of the article in our example. The *aside* section of the article will have the following code:

``` HTML
<aside>
    <p>Viewed by 1503 people</p>
    <p>Author: John Smith, Senior Software Developer at Google, Mountain View, California</p>
</aside>
```
With microdata included, the HTML code of the *aside* section would be:

``` HTML
<aside>
    <p>Viewed by 1503 people</p>
    <p itemscope itemtype="http://schema.org/Person">
        Author: 
        <span itemprop="name">John Smith</span>, 
        <span itemprop="jobTitle">Senior Software Developer</span> 
        at
        <span itemprop="worksFor" itemscope itemtype="https://schema.org/Corporation">
            <span itemtype="name">Google</span>,  
        </span>
        <span itemprop="address" itemscope itemtype="http://schema.org/PostalAddress">
            <span itemprop="addressLocality">Mountain View</span>,   
            <span itemprop="addressRegion">California</span>
        </span>
    </p>
</aside>
```
There is obviously much more data in the code above than in the previous code, but there is also much more information for machines. As you could notice, we used the following microdata attributes: *itemscope*, *itemtype* and *itemprop*. So, what do all these attributes mean?

The *itemscope* attribute indicates a new group of microdata. A group of microdata is called *item* and it contains pairs of properties and values. The type of an item is specified in the *itemtype* property and it is actually an URL to a web page that cointains information about the item. On that page we can see all properties the item could have. We usually use just several properties to describe an item and we define them in the *itemprop* attributes. A property can be simple or compound (like the *address* in our example).

We mentioned before in this tutorial that a banner can be described with microdata. So, the HTML code should look like this: 

``` HTML
<div itemscope itemtype="https://schema.org/WPAdBlock">
    <a href="site.com" itemprop="url"><img src="banner.gif" alt="banner" itemprop="image" /></a>
</div> 
```

There is a very useful tool for generating microdata, it is the [Microdata generator](http://webcode.tools/microdata-generator). It can save your time, and it can also teach you how to use microdata properly for various items. 

# Browser Support

According to [this](http://caniuse.com/#feat=html5semantic) table, the HTML5 semantic tags are not supported well in all browsers. However, the [HTML5Shiv](https://github.com/aFarkas/html5shiv) can be used for better support in some older versions of the Internet Explorer, Firefox and Safari. The HTML5Shiv is also distributed with the [Modernizr](https://modernizr.com).

# Additional Resources

For more information about this topic, please read these very useful articles:

- [HTML5 structure—div, section & article](http://oli.jp/2009/html5-structure1/)
- [The article element](http://html5doctor.com/the-article-element/)
- [Getting started with schema.org using Microdata](https://schema.org/docs/gs.html)
- [HTML5 semantic elements and Webflow: the essential guide](https://webflow.com/blog/html5-semantic-elements-and-webflow-the-essential-guide)

# Conclusion

This tutorial has covered some basics of the semantic HTML. 
