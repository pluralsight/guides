# Creating Graphs w/ Python Pygal


Py Gal - works well with Django Flask etc. 
+ SVG Graphics
+ PNG
+ Embeds etc

You can easiely and minimally create line charts, bar graphs, even Radar Chars with very little code. 



>Examples will be static images due to Githbubs supprt of SVG's is.....  sigh...
Just know SVG's are interactive. 

---
#### Let's Bgein - Install

Open Terminal
```
$  pip install Pygal
```
---

In an editor: Sublime, IDLE, etc type the following -
```
import pygal
```
For this example we'll create a bar chart and a stacked bar chart.  So, we simply need to create a varible and store ```pygal.Bar()``` inside.

```
b_chart = pygal.Bar()
```
You can easily use ```pygal.Line```, ```pygal.pie``` etc.

Next we need to start creating our chart.  I will be using Data that I scraped from a gaming tracker for the Game Destiny 2.  Eventually the graph will be live and I can see my stats change hopefully in real time.  Let's not get a head of our selves.

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

Technically we can be finished with out further customization.  We can render this quickly to a browser using:
```
import pygal

b_chart = pygal.Bar()
b_chart.add("Dijiphos", [0.93])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
b_chart.render_in_browser()
```
See the example below


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/af93de40-4e61-4329-93b3-835e127e77cc.gif)













