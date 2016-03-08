![elegant framework](http://getelegant.com/static/images/logo.png "elegant framework")

</h1>Installing Elegant</h1>
Before We Start , As you know Elegant is built using nodejs, So make sure that you already have nodejs and NPM(Node Package Manager) on your machine, for more information about installing node visit: How to Install Node.js.

elegant is a nodejs global module , to install it use the following command:

```
sudo npm install elegant -g
```

Now the elegant command should be available on your system, to make sure try this command:


```
elegant --version
```

Your First Elegant App
To create a new elegant app type this command into your commandline:


```
elegant create myapp
```

this command will create a new directory called "myapp" with your elegant project files.

Now to run your elegant project type the following commands:


```
cd myapp
node index.js
```

By default elegant app is configured to run at port 9090 so in your browser go to http://localhost:9090

**Notes: **
- you can stop your app by pressing Ctrl+C 
- you can change the app default port from the config.js file in the app directory 
<h1>App Directory Structure</h1>
After creating your first elegant project, you have a new directory with the following files structure:
- app: the root of your application, all your code should be placed here.
- lib: elegant core. 
- node_modules: node modules main directory.
- index.js: the application startup file.

<br/>
**As we mentioned before all your code should be placed inside the app directory , the app directory is structured as bellow:**

- **controllers**: app controllers directory, elegant controller is the connector between any page URL and your app logic.
- **interceptors**: custom interceptors directory, elegant interceptor is the code responsible for changing the request life cycle.
- **lib**: your own functions and modules 
- **static**: static files directory.
- **views**: app theme directory 
- **config.js**: app configuration file 

<h1>Hello World</h1>
As you notice your app is coming with a beautiful homepage, no lets replace it with our own one:
inside the controllers directory create a new javascript file and name it index.js, then write the following code in this file:


```javascript
var Controller = require("elegant-controller");
new Controller("/").get(function () {
  return "hello world";
});

```
Now you can restart your app to see your home page changed to "hello world"

Notes:
- The "/" we passed to the Controller constructor is the page url.
- the get method defines the request type for this controller , so if you wont to make it post you can use the post method instead.

Now lets try to return a complete html page, so instead of returning a string lets return an elegant view, first you should require the "elegant-view" module and return a new view:

```javascript
var Controller = require("elegant-controller");

var View = require("elegant-view");

new Controller("/").get(function () {
  return new View("index");
});
```

as you notice we have passed the "index" prefix to our new view this prefix will be used in your theme as identifier to this page, inside the views directory there is a directory called "default" this is your default theme directory, create a new html page name it "index.html" (same as the prefix we used), write your home page content inside this page and restart the app.

<h1>First Database Query</h1>
Now lets make its a bit dynamic, lets make our first database query but before we start make sure you have mysql installed and you have created a database called "myapp" with a table called "articles" as the following: id | title | description 
don't forget to insert some sample data in the articles table.
<br/>
Lets write a query that return our articles and display them in the index.html page ,So lets update our last controller (/app/controllers/index.js) with the following code :



```javascript
var Controller = require("elegant-controller");

var View = require("elegant-view");

var MySql = require("elegant-mysql");

new Controller("/").get(function () {

  this.vars = {};
  MySql.query("select * from `articles`",this.$(function (err, rows, fields) {
    this.vars.articles = rows;
  }));
​
}).ready(function(){
  return new View("index",this.vars);
});
```

Notes:
- as you know the nodejs is an event driven language so the database callback function will not be executed within the get method (so any other synchronous functions), so passing this function to the this.$ method will add this callback function to a monitor stack and the elegant will execute the controller ready method after all monitor stack functions are executed and all your data is ready. 

- the scope of controller get method is different from the controller ready method also defining variables outside the controller will make them global across all requests, so we used the "this" keyword to pass variables to the ready method using this.vars property (you can use any property name).

After we have passed our query result to the index view , Now we need to update our html page and display our articles as the following:

```html
<html>
  <head>
    <title>Articles</title>
  </head>
  <body>
    {% for article in articles %}
    <div>
      <h1>{{article.id}} - {{article.title}}</h1>
      <p>
        {{article.description}}
      </p>
    </div>
    {% endfor %}
  </body>
</html>
```


<h1>Form Submission</h1>
Now lets try to receive data from the client, so we need to create two controllers, the first one is an html page with a simple form to add a new article and the second one to receive this article data and save it to the database.
<br/>
So create a new javascript file in the controllers directory (/app/controllers/) and name it article.js then write in this file the following code for the first controller:

```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");

// first controller (add article page)
new Controller("/add/article").get(function () {
  return new View("add-article");
});
```

now in the views directory (/app/views/) create a new html file and name it "add-article.html", in this file add the following html code:

```html
<html>
  <head>
    <title>Add Article</title>
  </head>
  <body>
    <form target="_self" method="post" action="article">
      <div>
        <label>Title: </label>
        <input type="text" name="title"/>
      </div>
      <div>
        <label>Description: </label>
        <input type="text" name="description"/>
      </div>
      <div>
        <input type="submit"/>
      </div>
    </form>
  </body>
</html>
```

Now restart the application and you can access this page from the url http://localhost:9090/add/article, now for the second controller that will handle this form data, update the article.js file with the new controller as below:

```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");
var MySql = require("elegant-mysql");

// first controller (add article page)
new Controller("add/article").get(function () {
  return new View("add-article");
});

// second controller (save article data)
new Controller("add/article").post(function () {
  this.vars = {};

  // Fetch Post Parameters
  this.vars.title = this.POST['title'];
  this.vars.description = this.POST['description'];
  MySql.query("INSERT INTO `articles`(`title`,'description') VALUES(?,?)", [this.vars.title, this.vars.description], this.$(function (err, rows, fields) {
    if(err){
      this.vars.success = "no";
    }else{
      this.vars.success = "yes";
    }
  }));
}).ready(function () {
  return new View("add-article", this.vars);
});
```

in this way are passing the "success" variable to our "add-article.html" that will indicate if the article is inserted or not, so lest update our html page to display this to the user as follow:

```html
<html>
  <head>
    <title>Add Article</title>
  </head>
  <body>
    {% if success == "yes" %}
    <div>Your article is added successfully</div>
    <a href="article">Add another article ?!</a>
    {% elseif success == "no" %}
    <div>Sorry! There is a problem while adding your article, please try again</div>
    {% endif %}
    {% if !success || success == "no"%}
    <form target="_self" method="post" action="article">
      <div>
        <label>Title: </label>
        <input type="text" name="title" value="{{title}}"/>
      </div>
      <div>
        <label>Description: </label>
        <input type="text" name="description" value="{{description}}"/>
      </div>
      <div>
        <input type="submit"/>
      </div>
    </form>
    {% endif %}
  </body>
</html>
```

<h1>Using Session</h1>
a session is a bucket that hold some information on server side , Now lets Dive into using session in our elegant web application , so lets build a login form that checks user credentials and opens a session with the user information and then redirects that user to another page with his name on it.
<br/>
First we need to build our first controller login.js for the login page as the following :

```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");

// first controller (login Page)
new Controller("/user/login").get(function () {
  return new View("login");
});
```

now in the views directory (/app/views/) create a new html file and name it "login.html" and add the following code:

```html
<html>
  <head>
    <title>Login</title>
  </head>
  <body>
    <form method="post">
      Username:<input type="text" name="username"/><br>
      Password:<input type="password" name="password"/><br>
      <input type="submit" value="login"/>
    </form>
  </body>
</html>
```

after we have built our login form we need a controller that check if the credentials are true and creates a new session for the user then redirects the user to the (/user) page , update login.js as the following:

```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");
var Redirect = require("elegant-redirect");
var MySql = require("elegant-mysql");

// first controller (login Page)
new Controller("/user/login").get(function () {
  //lets check if the user is already logged in
  var user = this.SESSION.get("user");
  if(user){
    return new Redirect("/user");
  }else{
    return new View("login");
  }
});

// Secound controller to handle the login
new Controller("/user/login").post(function () {
  this.vars = {};
  // Fetch Post Parameters
  this.vars.username = this.POST['username'];
  this.vars.password = this.POST['password'];

  MySql.query("SELECT * from `user` where `username`=? AND `password`=?", [this.vars.username,this.vars.password], this.$(function (err, rows, fields) {
    if(!err){
      this.vars.user = rows[0];
    }
  }));
}).ready(function () {
  if(this.vars.user){
    this.SESSION.set("user",this.vars.user);
    return new Redirect("/user");
  }else{
    return new View("login",this.vars);
  }
});

// The user Dashbord Controller
new Controller("/user").get(function () {
  var vars = {}
  // lets get our user information from the session
  vars.user = this.SESSION.get("user");
  return new View("user",vars);
});
```

Now for the last step in our authentication app , lets build our user dashboard (user.html):

```html
<html>
  <head>
    <title>user dashbord</title>
  </head>
  <body>
    {% if user %}
    welcome {{user.username}}
    {% else %}
    You are not logged in , Please <a href="/user/login">Login</a>
    {% endif %}
  </body>
</html>
```

<h1>Html Templating</h1>
Templating is a very important aspact on any web application , the more flixable it is the more easy it can link between the server and the client , Elegant has a very flixable template engin that is easy to use and customize .
<br/>
Note: elegant template engine is built over swig, if you wont to learn more about swig syntax and features click here to visit swig documentation home page.
<br/><br/>
A Template is split into parts that we call regions such as ( header,body,footer ), every region is a stand alone html file that well be included to the master template file on render.
<br/>
Now lets Try a real life example and build our first elegant template , lets pretend that we have the following html page:

```html
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <div id="container">
      <div id="header">
        <ul>
          <li><a href="#">Home</a></li>
          <li><a href="#">About Us</a></li>
          <li><a href="#">Contact Us</a></li>
        </ul>
      </div>
      <div id="main-content">
        <div>
          Page Main Content
        </div>
      </div>
      <div id="footer">
        <div>
          Page Footer
        </div>
      </div>
    </div>
  </body>
</html>
```


Now Lets Start By Spliting our desgin into Regions , so as you can see in our html page its displayed as three parts the header , main-content and footer. so we are going to create three html files in app/view/default/regions folder ,the first html file is header.html as bellow: 
```html
<ul>
  <li><a href="#">Home</a></li>
  <li><a href="#">About Us</a></li>
  <li><a href="#">Contact Us</a></li>
</ul>
```

The secound html file is maincontent.html as bellow :
```html
<div>
  Page Main Content
</div>
```

The third html file is footer.html as bellow :

```html
<div>
  Page Footer
</div>
```

After We have finished creating our region , we should create the master page now in app/view/default and call it index.html with the following code :

```html
<html>
  <head>
    <title>Home</title>
  </head>
  <body>
    <div id="container">
      <div id="header">
        {% include header %}
      </div>
      <div id="main-content">
        {% include maincontent %}
      </div>
      <div id="footer">
        {% include footer %}
      </div>
    </div>
  </body>
</html>
```

The master page is where all the regions will be included and as you node the variables included has the same name of the region file.
<br/>
The final step to complete our elegant template is registering our regions in the regions.js file that is placed in app/views/default as the following : 

```javascript
module.exports = ['header','maincontent','footer'];
```

Now lets make a controller to view our index page as the following :

```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");
​
new Controller("/").get(function () {
  return new View("index");
});
```

As you see our View Object has a parameter "index" which is what we call a view prefix , this prefix lets the template engine prioritize regions for the viewed page. 
<br/><br/>
To be more specific , the master page includes three regions in our case (header, main content and footer) , by default elegant web template engine will look for the base regions as (header.html, maincontent.html and footer.html), the prefix is passed to the view object to let elegant template engine give priorite to the index specified regions , but how can we specifie a region for a certine prefix ? , simple by adding the prefix to the end of the file such as header.index.html , a huge benefit comes from this which is that you only build the template once and when adding a new page you just specify a new file for the prefix passed in most cases that will mostly happen for the main content region. 
<br/><br/>
Now lets try to add a new page to our app with a new main content to view , so first of all lets add a new controller:


```javascript
var Controller = require("elegant-controller");
var View = require("elegant-view");
​
new Controller("/").get(function () {
  return new View("index");
});
​
new Controller("/new").get(function () {
  return new View("new");
});
```

Lets just specify a new main content region for our new page for the prefix new, all you have to do is copy the maincontent.html and update the html in it to the new html and update the file name to maincontent.new.html , and your done you just added a new page!.



<h1>Managing Static Resources</h1>
The importance of managing static resources comes from it being the biggest frontend performance killer in most cases is the raw number of HTTP requests, and the biggest hammer for addressing it is to package related JS and CSS into larger files, so you send down all the related JS code in one file instead of a lot of smaller ones.
<br/><br/>
Static files has a lot of effect on performance and speed , elegant framework really focus on those issues , the site you are viewing now has 100/100 Google page speed and faster that 99% of the web by alexa.
<br/><br/>
Elegant Manages resources as bundles by grouping related file in a single bundle , also elegant gives you the ability to add some resources inline , in some cases its recommended to add the css and javascript file that is required to view the page inline and load the other files in bundles to avoid a lot of performance issues and really speed things up.
<br/><br/>
To Create a bundle , open the file app/views/default/resources.js and add you new bundle , it should look something like this :

```javascript
module.exports = {
  'base.css': {
    files: ['base.css', 'font-awesome/css/font-awesome.min.css']
  }
}
```

Now the above will create a bundle called "base.css" the is a combination of the files in the files array , this is very good for grouping related resources and keeping everything super clean while getting some real speed to yor application and we used this ourselves to achieve the full mark on google page speed and get our site really fast.

<h1>Caching and Compression</h1>
Most web pages include resources that change infrequently, such as CSS files, image files, JavaScript files, and so on. These resources take time to download over the network, which increases the time it takes to load a web page.

HTTP caching allows these resources to be saved, or cached, by a browser or proxy. Once a resource is cached, a browser or proxy can refer to the locally cached copy instead of having to download it again on subsequent visits to the web page.

Thus caching is a double win: you reduce round-trip time by eliminating numerous HTTP requests for the required resources, and you substantially reduce the total payload size of the responses. Besides leading to a dramatic reduction in page load time for subsequent user visits, enabling caching can also significantly reduce the bandwidth and hosting costs for your site.

To enable caching and compression in you application , just go to your app/config.js file and add the following :
```javascript
// Compressing Resources
config.STATIC.COMPRESS_RESOURCES = true;
​
// Compressing HTML
config.STATIC.COMPRESS_HTML = true;
​
// Cashing in Memory
config.STATIC.DEFAULT_MEMORY_CACHE = true;
​
// Enable Bundles
config.STATIC.COMPACT_RESOURCES= true;
​
// Cache Resources
config.STATIC.CACHE_RESOURCES= true;
​
// Cash HTML
config.STATIC.CASH_HTML= true;

```
To load any Static RESOURCES inline just add the following code in the head of you page using the RESOURCES Object with the name of the bundle you wont to use , such as :
```html
<style type="text/css">
  {
    {
      RESOURCES
      [
      "base.css"
      ]
      .
      content
    }
  }
</style>
​
<script type="text/javascript">
  {
    {
      RESOURCES["base.js"].content
    }
  }
</script>
```