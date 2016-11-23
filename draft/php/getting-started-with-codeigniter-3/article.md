### What is CodeIgniter ?

CodeIgniter is a powerful PHP framework with a very small footprint, built for developers who need a simple and elegant toolkit to create full-featured web applications.

### Who is CodeIgniter For?

* You want a framework with a small footprint.
* You need exceptional performance.
* You need broad compatibility with standard hosting accounts that run a variety of PHP versions and configurations.
* You want a framework that requires nearly zero configuration.
* You want a framework that does not require you to use the command line.
* You want a framework that does not require you to adhere to restrictive coding rules.
* You are not interested in large-scale monolithic libraries like PEAR.
* You do not want to be forced to learn a templating language (although a template parser is optionally available if you desire one).
* You eschew complexity, favoring simple solutions.
* You need clear, thorough documentation.

### How to install Codeigniter ?

1. Download the CodeIgniter from the link [Codeigniter](https://www.codeigniter.com/download) 
2. Unzip the folder.
3. Copy the folder on www folder if you use Wampserver or htdocs if you use Xampp.

### Application Architecture



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b213c382-07fa-45eb-a6c7-6607dd9c7555.gif)


* The index.php serves as the front controller, initializing the base resources needed to run CodeIgniter.
* The Router examines the HTTP request to determine what should be done with it.
* If a cache file exists, it is sent directly to the browser, bypassing the normal system execution.
* Security. Before the application controller is loaded, the HTTP request and any user submitted data is filtered for security.
* The Controller loads the model, core libraries, helpers, and any other resources needed to process the specific request.
* The finalized View is rendered then sent to the web browser to be seen. If caching is enabled, the view is cached first so that on subsequent requests it can be served.

### Model-View-Controller

CodeIgniter is based on the Model-View-Controller development pattern. MVC is a software approach that separates application logic from presentation. In practice, it permits your web pages to contain minimal scripting since the presentation is separate from the PHP scripting.

* The Model represents your data structures. Typically your model classes will contain functions that help you retrieve, insert, and update information in your database.
* The View is the information that is being presented to a user. A View will normally be a web page, but in CodeIgniter, a view can also be a page fragment like a header or footer. It can also be an RSS page, or any other type of “page”.
* The Controller serves as an intermediary between the Model, the View, and any other resources needed to process the HTTP request and generate a web page.









