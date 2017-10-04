Most enterprise apps currently use relational databases like SQL, MariaDB, or MySQL due to their popularity and stable nature. However, developers face issues when they try to scale such databases. Furthermore, considering the recent breed of web applications that handle large data sets, developers are constantly looking for more scalable databases. This has attributed to the rise of non-relational (NoSQL) databases. One such database that has become really popular is MongoDB.

In this tutorial we will follow a step-by-step approach to create charts using data stored in a MongoDB database. We will use the PHP scripting language to connect to the database and fetch the data, which would then be used to render the chart. 

We picked PHP over others as it comes with a MongoDB driver that connects it to the database. If you need to add more firepower to your web application, you can also use Node.js.

**Now that we have an overview, let’s get started...**

For creating charts using PHP and MongoDB, you need the following to be downloaded and installed on your system:
* XAMPP
* MongoDB
* PHP driver for MongoDB
* Composer

 
### Part 1: Including Dependencies
To render FusionCharts in PHP using MongoDB, we need to include following dependencies:
* FusionCharts Suite XT ([Download Link](https://www.fusioncharts.com/download/)):  Download FusionCharts Suite XT zip file and store all the extracted script files in a new folder inside the project folder, as shown below. Here, we have named the new folder as `fusioncharts`.
Include it inside the HTML section of the main page (`index.php`), using `<script>` tag.

```
<head>
    <script src="fusioncharts/fusioncharts.js"></script>
    <script src="fusioncharts/fusioncharts.theme.fint.js"></script>
</head>
```

* FusionCharts PHP Wrapper ([Download Link](https://www.fusioncharts.com/php-charts/)) : Extract the fusioncharts PHP wrapper and save the `fusioncharts.php` file inside the same folder created in the previous step for keeping the script files (for our example, `fusioncharts`). Include it in the PHP code section of the main page (`index.php`).

```
require("fusioncharts/fusioncharts.php")
```

Next, we move on to the database part.


### Part 2: Connecting Database and Verifying the Connection
* Database: We have data stored in database for monthly petrol and diesel prices. However, you can also use your own data to render the chart.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6fa9d5b5-2bf0-4a0e-890b-283639bcdd34.png)

* Configuration file: It is a good practice to separate the config file. Hence, create a file and save it as `dbconn.php` inside our project folder.

* Include the PHP MongoDB Driver Library: Include the `autoload.php` file in `dbconn.php` which is required for using mongodb with php.

```
require 'vendor\autoload.php';
```

* Establish Connection with Database: Next, we establish the connection between php and MongoDB.

```
$connection = new MongoDB\Client;
```

After completion of the configuration, `dbconn.php` will look like this.

```
<?php
    require 'vendor\autoload.php';
    $connection = new MongoDB\Client;
?>
```


### Part 3: Fetching Data and Rendering the Chart
* Main page: In `index.php` include `dbconn.php` file using the command shown below:

```
require 'doconn.php';
```

* Fetch Data and Form an Associative Array: Include the php code within the body tag. First, we will retrieve the data from the database and store it in a variable. 

```
// retrieving data from the database
$db = $connection->myProject;

$myObj = $db->chartData->find();
```
 
Here we are storing the data in a variable called `$myObj` and `chartData` is the name of collection in our database `myProject`.

Now lets convert data fetched into an associative array.

```
// convert mongoCursor into an array
$data=iterator_to_array($myObj);

// sorting the array
asort($data);
```

