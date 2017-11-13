
# Creating Graphs w/ Python Pygal


Py Gal - works well with Django Flask etc. 
+ SVG Graphics
+ PNG
+ Embeds etc

You can easiely and minimally create line charts, bar graphs, even Radar Chars with very little code. 



>Examples will be static images due to Githbubs supprt of SVG's is.....  sigh...
Just know SVG's are interactive. 

---
#### Install

Open Term
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
Next we need to create an axis with a range.  
```
line_chart.x_labels = map(str, range(0, 1.25))
```
Don't forget to include ```str``` to convert the int into a string.

Now we can start adding some data. I need 3 bars for each player.
```
bar_chart.add("Dijiphos", [0.93])
bar_chart.add("Punisherdonk", [1.05])
bar_chart.add("Punisherdonk", [1.10])
```

Technically we can be finished with out further customization.







