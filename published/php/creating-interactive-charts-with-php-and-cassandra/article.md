Apache Cassandra is a distributed database with unique benefits that strengthen a developer's toolkit. Cassandra is free and open-source distributed database management system which is designed to manage large amounts of structured data.

In this article, we’ll discuss the step-by-step process of creating charts by fetching chart data from a Cassandra database using FusionCharts, a JavaScript-charting library, and its PHP wrapper.


## Requirements
To render charts in PHP, we need three components, as listed below:

* [FusionCharts Core Package JS Files](https://www.fusioncharts.com/download/)
* [FusionCharts PHP Wrapper](https://www.fusioncharts.com/php-charts/)
* [PHP Driver for Apache Cassandra](http://downloads.datastax.com/php-driver/)

### Including FusionCharts Core JS Files
To include FusionCharts core packages (`fusioncharts.js` and `fusioncharts.charts.js`) in the HTML file, refer to the code below:

```
<html>
  <head>
    <!-- including FusionCharts core package JS files -->
    <script src="path/to/fusioncharts.js"></script>
    <script src="path/to/fusioncharts.charts.js"></script>
  </head>
</html>
```

### Including FusionCharts PHP Wrapper
Refer to the code below to include FusionCharts PHP wrapper (`fusioncharts.php`) in your PHP code.

```
<?php
   // including FusionCharts PHP wrapper
   include("path/to/fusioncharts.php");
?>
```

Once this step is complete, you are all set with the environment. You should be able to work with a Cassandra database on your system.


## Step 1: Establishing and Verifying Connection with Database
Establish the connection with Cassandra database to fetch data for the chart with the PHP code below:

```
$cluster = Cassandra::cluster()->build();
	
$keyspace = 'marathons';

// creating session with cassandra scope by keyspace
$session = $cluster->connect($keyspace);

// verifying connection with database
if(!$session) {
	echo "Error - Unable to connect to database";
}
```

In the above code:

* `Cassandra::cluster()->build();` by default establishes a connection with a localhost cluster. You can also specify remote URL to establish connection with remote cluster. `withContactPoints()` and `withPort()` methods are used to specify IP addresses or hostnames, and port number of the nodes in a given Cassandra cluster. Refer to [this link](http://docs.datastax.com/en/developer/php-driver/1.3/features/#specifying-addresses-of-cassandra-nodes) for more details.
* `$keyspace` holds the name of keyspace (database) from which data will be fetched for the chart.  
* The `$session` variable is used to establish the connection with the database. In case of any connection error, the above code will throw an error message. You can also create a separate `.php` file for this step and include it instead of writing it every time.

**Note**: To use Cassandra in PHP, please make sure you have enabled the necessary extensions by including `extension=php_cassandra.dll` in your `php.ini` configuration file.


## Step 2: Fetching Data and Forming the JSON Array
Now that we have successfully established connection with the database, we will write the query statement to fetch data for the chart.

Refer to the code  below to fetch the data:

```
$statement = new Cassandra\SimpleStatement( 'SELECT id, name, entry_cost, permile_cost, finisher_count FROM topten' );

// query execution - fully asynchronous
$exec = $session->executeAsync($statement);  
			
// getting query result in a variable
$result = $exec->get();
```

Here, `$result` variable holds the data once the above code is executed.

FusionCharts understands both XML and JSON data formats. Since we will be using JSON, we will now append the data (located in `$result` variable) along with FusionCharts chart configurations and parse the end result as an associative JSON array.

Refer to the code below to append data:

```
if($result) {
				
	// creating an associative array to store the chart attributes    	
	$arrData = array(
		"chart" => array(
			"caption"=> "World's Top Marathons",
			"captionFontBold"=> "1",
			"captionFontSize"=> "24",
			"captionFont"=> "Assistant",
			"subcaption"=> "By Entry Cost (In Pounds)",
			"subCaptionFontBold"=> "0",
			"subCaptionFontSize"=> "19",
			"subCaptionFont"=> "Assistant",
			"captionPadding"=> "20",
			"numberPrefix"=> "£",
			"canvasBgColor"=> "#729BDF",
			"bgColor"=> "#729BDF",
			"canvasBgAlpha"=> "0",
			"bgAlpha"=> "100",
			"showBorder"=> "0",
			"showCanvasBorder"=> "0",
			"showPlotBorder"=> "0",
			"paletteColors"=> "#FED34B",
			"showValues"=> "0",
			"decimals"=> "2",
			"usePlotGradientColor"=> "0",
			"baseFontColor"=> "#FFFFFF",
			"baseFont"=> "Assistant",
			"baseFontSize"=> "16",
			"showAlternateVGridColor"=> "0",
			"divLineColor"=> "#DBEAF8",
			"divLineThickness"=> "0.9",
			"divLineAlpha"=> "60",
			"toolTipPadding"=> "7",
			"toolTipBgColor"=> "#000000",
			"toolTipBorderAlpha"=> "0",
			"toolTipBorderRadius"=> "3"
	));
	
	$arrData["data"] = array();
		// iterating over each data and pushing it into $arrData array
		foreach ($result as $row) {
			array_push($arrData["data"], array(
				"label" => $row["name"],
				"value" => $row["entry_cost"]->value(),
				"toolText" => "<b>" . $row["name"] . "</b><hr>Entry Cost: £" . number_format((float)$row["entry_cost"]->value(), 2, '.', '') . "<br> Per-mile Cost: £" .number_format((float)$row["permile_cost"]->value(), 2, '.', '') . "<br>Finishers: " . $row["finisher_count"]->value()
			));
		}
  				
	$jsonEncodedData = json_encode($arrData);
```

In the code shown, an if-statement determines whether the  value of `$result` is valid and creates an associative JSON array to form data for the chart.

The chart object under the `$arrData` variable contains chart configuration options for caption, sub-caption, div lines, values, tooltips, color, etc.

To find out more about customizing the chart cosmetics, you can refer to [this developer documentation page](https://www.fusioncharts.com/dev/chart-attributes.html?chart=bar2d).


## Step 3: Creating the Chart Container
To create an HTML container for the chart, use `<div>` as shown below:

```
<body>
<div id="chart-container"></div>
</body>
```

This tells the browser that all elements within `<div ...>` and `</div>` belong to the same container. 

## Step 4: Creating a Chart Instance and Rendering the Chart
* Now, that we have the chart data and container in place, let’s instantiate the FusionCharts PHP wrapper class and specify details like chart type, chart ID, chart dimensions, chart container ID, and more to render the chart:

```
// creating FusionCharts instance
$toptenChart = new FusionCharts("bar2d", "topChart" , '600', '450', "chart-container", "json", $jsonEncodedData);
```

* Next, call the `render()` method to render the chart. `render()` is a PHP class function that invokes the JS `render()` method to render the chart. Call `render()` as follows:

```
// FusionCharts render method
$toptenChart->render();	
```

We are done with all the steps to render an interactive chart using FusionCharts. The output looks like this:

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/8849f396-2aea-436d-a5ec-5cb0f90b504c.jpeg)

Perfect, isn’t it? If you are having trouble rendering the chart, refer to source code from [GitHub repository](https://github.com/sikrigagan/PHP-Cassandra-Charts).

_____

Thanks for reading! If you found this guide informative or engaging, please add it to your favorites! Otherwise, feel free to leave a comment or feel free to say hi on [Twitter](https://twitter.com/sikrigagan)!
