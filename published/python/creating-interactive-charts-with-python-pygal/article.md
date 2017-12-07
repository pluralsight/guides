<br>
# A Quick Introduction to Pygal

When it comes to interactive charts and the modules you can use, you may be familiar with Matplotlib, Seaborn, and Bokeh, which each allow interactivity and support additional features. 

However, Pygal specializes in allowing the user to create SVGs. Besides the scalability of an SVG, you can edit them in any editor and print them in very high quality resolution. SVG formatting is easily integrated with Flask and Django as well. Lastly, you can easily and minimally create line charts, bar graphs, and Radar Charts (_to name a few_) with very little code, as we'll see shortly with the following examples:

<details>
<summary><b>```Click``` Pie Example</b></summary>
<br>



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/fb8f5685-d93f-4876-ace5-80e69d81a402.gif)


</details>

<details>
<summary><b>```Click```  Dot Example</b></summary>
<br>



![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ee93583e-d848-451b-b23c-94b927e13caf.gif)

</details>

<details>
<summary><b>```Click``` KPI Example</b></summary>
<br>




![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/567bc7bb-b0a0-4047-9654-59c91e868144.gif)


</details>


# Installation

Open Terminal
```
$  pip install Pygal
```
---
<b>Extra Help:</b>  Installing and Dependencies  - [Link](http://www.pygal.org/en/stable/installing.html)

---

# Using Pygal

### Step 1:  Import

```
import pygal
```

### Step 2: Create a Variable to Store the Graph

For our first example we'll create a bar chart.  We'll simply need to create a variable and store ```pygal.Bar()``` inside.

```
import pygal
b_chart = pygal.Bar()
```
You can easily use ```pygal.Line```, ```pygal.pie```, or any of the [following](http://www.pygal.org/en/stable/documentation/index.html).


### Step 3: Add Some Values

Next we need to start creating our chart.  I will be using Data that I scraped from a gaming tracker for the game <b>Destiny 2.</b>  Eventually the graph will be live and I'll be able to see my stats change (_hopefully_) in real time. 

---

<details>
<summary><b>```Click``` Game Play + Graph Change Real Time Example</b></summary>
<br>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/1ea09d8f-05e0-4814-9255-8ef430d34200.gif)
</details>

---

<i>Let's not get ahead of ourselves.</i> <br>In short, everytime I play in a PvP or Crusible Match and eliminate an opponent or end up getting eliminated myself, the KD (_Kill/Death Ratio_) will change.  I simply want to compare myself to my clan mates.

Thus, to start things off, we'll need a chart title.

```
import pygal
b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
```
Now we can start adding in our data. I need 3 bars, one for each player. To accomplish this, I'll need to use ```add``` followed by a title and some values.  
```
import pygal
b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
```

Technically, we can finish and render without further customization.  To render quickly to a browser, we'll use ```render_in_browser()``` as our output. 
```
import pygal
b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```
---
<details>
<summary><b>```Click``` To Reveal Our Example</b></summary>
<br>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ec4a6f0d-9cde-4785-8a76-2b819df70775.gif)
</details>

---


## Customizing the Graph
Say we want some custom colors added to our graph<br>
![#E80080](https://placehold.it/15/e80080/000000?text=+) `#E80080`
![#404040](https://placehold.it/15/404040/000000?text=+) `#404040`
![#9bc850](https://placehold.it/15/9bc850/000000?text=+) `#9BC850`

This is easy to do, and can actually be achieved in multiple ways.
First, import ```style``` ```from``` ```pygal.style```.
```
import pygal
from pygal.style import Style

b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```
You can change a number of objects by simply adding:
```
import pygal
from pygal.style import Style
custom_style = Style(

b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```
_Notice I left the parentheses open_.

---
<details>
<summary><b>```Click``` To Reveal Some Objects</b></summary>
<br>
Properties & Description<br>

```plot_background ```The color of the chart area background<br>
``` background```The color of the image background<br>
```foreground ```|The main foregrond color<br>
``` colors```The serie color list<br>
``` value_colors```The print_values color list<br>
Complete List: http://www.pygal.org/en/stable/documentation/custom_styles.html
</details>

----

Let's change the color of our bars by using the object ```colors```.<br>_Don't forget to indent and close our final parentheses._
```
import pygal
from pygal.style import Style
custom_style = Style(
  colors=('#E80080', '#404040', '#9BC850'))
  
b_chart = pygal.Bar()
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```
Now, all we need to do is pass ```style=custom_style``` in our ```pygal.Bar()``` to get it to work.
```
import pygal
from pygal.style import Style
custom_style = Style(
  colors=('#E80080', '#404040', '#9BC850'))
  
b_chart = pygal.Bar(style=custom_style)
b_chart.title = "Destiny Kill/Death Ratio"
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```

---
<details>
<summary><b>```Click``` To Reveal Our Example</b></summary>
<br>


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f522dcab-e4f3-4a1e-9e94-0e318634c7ee.gif)
</details>

----
#### That's a Wrap

It's that simple.  I would check out [Pygal's](http://www.pygal.org/en/stable/installing.html) documentation for further builds.  However, after reading this guide, you should have all the knowledge needed to create visually appealing graphs and charts with Python Pygal. 
___
Thanks for reading this guide on using the Pygal module. Leave a comment with your favorite Pygal graph or reach out and say hi on [Twitter](https://twitter.com/TroyKranendonk). And, of course, please add this guide to your favorites if you found it informative or engaging.