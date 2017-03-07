# What is CodeIgniter ?

CodeIgniter is a powerful PHP framework with a very small footprint, built for developers who need a simple and elegant toolkit to create fully-featured web applications.

### Who is CodeIgniter For?

CodeIgniter is for you if:

* You want a framework with a small footprint.
* You need exceptional performance.
* You need broad compatibility with standard hosting accounts that run a variety of PHP versions and configurations.
* You want a framework that requires nearly zero configuration.
* You do not want to rely on the command line.
* You want a framework that does not impose restrictions on the way you code.
* You do not need large-scale libraries like PEAR.
* You do not want to learn a templating language (although a template parser is optionally available if you desire one).
* You need clear, thorough documentation.

### CodeIgniter setup

1. Download CodeIgniter from [codeigniter.com](https://www.codeigniter.com/download) 
2. Unzip the folder.
3. Copy the folder on `www folder` if you use Wampserver or `htdocs` if you use [Xampp](http://www.phpknowhow.com/basics/working-with-xampp/).

# Basics

### Application Architecture



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/b213c382-07fa-45eb-a6c7-6607dd9c7555.gif)


* `index.php` serves as the front controller, initializing the base resources needed to run CodeIgniter.
* The router examines each HTTP request to determine what should be done with it.
* If a cache file exists, the request is sent directly to the browser, bypassing the normal system execution.
* Before the application controller is loaded, the HTTP request and any user-submitted data is filtered for security.
* The application controller loads the model, core libraries, helpers, and any other resources needed to process the specific request.
* The finalized View is rendered and sent to the web browser to be seen. If caching is enabled, the view is cached first so that on subsequent requests it can be served.

### Model-View-Controller (MVC) Basics

CodeIgniter is based on the Model-View-Controller development pattern. MVC is a software approach that separates application logic from presentation. In practice, it permits your web pages to contain minimal scripting since the presentation is apart from the PHP.

* Model: the model represents your data structures. Typically your model classes will contain functions that help you retrieve, insert, and update information in your database.
* View: the View is the information that is being presented to a user. A View will normally be a web page, but in CodeIgniter, a view can also be a page fragment like a header or footer. It can also be an RSS page, or any other type of “page”.
* Controller: the Controller serves as a connector between the Model, the View, and any other resources needed to process the HTTP request and generate a web page.


### Configuration

* `.htaccess` file
update the `.htaccess` file of the application folder to include this code:


```php
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule .* index.php/$0 [PT,L]
```
* Config file 
 
 CodeIgniter has one primary config file, located at `application/config/config.php`
    

```php
/* Base URL Configuration  */
$config['base_url'] = 'http://localhost/test/';

 /* Remove index file from the URL  */ 
$config['index_page'] = '';
```
* Autoload file 

The autoload file allows libraries, helpers, and models to be initialized automatically every time the system runs.

```php
/* Libraries load  */
$autoload['libraries'] = array('session','database','form_validation');

 /* Helpers load  */ 
$autoload['helper'] = array('url','text','form','date');
```
* Database file 

The database file lets the developer store database connection values (username, password, database name, etc.).

```php
$db['default'] = array(
	'dsn'	=> '',
	'hostname' => 'localhost',
	'username' => 'root',
	'password' => '',
	'database' => 'test',
	'dbdriver' => 'mysqli',
	'dbprefix' => '',
	'pconnect' => FALSE,
	'db_debug' => (ENVIRONMENT !== 'production'),
	'cache_on' => FALSE,
	'cachedir' => '',
	'char_set' => 'utf8',
	'dbcollat' => 'utf8_general_ci',
	'swap_pre' => '',
	'encrypt' => FALSE,
	'compress' => FALSE,
	'stricton' => FALSE,
	'failover' => array(),
	'save_queries' => TRUE
);
```

# Example


Now let's try out an example. We want to show a list of users from a data table. Let's start with the model.

### Table Structure 

#### User 

| id   |      username      |  password |
|----------|:-------------:|------:|
| 1 |  Adib | 123456
| 2|    Wadii   |   abcdef |
| 3 | Ayoub |    azerty |

### Model

We see that there are a few properties to account for with each user. For example, every user has an `id`, `username`, and `password`. These will become our Model attributes. Remember that the Model handles the back-end, per se. Therefore, any operations dealing with table manipulation should be completed in this class (or its related non-View, non-Controller classes).

```php
class User_model extends CI_Model {

    public $id;
    public $username;
    public $password;
    
    function getId() {
        return $this->id;
    }

    function getUsername() {
        return $this->username;
    }

    function getPassword() {
        return $this->password;
    }

    function setId($id) {
        $this->id = $id;
    }

    function setUsername($username) {
        $this->username = $username;
    }

    function setPassword($password) {
        $this->password = $password;
    }

    
    function __construct() {
        parent::__construct();
    }
    
    public function user_list() {
        return
                        $this->db->select('*')
                        ->from('user')
                        ->order_by('id_user', 'DESC')
                        ->get()
                        ->result_array();
    }
}
```


### Controller

Next we will consider the controller. Remember that the controller relays information and requests between the model and the view. We want to transmit all changes going from model to view and vice versa while keeping abstraction barriers intact (i.e. while keeping the user away from the inner pieces of the model itself). 
    
```php
class user extends CI_Controller {

    public function __construct() {
        parent::__construct();
        $this->load->model('user_model');
    }
    
    public function user_list() {
        $data = array();
        $tab = $this->User_model->user_list();
        $this->load->view('list', $data);
    }
}
```
The constructor for the controller loads the `model` from `user_model` in order to retrieve the values in the table itself. As seen by the `user_list()` function, the controller is going between the `User_model` class and loading `view` elements.

### View

Lastly we have the view, which dictates what the user sees. We will use HTML to place users into each spot in the HTML table (denoted by the `<table>` tags). Notice that we use `$tab` in the `foreach`. This tells the view to construct a table containing values from the `user_list()`. Furthermore, by using `foreach` we can expand the table if more users are added.

```php
<html>
<head>
<title>Users list</title>
</head>
<body>
<table>
<thead>
<tr>
<th>ID</th>
<th>Username</th>
<th>Password</th>
</tr>
</thead>
<tbody>
<?php 
    foreach($tab as $key => $value){
        echo "<tr>" ; 
            echo "<td>";
            echo $value["id"] ; 
            echo "</td>";
            echo "<td>";
            echo $value["username"] ; 
            echo "</td>";
            echo "<td>";
            echo $value["password"] ; 
            echo "</td>";
        echo "</tr>" ; 
    }
?>
</tbody>
</table>
</body>
</html>
```

And with that, we have a responsive, script-based table that shuffles between Model, View, and Controller.

# Conclusion

In this tutorial, we covered the basics of application architecture, Model-View-Controller, and writing short, quick scripts using CodeIgniter. Hopefully this guide demonstrated CodeIgniter's very straightforward setup and configuration process as well as its ease of use. 

Let me know in the comments section below if you found this guide useful or if there are any cool tips and tricks for using CodeIgniter. Thanks for reading.


