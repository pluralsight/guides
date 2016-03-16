Here’s the thing about dashboards: No matter what you do, you can’t escape them. Every department inside every company is chasing a goal—if not several—and there’s no better way to monitor a goal’s status than to have a compact dashboard that monitors all related metrics.
 
Just think about it. Marketing is running after leads, sales is running after closing those leads and developers are running to solve problems. There’s a dashboard for all of these cases—and, if you’re a developer, at some point you’ll have to make a dashboard for your company. So, I why not learn how to create one now?
 
In this tutorial, we’ll use JavaScript charts library, offered by [FusionCharts](http://www.fusioncharts.com/), for making charts. But instead of doing it in vanilla JavaScript, we’re going to use React; the hottest JavaScript framework at the moment (and rightly so, if you ask me). In our dashboard we’ll explore the performance of Acme, Inc. in 2015; we’ll see what its top revenue generating countries, monthly revenue trend and highest performing product categories. This is what our final dashboard will look like:
 
![A simple dashboard built in React](http://i.imgur.com/7dnsHHq.png)
 
You can see the live version [here](https://rohitb4.github.io/react-dashboard/) and find the GitHub repo for the project [here](https://github.com/rohitb4/react-dashboard).
 
FusionCharts makes our job a bit easier by providing a dedicated plugin for making [charts in React](http://www.fusioncharts.com/reactjs-charts/#/demos/ex1). Before we jump right in, let’s first explore a bit more about React.
 
### What is React and why should I use it?

React is a JavaScript library for creating reusable View components for the Web. React is fast in that if you want to update your View, you simply re-render the whole View and React will figure out the optimal way to update the Document Object Model (DOM). To achieve this, React uses Virtual DOM and DOM Diff algorithm. You can read more about it on [this StackOverflow thread](http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode).
 
Note: If you want to follow along with this tutorial, please download FusionCharts core library [here](http://www.fusioncharts.com/download/), and its React plugin [here](http://www.fusioncharts.com/reactjs-charts/).
 
### Setup

As mentioned, to create our dashboard we’ll use React and FusionCharts. To abstract out the details of FusionCharts in a React-friendly manner, we’ll use the React Plugin provided by FusionCharts. So, we need to include the JavaScript files of React, FusionCharts and React-FusionCharts plugin. To get started, embed the above three scripts in the HTML page using `<script>` tags.
 
To write applications in React, it’s better if we use JSX syntax. Although it’s not advised, and applications can be created in vanilla javascript, using JSX will give a clear picture of the code and the code will be concise. For that reason we’ll build this dashboard in JSX. You can [read more about JSX [here](https://facebook.github.io/react/docs/jsx-in-depth.html).

```react.js
<script src='https://fb.me/react-with-addons-0.14.6.js'></script>
<script src='https://fb.me/JSXTransformer-0.13.3.js'></script> // React Scripts

<script src='js/fusioncharts.js'></script>
<script src='js/fusioncharts.charts.js'></script>
<script src='js/fusioncharts.powercharts.js'></script> // FusionCharts files
 
<script src='js/react-fusioncharts.min.js'></script> // React FusionCharts plugin

<script src='js/fusioncharts.theme.carbon.js'></script>  // theme file
```
 
Note: Including the JSX Transformer is only suggested for development applications. For production-ready code, precompile the JSX and you won’t need to include this file.
 
With the basic setup ready, we’ll add some markup and React JSX code to create the dashboard.
 
### Creating HTML page for the dashboard

In the dashboard we’re building, we’re creating three visualizations of various revenue metrics of Acme Inc. We’ll create a column chart to show revenues from top six revenue generating countries, a spline chart (smoother version of line chart) to show the revenue distribution over various months of the year and a donut chart to figure out which product category generated the highest percentage share of the revenue. In short, we’ll render column, spline and donut charts. vWe will create the column chart spanning the entire width of the page, and spline and donut charts in a two column layout to span 50 percent width. The following HTML and CSS would create HTML container to create this layout. The HTML containers are where the charts will be rendered.
 
HTML:
```
<div class="chart-row">
        	<div id="country-revenue"></div>
</div>
 
<div class="chart-row">
        	<div id="monthly-revenue" class="inline-chart">
        	</div>
        	<div id="product-revenue" class="inline-chart">
        	</div>
</div>
```
 
CSS:
```
.chart-row {
  margin-bottom: 15px;
}
.inline-chart {
  display: inline-block;
  width: 48%;
  margin-left: 1%;
}
```
 
Now we’ll prepare the data for these three charts and render them in the containers we have just created.
 
### Creating the React component

With the HTML structure ready, we will create a root React component based on our HTML structure. We make the HTML structure into a valid JSX and return it in the render function of the component. To achieve that, we make a few changes like changing `class` to `className`.
 
This is the react component that we create:

```
var DashboardApp = React.createClass({
  render: function() {
	return (
  	<div>
    	<h1 className="main-title">Acme Inc. Revenue Analysis for 2015</h1>
    	<div id="interactive-dashbaord"></div>
    	<div className="chart-row">
      	<div id="country-revenue">
        	// country revenue chart here
      	</div>
    	</div>
    	<div className="chart-row">
          <div id="monthly-revenue" className="inline-chart">
        	// chart 2 here - spline
      	</div>
      	<div id="product-revenue" className="inline-chart">
        	// chart 3 here - donut
      	</div>
    	</div>
  	</div>
	);
  }
});
```
 
Next we render this component into an HTML element:

```
React.render(
        	<DashboardApp />,
        	document.getElementById("dashboard")
);
```
 
This is an empty react component that we have created and rendered. We need to create the charts in it and render them in the skeleton markup that we’ve designed (we’ll cover this in the following sections, so sit tight!).
 
### Creating multiple chart components

Now, let’s go through the process of creating one chart (column) in detail. Creating the other two charts is similar and you’ll notice the differences in the description below.
 
### Preparing data for the chart

Every charting library has a slightly different input data format. Since we’re using FusionCharts for our project, we need to format our data in a format that FusionCharts understands. It accepts JSON data as an array of objects containing label and value. This is what the syntax looks like:
 
```
[{
        	"label": "Orange",
        	"value": "90"
},{
        	"label": "Yellow",
        	"value": "60"
}...]
```
 
### Creating the chart configuration

Chart configuration contains various attributes of the chart-like chart type, caption, sub-caption and theme, to name a few. It also contains the data to be plotted. Remember those differences in attributes I just mentioned? These include data, chart type, height and width. There are other theme-related attributes as well, which will be mostly the same for all three charts.
 
A sample chart configuration looks something like this:

```
{
  type: "doughnut2d",
  renderAt: "product-revenue",
  width: '100%',
  height: 400,
  dataFormat: "json",
  dataSource: {
	chart: {
  	caption: "Top 5 stores in last month by revenue",
  	theme: "carbon"
	},
	data: [{
        "label": "Jan",
        "value": "657000"
  	}, {
        "label": "Feb",
        "value": "138000"
  	},
  	... {
        "label": "Dec",
        "value": "730000"
  	}
	]
  }
}
```
 
### Creating a chart using the React plugin

With the chart configuration ready, we can create the React chart. We’ll use the React plugin API to create it. The only input to the plugin is the configuration that we created earlier. We can simply create it using the following code.

```
<react_fc.FusionCharts {...countryChartConfigs} />
```

The chart we created above is a column chart, the other two charts are spline and donut. We’ll create chart configurations for these two and mention the right types and create the React chart component.

```
<react_fc.FusionCharts {...monthlyChartConfigs} />
<react_fc.FusionCharts {...productChartConfigs} />
```
 
Now we need to place the three chart components appropriately in the JSX of the `DashboardApp` we created earlier. This is the new JSX after placing the chart components.
 
```
<div>
  <h1 className="main-title">Acme Inc. Revenue Analysis for 2015</h1>
  <div id="interactive-dashbaord"></div>
  <div className="chart-row">
	<div id="country-revenue">
      <react_fc.FusionCharts {...countryChartConfigs} />
	</div>
  </div>
  <div className="chart-row">
	<div id="monthly-revenue" className="inline-chart">
      <react_fc.FusionCharts {...monthlyChartConfigs} />
	</div>
	<div id="product-revenue" className="inline-chart">
      <react_fc.FusionCharts {...productChartConfigs} />
	</div>
  </div>
</div>
```
 
After adding this you should see the dashboard rendered on your page. If you had any issues or want to see the source code of the project, head over to project’s [GitHub repo](https://github.com/rohitb4/react-dashboard).
 
#### Additional resources

Using the process I described above, you can create relatively sophisticated charts and dashboards in React. Of course, we all come across issues when we try to build things on our own. To help you out in times like these, I’ve listed some resources below.
 
- Official plugin page and GitHub repo: Here are handy links for [React charts](http://www.fusioncharts.com/reactjs-charts/#/demos/ex1) official plugin page and its [GitHub repo](https://github.com/fusioncharts/react-fusioncharts). These are the first places you should head if you have any issues with the plugin, as they contain all the relevant information.
- Documentation: You can find the official documentation for the plugin [here](http://www.fusioncharts.com/dev/using-with-javascript-libraries/reactjs/introduction.html). It covers everything from getting started to some advanced topics like adding annotations to the chart.
- Make the dashboard more interactive: I made a relatively simple dashboard in this tutorial, as the objective was to get you started with making charts in React. But there is a lot of interactivity you can add to the charts using [events](http://www.fusioncharts.com/dev/api/fusioncharts/fusioncharts-events.html) and [methods](http://www.fusioncharts.com/dev/api/fusioncharts/fusioncharts-methods.html) provided by FusionCharts. Check out this [advanced dashboard](http://fusioncharts.github.io/react-fusioncharts/#/dashboard) depicting Global Terrorism stats for some inspiration (and yes, it was built using React and FusionCharts).
 
Feel free to post any questions you might have about this article in the comments section below and I’ll be sure to get back to you as soon as I can.
 
