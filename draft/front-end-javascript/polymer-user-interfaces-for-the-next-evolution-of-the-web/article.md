## What is Polymer?
[Polymer](https://www.polymer-project.org/1.0/) is a lean & light library from Google that pushes the next evolution of the web.
It adds elegance in building custom DOM/html elements, while providing less boilerplate code.
## Goals
Polymer has set out with some goals in mind.  The main goal is to enable to developers to create compelex apps and user interfaces
with an already familiar DOM as a framework.  A secondary goal is to promote the http/2 way of creating webapps with smart caching,
which is covered later in the tutorial.  There are other ad hoc goals, but they are aimed at encapsulating and creating tools to improve
efficiency in development, testing, and performance.  For example, using [polygit](https://github.com/PolymerLabs/polygit) and 
[bower](http://bower.io/) for managing elements, [polylint](https://github.com/PolymerLabs/polylint) 
for error management, and [polydev](https://github.com/PolymerLabs/polydev) to measure the efficiency of your elements.


## Thinking in Polymer
Polymer is unlike a lot of libraries becuase it is built on and extends the HTML + JavaScript standard for the web.  Sometimes, it can
become a cumbersome task to 'just making things work', which makes it easy to ignore the fact that this is 80% reusable custom HTML and
20% JavaScript for control and definitions.

It often helps in building a polymer application to 'think locally', meaning examine one element at a time in relation to it's sibling
and parent elements.  Another important mindset to have when working with Polymer is the 'composition' mindset.  The 'composition' mindset
allows you to think of how the elements really do represent building blocks in the context of the larger app.  In browsers that support
it, you can build in what's called the "shadowDOM" (see below) to manipulate state and values which will be passed down
to the "localDOM".

The flow of data or passing of information from element to element is also required to create a robust Polymer webapp.  We call this
the "mediator pattern", but in some ways it reflects the functionality of the controller in MVC or
the dispatcher in Flux (React.js).  The general idea is reflected in the picture below.  Events that deal with
state information are passed from child to child via the mediator.

![mediator](https://raw.githubusercontent.com/pluralsight/guides/master/images/mediator.png)

### A bit about Shadow DOM
Shadow DOM is a tricky concept to understand.  It's defined in the [Web Component Spec](https://www.w3.org/TR/2013/WD-components-intro-20130606/), which
includes things like CSS decorators and custom elements. [Agraj Mangal](http://code.tutsplus.com/tutorials/intro-to-shadow-dom--net-34966) 
describes the "shadow DOM" as an encapsulation for HTML from JavaScript and CSS.  [Scott Miles](https://www.polymer-project.org/1.0/articles/shadydom.html)
 describes it as hiding the "tree scoping", where a scoped part of the tree is hidden in the "shadows" when he presents the "shady DOM" concept. 
 Essentially, when you write Polymer custom elements, they are not seen in the UI until render time (if at all), where the normal HTML tree is replaced by the abstract "shadow DOM".
Currently, Agraj cites examples like the ```<audio></audio>``` tag.  In that element, the normal DOM is replaced
by abstract elements that show up as the ```<audio></audio>``` tag when you inspect the source.  It's not until
one enables the option in Chrome Dev Tools (or any other supporting browser like Opera).

* **Shadow Host**: is the DOM element which is hosting the Shadow DOM subtree or it is the DOM node which contains the Shadow Root.
* **Shadow Root**: is the root of the DOM subtree containing the shadow DOM nodes. It is a special node, which creates the boundary between the normal DOM nodes and the Shadow DOM nodes. It is this boundary, which encapsulates the Shadow DOM nodes from any JavaScript or CSS code on the consuming page.
* **Shadow DOM**: allows for multiple DOM subtrees to be composed into one larger tree. 
* **Composition**: the process of overlaying the nodes
* **Shadow Boundary**: is denoted by the dotted line in the image above. This denotes the separation between the normal DOM world and the Shadow DOM world. The scripts from either side cannot cross this boundary and create havoc on the other side.
* **Shady DOM**: a super-fast shim for shadow DOM that provides tree-scoping, but has drawbacks like complicated CSS.  useful for restoring Shadow DOM values and examining subtrees quickly.

## The Polymer Starter Kit

Want to start building right away?  Take a look at the [Polymer Start Kit](https://developers.google.com/web/tools/polymer-starter-kit/?hl=en).  The PSK is a wonderful way to avoid what we call
the "blank page effect".  Many generators (including Yeoman for Angular) require some time to configure.  Then, after all the complexity of configuration, the end result is essentially a blank page.
Being of a Polymer mindset, we want to keep the complexity at a minimum.  PSK allows us to set up a commercially viable UI in a few CLI commands or with a [simple unzipping](https://github.com/polymerelements/polymer-starter-kit/releases).

[Confused?  Learn more here.](#psk)

## Polylint

## Data-Binding




<h2 id="psk">The PSK Explained</h2>

examples, details, etc.

<h2 id="psk">Polylint - Standardize your formatting</h2>

examples, details, etc.

<h2 id="psk">Binding your data to the UI</h2>

<h2 id="psk">The Polymer Test suite</h2>

<h2 id="psk">Polylint - Standardize your formatting</h2>

<h2 id="psk">Conserving bandwidth with HTTP/2</h2>

<h2 id="psk">E-Commerce elements</h2>

<h2 id="psk">Empowerment with Google Web Components</h2>

## Hello, world!

# organization / branches? in series

     - animation
     - data-binding
     - e-commerce
     - http-2
     - intro
     - polydev
     - polylint
     - psk
     - testing
     - web-components
     
# proposed series order 
     
     1. intro
     2. psk (Polymer Starter Kit)
     3. polylint (error linting)
     4. data-binding
     5. testing
     6. http-2
     7. e-commerce
     8. web-components