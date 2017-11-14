
Pygal is amazing for creating customizable SVG's that can easily scale.  



<br>
<br>

You can easiely and minimally create line charts, bar graphs, even Radar Chars with very little code. 
<br>
<br>

---
#### <b>Let's Begin with an Install</b>

Open Terminal
```
$  pip install Pygal
```
<b>Extra Help:</b>  Installing and Dependencies  - http://www.pygal.org/en/stable/installing.html

---

#### <b>Step 1:  Import</b>

```
import pygal
```

#### <b>Step 2: Create a Variable to Store Your Graph</b>
For our first example we'll create a bar chart.  We simply need to create a varible and store ```pygal.Bar()``` inside.

```
b_chart = pygal.Bar()
```
You can easily use ```pygal.Line```, ```pygal.pie``` etc.


#### <b>Step 3: Let's Add Some Values</b>
Next we need to start creating our chart.  I will be using Data that I scraped from a gaming tracker for the game <b>Destiny 2.</b>  Eventually the graph will be live and I can see my stats change, hopefully in real time.  <i>Let's not get ahead of our selves.</i>

We need to give our chart a title.
```
b_chart.title = "Destiny Kill/Death Ratio"
```
Now we can start adding some data. I need 3 bars for each player.
```
b_chart.add("Dijiphos", [0.93])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
```

Technically we can be finished without further customization.  We can now render this quickly to a browser using ```render_in_browser()```:
```
b_chart.render_in_browser()
```
#### <b> Example</b>


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/af93de40-4e61-4329-93b3-835e127e77cc.gif)













