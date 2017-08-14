## Vue.Js

Vue (which is pronounced as view) is very liberal framework for construction of the user interface. Unlike the others monumental frameworks, a Vue is designed from a minced up to be incrementally acceptable. Its core library is now concentrated on the view layers only, and it is very informal to give a lift to and to integrate with the already exist other libraries or with an existing project. Beside this, Vue have also a capability which is perfect and operating the urbane Single-Page Applets when it is used in a permutation with a recent tools and subsidiary libraries.

If you have an experience of frontend developing and you may want to know how the Vue is compared to other formal framework and libraries, and check this comparison.

# Features

Vue.js just offers a very large functionality for its view layer and it can be used further for the building of very powerful webapp which is single-page. In the following you can find a list of features:

- Template Logic
- Computed Properties
- Data Binding
- Directives
- Filters
- Declarative Rendering
- CSS Transitions and Animations

# How to use Vue.js

There are many ways to include a Vue.js in our web based projects:

- In first method we have to use a CDN by including a &lt;script&gt; tags in our HTML type file.
- Then the second way is to install that by using a Node Package Manager (NPM).
- Third method is to Install using the Bower
- Fourthly we have to use a Vue-cli to setup our project

In the above following now we&#39;re trying to setup the Vue-cli to a new project and install the recommended Vue.js 2 library.

# Getting Started with Vue

The coolest way is to try out a Vue.js which is used the JSFiddle in the example of Hello World. Then feel free to further open it in the other tabs and then follow along with as we have to go through this basic example. As like

[When we have to create an index.html and then we include the Vue with then:
https://unpkg.com/vue

 This Installation page has provided more choices of installing the Vue&#39;s. But we have to note that we don&#39;t have any recommend trainees to initiate with file vue-cli, and especially if we are not quite aware with a Node.js oriented build tool.

The core of the Vue.js is a method that supports us to a declaratively extract the data to this DOM by using simple template used as syntax:

```
| &lt;div id=&quot;apps&quot;&gt;  {{ message }}&lt;/div&gt; |
| --- |

| var apps = new Vue({  el: &#39;#apps&#39;,  data: {    message: &#39;Hi its vue!&#39;  }}) |
| --- |
```

Output:

**Hi its vue!**

##

:app.message:
**Declarative Rendering**

Declarative Rendering data to DOM can be use using the below template
syntax

 

``` 

&lt;div id="app"&gt;

  {{ message }}

&lt;/div&gt;

 

 

var app = new Vue({

  el: '#app',

  data: {

    message: 'Hello to Vue!'

  }

})

``` 

Hello to Vue!

This is pretty similar to the rendering a string template but the Vue lot of work done under the hood. Now the DOM and data are linked and everything is reactive.
This can be verified by opening the browser's Java script console which is on
the right of this page and see app,message to a different value.

In addition to text interpolation, we can also bind element attributes
like this:

 

```
&lt;div id="app-2"&gt;

  &lt;span v-bind:title="message"&gt;

    Hover your mouse over me for a few
  seconds

    to see my dynamically bound title!

  &lt;/span&gt;

&lt;/div&gt;

 

 

var app2 = new Vue({

  el: '#app-2',

  data: {

    message: 'You loaded this page on the following Date: ' + newDate()

  }

})
```

Hover your mouse over me for a few seconds to see my dynamically bound
title!

Furthermore we have to use text exclamation; but you can merge an element property like this:

```
| <div id="app-a">  <spanv-bind:title="message">   Type here something  </span></div> |
| --- |

| varappa = new Vue({  el: &#39;#app-a&#39;,  data: {    message: &#39;we are now loaded our page on &#39; + new Date()  }}) |
| --- |
```

Output:

**Type here something**

# Conditions and Loop

#

```
It&#39;s a very simple for us to toggle the presence of a single element:

| &lt;div id=&quot;app-b&quot;&gt;  &lt;p v-if=&quot;seen&quot;&gt;So now you can see me&lt;/p&gt;&lt;/div&gt; |
| --- |

| varappb = new Vue({  el: &#39;#app-b&#39;,  data: {    seen: true  }}) |
| --- |

```

Output:

**So now you can see me**


```
| &lt;div id=&quot;app-c&quot;&gt;  &lt;ol&gt;    &lt;li v-for=&quot;todo in todos&quot;&gt;      {{ todo.text }}    &lt;/li&gt;  &lt;/ol&gt;&lt;/div&gt; |
| --- |

| varappc = new Vue({  el: &#39;#app-c&#39;,  data: {    todos: [      { text: &#39;Learn HTML5&#39; },      { text: &#39;Learn The Vue&#39; },      { text: &#39;Learn JQuery&#39; }   ]  }}) |
| --- |
```

##

Output:

**1. Learn JQuery**

**2. Learn TheVue**

**3. Learn HTML5**



# Handle the User Inputs

To let the user have an interaction with the apps, we can also use a directive named v-on that is used to bind the event listener and calls the methods on their instances of Vue:

##

```
| &lt;div id=&quot;app-d&quot;&gt;  &lt;p&gt;{{ message }}&lt;/p&gt;  &lt;buttonv-on:click=&quot;reverseMessage&quot;&gt;Messageis now Reversed &lt;/button&gt;&lt;/div&gt; |
| --- |

| varappd = new Vue({  el: &#39;#app-d&#39;,  data: {    message: &#39;Hi its Vue.js!&#39;  },  methods: {    reverseMessage: function () {      this.message = this.message.split(&#39;&#39;).reverse().join(&#39;&#39;)    }  }})Output: |
| --- |
```

**Hi its Vue.js!**

**Reverse Message**

##

Vue can also provide the directive of v-model that enable two-way binding between the app state and the form inputs:

```
| &lt;div id=&quot;app-e&quot;&gt;  &lt;p&gt;{{ message }}&lt;/p&gt;  &lt;input v-model=&quot;message&quot;&gt;&lt;/div&gt; |
| --- |

| varappe = new Vue({  el: &#39;#app-e&#39;,  data: {    message: &#39;Hi it&#39;s me Vue!&#39;  }}) |
| --- |

```

Output:

**Hi it&#39;s me Vue**
