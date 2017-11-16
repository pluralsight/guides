
## Creating Charts with Python Pygal

You may be familar with Matplotlib, Seaborn, and Bokeh to name a few.  Sure, there are a few others the allow interactivty and soforth, but it's Pygal that allows you to create SVG's. Besides the scalability of an SVG, you can edit them in any editor and can print them with very high quality resolution.  My favorite part is that I can import it straight into Illustrator to enhance my aesthetics. Lastly, you can easiely and minimally create line charts, bar graphs, even Radar Charts with very little code as you'll see below. 

#### Check Out The Some Examples Below:

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


#### Let's Begin with an Install

Open Terminal
```
$  pip install Pygal
```
---
<b>Extra Help:</b>  Installing and Dependencies  - [Link](http://www.pygal.org/en/stable/installing.html)

---

#### Step 1:  Import

```
import pygal
```

#### Step 2: Create a Variable to Store Your Graph In
For our first example we'll create a bar chart.  We'll simply need to create a varible and store ```pygal.Bar()``` inside.

```
b_chart = pygal.Bar()
```
You can easily use ```pygal.Line```, ```pygal.pie``` as well.


#### Step 3: Let's Add Some Values
Next we need to start creating our chart.  I will be using Data that I scraped from a gaming tracker for the game <b>Destiny 2.</b>  Eventually the graph will be live and I'll be able to see my stats change (_hopefully_) in real time. 

---

<details>
<summary><b>```Click``` Game Play + Graph Change Real Time Example</b></summary>
<br>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/1ea09d8f-05e0-4814-9255-8ef430d34200.gif)
</details>

---

<i>Let's not get ahead of our selves.</i> <br>In short, everytime I play in a PvP or Crusible Match and eliminate an opponient, or, be eliminated myself, the KD (_Kill/Death Ratio_) will change.  I simply want to compare myself to my clan mates.



To start things off we'll need a chart title.
```
b_chart.title = "Destiny Kill/Death Ratio"
```
Now we can start adding in our data. I need 3 bars, one for each player.  To accomplish this, i'll need to use ```add``` followed by a title and some values.  
```
b_chart.add("Dijiphos", [0.94])
b_chart.add("Punisherdonk", [1.05])
b_chart.add("Musclemuffins20", [1.10])
```

Technically, we can finish and render without further customization.  To render quickly to a browser we'll use ```render_in_browser()``` as our output. 
```
b_chart.render_in_browser()
```
---
<details>
<summary><b>```Click``` To Reveal Our Example</b></summary>
<br>
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/ec4a6f0d-9cde-4785-8a76-2b819df70775.gif)
</details>

---


##### <b>Let's Customize</b>
Lets say we want some custom colors added to our graph<br>
![#E80080](https://placehold.it/15/e80080/000000?text=+) `#E80080`
![#404040](https://placehold.it/15/404040/000000?text=+) `#404040`
![#9bc850](https://placehold.it/15/9bc850/000000?text=+) `#9BC850`

This is easy to do, and can actually be achieved in multiple ways.
First, import ```style```.
```
from pygal.style import Style
```
You can change a number of objects by simply adding:
```
custom_style = Style(
```
_Notice I left the parentheses open_

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

Let's change the color of our bars.<br>_Don't forget to indent and close our final parentheses._
```
custom_style = Style(
  colors=('#E80080', '#404040', '#9BC850'))
```
Now all we need to do is pass ```style=custom_style``` in our ```pygal.Bar()```
```
custom_style = Style(
  colors=('#E80080', '#404040', '#9BC850'))

b_chart = pygal.Bar(style=custom_style)
```

---
<details>
<summary><b>```Click``` To Reveal Our Example</b></summary>
<br>


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/f522dcab-e4f3-4a1e-9e94-0e318634c7ee.gif)
</details>

----
#### It's a Wrap

I would check out [Pygal's](http://www.pygal.org/en/stable/installing.html) documentation to for further builds.  You should have all the tools needed to create great looking graphs with Python Pygal :)

<br>
Leave a comment or reach out and say hi on [Twitter](https://twitter.com/TroyKranendonk)
Read more at 



<br>

<br>
<br>



<br>

<br>