#Using ​a Spreadsheet​ ​to Power Charts in AngularJS Apps



AngularJS and Google Sheets are all the rage nowadays. Both are being used very commonly, and developers are finding interesting use cases around them. Today I am going to share something I have built that is cool and may have some practical applications as well.

We will build an AngularJS application to create interactive JavaScript charts which will contain data exported from Google Sheets in real-time. First, I will briefly talk about [AngularJS](https://angularjs.org), [FusionCharts](http://www.fusioncharts.com) and [Google Sheets](https://www.google.com/sheets/about/). And then we’ll put all of these together to build our project. If you are already aware of these technologies, you can skip the introductions and dive right into the tutorial.

##AngularJS
AngularJS is one of the most popular JavaScript web frameworks to create single page web applications. Angular allows you to declare components directly in the HTML with the help of Angular directives. Angular’s declarative way with help of [AngularJS directives](https://docs.angularjs.org/guide/directive), and two way binding makes developing web applications a breeze. It is also pretty easy to maintain and test AngularJS applications.

##FusionCharts
As I mentioned above, we are going to use FusionCharts’ [JavaScript charts](http://www.fusioncharts.com) library for this project. It has an exhaustive collection of charts and graphs. With over 90+ chart types and 965 maps, you’ll find everything that you need right out of the box. It not only supports modern browsers, but also older browsers starting from IE6.

FusionCharts supports both JSON and XML data formats, and you can export charts in PNG, JPEG, SVG or PDF. It offers some nifty [plugins and wrappers](http://www.fusioncharts.com/extensions/), one of which we are going to use today - AngularJS.

##Google Sheets
[Google Sheets](https://www.google.com/sheets/about/) (or Google Spreadsheets) is a web-based app that allows you to create, update and modify spreadsheets and share the data live online. It is basically like an earlier version of MS Excel in cloud. But it is steadily becoming better, and I have no doubt that it will become the default spreadsheet app in near future.

Best part you ask? You can export the data present in your sheet via JSON! And that’s exactly the feature we need for this project. Once you set up your sheet to export data in JSON format, it opens up a wide array of possibilities. And creating interactive charts is just one of those that we are going to explore.

A quick look into what we will be making (see the [live CodePen demo here](http://codepen.io/brohit4/pen/gPEMwQ?editors=1010)):

![Donut Chart](donut-chart.png)

Now, let’s get started with the tutorial!

##Setting the Spreadsheet
For this tutorial, I am going to create a sheet with two columns of data. One column will contain the labels and the other column will contain values corresponding to the those labels. You can see the sample sheet for this project here: [https://docs.google.com/spreadsheets/d/187iKB7ekhP96evCySKBWy5LfWErrJDono-8glzFPcCY/edit#gid=0]. 


##How to Extract Data Present in Google Sheets
To be able to create a chart from Google Sheets, we need to somehow fetch the data present in our sheet. Luckily for us, we just need to make some changes in settings of the sheet.

The default setting of the Google Sheet makes the sheet not accessible on the web without authentication. So, first we need to modify this setting. To do that we need to change the sharing settings to **Anyone with the link** or **Public on the web**. See the steps below for reference.


- Step-1: Click ‘Share’ button on top right corner of your sheet. You should see this screen after clicking:
![step-1](step-1.png) 
- Step-2: Click on ‘Advanced’ on the bottom right corner of above screen. You will get this:
![step-2](step-2.jpg)
- Step-3: Right now above screen says ‘Specific people can access’. To change this click ‘Change...’. Choose ‘On  - Anyone with the link’ on the screen that appears (see below).
![step-3](step-3.png)
    
Now our sheet is accessible to anyone who has a link to it. But we are not done yet. To make the data available in different formats like JSON or XML for the chart, we need to publish the Google Sheet to the web. Click on ‘File’ >> ‘Publish to the web...’. Then click ‘Publish’ button on the screen that appears:

![publish to web](publish-to-web.png)

Now our sheet is all set up to make some kickass charts! With these settings made, the JSON data should be available through this URL: [https://spreadsheets.google.com/feeds/list/187iKB7ekhP96evCySKBWy5LfWErrJDono-8glzFPcCY/od6/public/basic?alt=json]. 


Above URL is of this format:

```
spreadsheets.google.com/feeds/list/{{ID}}/od6/public/basic?alt=json
```

Where ***‘id’*** is the spreadsheet id. Every spreadsheet has a unique id and you can replace my sheet’s id with yours in the live [CodePen demo](http://codepen.io/brohit4/pen/gPEMwQ?editors=1010) to plot your data

##Creating an AngularJS Application
Creating an AngularJS application is pretty easy after the initial learning curve, but it might look overwhelming at first glance. We will not go through the process of creating an AngularJS application in detail here, but most of the AngularJS concepts can be looked over here: [https://docs.angularjs.org/guide/](https://docs.angularjs.org/guide/).

To create a bare minimum Angular application, we need to include AngularJS library, template file (HTML) and a JS file which creates the application. The template will contain HTML as well as some Angular directives. The app will create a controller which is responsible for fetching the data and binding the data to the template. The linking between the controller and template happens through a couple of directives - `ng-app` and `ng-controller`.

This is the `<script>` tag that needs to be added to include minified AngularJS 1.5.0 file from CDN:


```
<script type=”text/javascript” src=”https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.0-rc.2/angular.min.js”></script>
```


This is the HTML we would need for the app:

```
<div ng-app="chartApp" ng-controller="chartController">
</div>
```


And finally, add this piece of code in a JavaScript file to define the angular app and controller:

```
var myApp = angular.module('chartApp',[]);

myApp.controller('chartController', ['$scope', function($scope) {
    // Further code will be added here
});
```

Now that we have bootstrapped the angular application, we’ll proceed to creating the chart.

##Creating the Chart
To create the chart, first we will fetch the data present in the our spreadsheet through an AJAX call. After the AJAX request is successful, we will restructure spreadsheet data and create the chart using FusionCharts. 

To the make the AJAX call, we will use jQuery’s `$.get()` method.


###Fetching the Data
To include jQuery, add the following script tag:

```
<script type=”text/javascript” src=”https://code.jquery.com/jquery-1.12.0.min.js
```


Now we will make an AJAX call to the Google sheet that we created using jQuery’s `$.get()` method in the controller of our application.

```
$.get({
    url:'https://spreadsheets.google.com/feeds/list/187iKB7ekhP96evCySKBWy5LfWErrJDono-8glzFPcCY/od6/public/basic?alt=json',

    success: function (revenueData) {

    // data processing and chart creation to go here

});
```



###Parsing the Data
The data that we fetched from spreadsheet has to be converted into the format that FusionCharts understands. FusionCharts accepts JSON data as an array of objects containing `label` and `value`. This is how it looks like:

```
[{
    “label”: “Potato”,
    “value”: “80”
},{
    “label”: “Tomato”,
    “value”: “70”
}...]
```

To achieve this, we need to run following piece of code in the success function of the jQuery `get` function we defined earlier.

```
var data = revenueData.feed.entry,
    revenueDataLength = data.length,
    chartData = [],
    datum;

for (var i = 0; i < revenueDataLength; i++) {
    datum = data[i];
    chartData.push({
        label: datum.title.$t,
        value: datum.content.$t.replace('revenue: ', '')
    });
}
```

Explanation on above code: we are using a `for` loop to iterate over each spreadsheet’s data object present in the `revenueData.feed.entry`. We are doing this to extract `label` and `value` out of it. Then we are storing the extracted values in a new array called `chartData`, which will be used by the chart.

The `label` is available in the `title.$t` key and the value is available in `content.$t` key of the data object. The value we extracted contains the spreadsheet’s column name, which is not necessary, and hence we remove it. The rest of the data in the spreadsheet is not needed for our project.

`chartData` now contains data in a format which will be understood by FusionCharts. Finally, we are ready to create the chart!


###Rendering the Chart
To create the chart we will use the FusionCharts’ [AngularJS charts](http://www.fusioncharts.com/angularjs-charts/) plugin. The plugin exposes FusionCharts through an AngularJS directive which makes creating the chart in AngularJS little easier. To learn more about the plugin or see some live demos, head over to [official page](http://www.fusioncharts.com/angularjs-charts/). For this tutorial, I have hosted the plugin on Google Drive. 

We will be creating a 3D doughnut chart, so we just need FusionCharts core library - `fusioncharts.js`.

We will include AngularJS, jQuery, FusionCharts, and FusionCharts’ AngularJS plugin using HTML <script> tags:

```
<script src='https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.5.0-rc.2/angular.min.js'></script>
<script src='https://code.jquery.com/jquery-1.12.0.min.js'></script>
<script src='/location/of/fusioncharts.js'></script>
<script src='/location/of/angular-fusioncharts.min.js'></script>
```

After including these dependencies, we modify the app that we have created to include `ng-fusioncharts` module which is the plugin we are using.

```
var myApp = angular.module('chartApp',["ng-fusioncharts"]);
```

And now, we need to add the plugin’s directive into the template:

```
<fusioncharts
    width="100%"
    height="400"
    type="doughnut3d"
    datasource="{{sheetDataSource}}">
</fusioncharts>
```

This code tells the plugin to create the `doughnut3d` chart using the `sheetDataSource` object and the dimensions specified. Once we set the `sheetDataSource` object after fetching the data, the chart will be rendered. The following code in the success callback sets the `sheetDataSource`:

```
$scope.$apply(function(){
  
  $scope.sheetDataSource= {
      
    chart: {
        "caption": "Revenue Split",
        "captionFontColor": "#000",
      // more chart cosmetics
    },
        
    data: chartData

  };

});
```

Here we are wrapping the setup of `sheetDataSource` in `$scope.$apply` to trigger a watcher on it and update the chart. This is done because by default Angular doesn’t update the UI, once the variable is updated, we have to trigger it manually. Much more about `$apply` [here](https://docs.angularjs.org/api/ng/type/$rootScope.Scope).

In the above code snippet I have only displayed only two attributes - `caption` and `captionFontColor`, but there are literally hundreds of attributes that FusionCharts offers to customize a chart. It is not possible to define all attributes here, so I have only explained few that are most important.

- `baseFont`: You can use `baseFont` attribute to adjust the font family being used on your chart. You can use any font under the sun. In my example, I have used ‘Open Sans’. Just include the relevant font file in your HTML and you are all set to start using it in your chart
- `paletteColors`: You can define a set of colors in hex-code using this attribute. Biggest pie will take the first color and next one will take second and so on.
- `toolTipBgColor`: This attribute controls the background color of the tooltip. You can use similar attributes like `toolTipPadding` and `toolTipBgAlpha` to control padding and background transparency respectively.

For more, you can visit [chart attributes page for doughnut chart](http://www.fusioncharts.com/dev/chart-attributes.html?chart=doughnut3d).

##More Resources
I hope you found my tutorial useful and can make use of it in making some real-life applications. But a tutorial like mine can’t explain everything. It is just the beginning and you will have to explore further on your own if you want to take this further. To help you in your journey, here are some resources that might help:

- Documentation: If you want to do anything other than what I described above, [AngularJS developer guide](https://docs.angularjs.org/guide) and [FusionCharts’ developer center](http://www.fusioncharts.com/dev/) should be your first points of reference.
- GitHub page for the plugin: If you are ever stuck, you can catch the developers of the plugin on its [GitHub page](http://fusioncharts.github.io/angular-fusioncharts/).

In case you have any questions, feel free to leave a comment below or [say hello on Twitter](https://twitter.com/4two2)!