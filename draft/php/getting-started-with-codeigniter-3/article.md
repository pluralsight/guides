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


### Configuration

* .htaccess file
update the .htaccess file of the application folder and put this code


```php
RewriteEngine on
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule .* index.php/$0 [PT,L]
```
* config file 
 
 CodeIgniter has one primary config file, located at application/config/config.php
    

```php
/* Base URL Configuration  */
$config['base_url'] = 'http://localhost/test/';

 /* Remove index file from the URL  */ 
$config['index_page'] = '';
```
* autoload file 

the role of the autoload file is to permits libraries, helpers, and models to be initialized automatically every time the system runs.

```php
/* Libraries load  */
$autoload['libraries'] = array('session','database','form_validation');

 /* Helpers load  */ 
$autoload['helper'] = array('url','text','form','date');
```
* Database file 

the role of the database file is lets you store your database connection values (username, password, database name, etc.).

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

Now let's talk about the example of our guides is showing users list from the table of the database let's start with the model 
### Table Structure 

#### User 

| id   |      username      |  password |
|----------|:-------------:|------:|
| 1 |  Adib | 123456
| 2|    Wadii   |   abcdef |
| 3 | Ayoub |    azerty |

### Model

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

    
```php
class user extends CI_Controller {

    public function __construct() {
        parent::__construct();
        $this->load->model('user_model') ;
    }
    
     public function user_list() {
        $data = array();
        $tab = $this->User_model->user_list();
        $this->load->view('list', $data);
    }
}
```

### View
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


