We are living in a world when (lots of) people use big frameworks in their apps. In general that's easier. In some cases that's cool. In terms of performance that could be painful.

Let's talk about CSS grid systems. There are a lot of libraries/frameworks providing grid functionalities. Twitter Bootstrap is one of these [big] frameworks. Loading it would make sense if you really needed it. Just for using it as grid system you most probably shouldn't include Bootstrap (lot of developers do that, though). This just adds needless pain and possible confusion for you or others later down the road.

I really wanted to have a **minimal** and **usable** solution for **modern browsers**. I did a quick research and found some information, but all the libraries I found were providing too many features, making the things slower and bigger.

---

Then I quickly hacked my solution: [**Gridly**](https://github.com/IonicaBizau/gridly).

[![gridly](http://i.imgur.com/kPrOESX.png)](http://ionicabizau.github.io/gridly/example/)

So, thinking from the highest level, I thought the nice HTML syntax would be:

```html
<div class="row">
   <div class="col">Column 1</div>
   <div class="col">Column 2</div>
   <div class="col">Column 3</div>
   ...
   <div class="col">Column N</div>
</div>
```

Basically, just having rows (`.row`) and columns (`.col`). I was thinking it's enough if these columns have the same width (that's a common use case, at least in my projects: having rows containing equal-width columns).

Of course, the next thing was to write some CSS snippets. I chose [CSS flexbox](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Flexible_Box_Layout/Using_CSS_flexible_boxes). 

That's being said, I started with one file: `gridly.css` containing:

```css
.row { display: flex; }
.col { flex: 1; }
```

Then, to handle the smaller devices (to create full-width columns), I added:

```css
@media (max-width: 48em) {
    .row { flex-direction: column; }
    .col { flex: 0 0 auto; }
}
```

[![gridly](http://i.imgur.com/m4pwrnO.png)](http://ionicabizau.github.io/gridly/example/)


Then, I thought it could be convenient to add percent values (e.g. `third`–33%, `quarter`–25% etc), so the columns could have different widths inside of the same row. I did this keeping the aspect of modularization in mind. I created  `gridly-col-widths.css` and put the following things there:

```css
@media (min-width: 48em) {
  .col-tenth { flex: 0 0 10%; }
  .col-fifth { flex: 0 0 20%; }
  .col-quarter { flex: 0 0 25%; }
  .col-third { flex: 0 0 33.3333334%; }
  .col-half { flex: 0 0 50%; }
}
```

And that was it. I wrote some nice docs, published it on GitHub, and there we go: [**Gridly**](https://github.com/IonicaBizau/gridly)––the core part (responsive equal-width columns) weighting not more than 105B (minified and gzipped).

Some of you may think this code can be just ~~copy-pasted~~ copy-wasted in random websites CSS. Others may say this would be a nice [Gist](https://gist.github.com/). I would say that would be a nice tweet.

But what makes Gridly awesome is the size and the functionality it *still* offers––being one of the tiniest CSS grid libraries (let's not call it framework)––and still compatible with the modern browsers (boy, don't call IE a *browser*, even this would work on IE10+, tho).

Check out [**Gridly on GitHub**](https://github.com/IonicaBizau/gridly)!
