**CSS isn’t always easy to deal with.** Depending on your skills and your experience, CSS coding can sometimes become a nightmare, particularly if you aren’t sure which selectors are actually being applied to document elements. An easy way to minimize the complexity of the code is as useful as not-so-well-known CSS attributes and properties you can use to create a semantically correct markup.

We’ve taken a close look at some of the most useful CSS tricks, tips, ideas, methods, techniques and coding solutions and listed them below. We also included some basic techniques you can probably use in every project you are developing, but which are hard to find once you need them.

And what has come out of it is an overview of **over 70 expert CSS ideas** which can improve your efficiency of CSS coding. You might be willing to check out the list of references and related articles in the end of this post.

We’d like to **express sincere gratitude to all designers** who shared their ideas, techniques, methods, knowledge and experience with their readers. Thank you, we, coders, designers, developers, information architects – you name it – really appreciate it.

You might be interested in reading our article [53 CSS-Techniques You Couldn’t Live Without](http://www.smashingmagazine.com/2007/01/19/53-css-techniques-you-couldnt-live-without/)<sup>[1](#1)</sup>, which should provide you with a basic toolbox for CSS-based techniques you might use in your next projects.

Update (29/05/2007): [Brazilian-Portuguese translation of the article](http://www.maujor.com/blog/2007/05/29/70-dicas-para-escrever-css/)<sup>[2](#2)</sup> is also available. Thanks to Maurício Samy Silva.

### 1.1\. Workflow: Getting Started

* **After you have a design, start with a blank page of content.** “Include your headers, your navigation, a sample of the content, and your footer. Then start adding your html markup. Then start adding your CSS. It works out much better.” [[CSSing](http://cssing.blogspot.com/2006/02/10-css-tips-for-new.html)<sup>[17](#17)</sup><sup>[3](#3)</sup>]
* **Use a master stylesheet.** “One of the most common mistakes I see beginners and intermediates fall victim to when it comes to CSS is not removing the default browser styling. This leads to inconsistencies in the appearance of your design across browsers, and ultimately leaves a lot of designers blaming the browser. It is a misplaced blame, of course. Before you do anything else when coding a website, you should reset the styling.” [[Master Stylesheet: The Most Useful CSS Technique](http://www.crucialwebhost.com/blog/master-stylesheet-the-most-useful-css-technique/)<sup>[4](#4)</sup>], [Ryan Parr]

```css
/* master.css */
@import url("reset.css");
@import url("global.css");  
@import url("flash.css");
@import url("structure.css");
```
```html
<style type="text/css" media="Screen">
   @import url("css/master.css");
</style>
```

* **Reset your CSS-styles first.** “You can often eliminate the need to specify a value for a property by taking advantage of that property’s default value. Some people like doing a [Global white space reset](http://leftjustified.net/journal/2004/10/19/global-ws-reset/)<sup>[5](#5)</sup> by zeroing both margin and padding for all elements at the top of their stylesheets. [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]
* **Keep a library of helpful CSS classes.** Useful for debugging, but should be avoided in the release version (separate markup and presentation). Since you can use multiple class names (i.e. `<p class="floatLeft alignLeft width75">...</p>`), make use of them debugging your markup. (_updated_) [[Richard K. Miller](http://www.richardkmiller.com/blog/archives/2006/08/css-best-practices)<sup>[7](#7)</sup>]

```css
.width100 { width: 100%; }
.width75 { width: 75%; }
.width50 { width: 50%; }
.floatLeft { float: left; }
.floatRight { float: right; }
.alignLeft { text-align: left; }
.alignRight { text-align: right; }
```

* Eric Meyer’s [Global Reset](http://meyerweb.com/eric/thoughts/2007/05/01/reset-reloaded/)<sup>[8](#8)</sup>, [Christian Montoya’s initial CSS file](http://www.christianmontoya.com/2007/02/01/css-techniques-i-use-all-the-time/)<sup>[9](#9)</sup>, [Mike Rundle’s initial CSS file](http://businesslogs.com/design_and_usability/my_5_css_tips.php)<sup>[10](#10)</sup>, [Ping Mag’s initial CSS file](http://www.pingmag.jp/2006/05/18/5-steps-to-css-heaven/)<sup>[11](#11)</sup>.

### 1.2\. Organize your CSS-code

* **Organize your CSS-styles, using master style sheets.** “Organizing your CSS helps with future maintainability of the site. Start with a master style sheet. Within this style sheet import your `reset.css`, `global.css`, `flash.css` (if needed) and `structure.css` and on occasion a typography style sheet. Here is an example of a “master” style sheet and how it is embedded in the document:”

```css
h2 { }
#snapshot_box h2 { padding: 0 0 6px 0; font: bold 14px/14px "Verdana", sans-serif; }
#main_side h2 { color: #444; font: bold 14px/14px "Verdana", sans-serif; }
.sidetagselection h2 { color: #fff; font: bold 14px/14px "Verdana", sans-serif; }
```

* **Organize your CSS-styles, using flags.** “Divide your stylesheet into specific sections: i.e. Global Styles – (body, paragraphs, lists, etc), Header, Page Structure, Headings, Text Styles, Navigation, Forms, Comments, Extras. [[5 Tips for Organizing Your CSS](http://www.erraticwisdom.com/2006/01/18/5-tips-for-organizing-your-css)<sup>[12](#12)</sup>]

```css
/* ------------------------*/ /* ---------->>> GLOBAL <<<-----------*/ /* ------------------------*/
```

* **Organize your CSS-styles, making a table of contents.** At the top of your CSS document, write out a table of contents. For example, you could outline the different areas that your CSS document is styling (header, main, footer etc). Then, use a large, obvious section break to separate the areas. [[5 Steps to CSS Heaven](http://www.pingmag.jp/2006/05/18/5-steps-to-css-heaven/)<sup>[89](#89)</sup><sup>[24](#24)</sup><sup>[13](#13)</sup>]
* **Organize your CSS-styles, ordering properties alphabetically.** “I don’t know where I got the idea, but I have been alphabetizing my CSS properties for months now, and believe it or not, it makes specific properties much easier to find.” [[Christian Montoya](http://www.christianmontoya.com/2007/02/01/css-techniques-i-use-all-the-time/)<sup>[38](#38)</sup><sup>[14](#14)</sup>]

```css
body {
   background: #fdfdfd;
   color: #333;
   font-size: 1em;
   line-height: 1.4;
   margin: 0;
   padding: 0;
}
```

* **Separate code into blocks.**. “This might be common sense to some of you but sometimes I look at CSS and it’s not broken down into “sections.” It’s easy to do an it makes working with code weeks, months, or years later much easier. You’ll have an easier time finding classes and elements that you need to change. Examples: `/* Structure */`, `/* Typography */` etc.” [[CSS Tips and Tricks](http://www.blogherald.com/2006/09/08/css-tips-and-tricks/)<sup>[15](#15)</sup>]
* **Hook, line, and sinker.** Once you have your CSS and sections in place start considering where your selector “hooks” will live by using structural hooks in your mark up. This is your saving grace for future editing and maintenance of the site. This will also give you strength in your document.” [Ryan Parr]
* **Break your style sheet in separate blocks.** “I break down my style sheet into three separate blocks. The first is straight element declarations. Change the body, some links styles, some header styles, reset margins and padding on forms, and so on. […] After element declarations, I have my class declarations; things like classes for an error message or a callout would go here. [..] I start by declaring my main containers and then any styles for elements within those containers are indented. At a quick glance, I can see how my page is broken down and makes it easier to know where to look for things. I’ll also declare containers even if they don’t have any rules.” [[Jonathan Snook](http://snook.ca/archives/html_and_css/top_css_tips/)<sup>[68](#68)</sup><sup>[34](#34)</sup><sup>[18](#18)</sup><sup>[16](#16)</sup>]

### 1.3\. Workflow: Handling IDs, Classes, Selectors, Properties

* **Keep containers to a minimum.** “Save your document from structural bloat. New developers will use many div’s similar to table cells to achieve layout. Take advantage of the many structural elements to achieve layout. Do not add more div’s. Consider all options before adding additional wrappers (div’s) to achieve an effect when using a little nifty CSS can get you that same desired effect.” [Ryan Parr]
* **Keep properties to a minimum.** “Work smarter, not harder with CSS. Under this rule, there are a number of subrules: if there isn’t a point to adding a CSS property, don’t add it; if you’re not sure why you’re adding a CSS property, don’t add; and if you feel like you’ve added the same property in lots of places, figure out how to add it in only one place.” [[CSSing](http://cssing.blogspot.com/2006/02/10-css-tips-for-new.html)<sup>[17](#17)</sup><sup>[3](#3)</sup>]
* **Keep selectors to a minimum.** “Avoid unnecessary selectors. Using less selectors will mean less selectors will be needed to override any particular style — that means it’s easier to troubleshoot.” [[Jonathan Snook](http://snook.ca/archives/html_and_css/top_css_tips/)<sup>[68](#68)</sup><sup>[34](#34)</sup><sup>[18](#18)</sup><sup>[16](#16)</sup>]
* **Keep CSS hacks to a minimum.** “Don’t use hacks unless its a known and documented bug. This is an important point as I too often see hacks employed to fix things that aren’t really broken in the first place. If you find that you are looking for a hack to fix a certain issue in your design then first do some research (Google is your friend here) and try to identify the issue you are having problems with. [[10 Quick Tips for an easier CSS life](http://www.search-this.com/2007/03/26/10-quick-tips-for-an-easier-css-life/)<sup>[19](#19)</sup>]
* **Use CSS Constants for faster development.** “The concept of constants – fixed values that can be used through your code [is useful]. [..] One way to get round the lack of constants in CSS is to create some definitions at the top of your CSS file in comments, to define ‘constants’. A common use for this is to create a ‘color glossary’. This means that you have a quick reference to the colors used in the site to avoid using alternates by mistake and, if you need to change the colors, you have a quick list to go down and do a search and replace.” [[Rachel Andrew](http://24ways.org/2006/faster-development-with-css-constants)<sup>[20](#20)</sup>]

```css
/*
   # Dark grey (text): #333333
   # Dark Blue (headings, links) #000066
   # Mid Blue (header) #333399
   # Light blue (top navigation) #CCCCFF
   # Mid grey: #666666 #
*/
```

* **Use a common naming system.** Having a naming system for id’s and classes saves you a lot of time when looking for bugs, or updating your document. Especially in large CSS documents, things can get a big confusing quickly if your names are all different. I recommend using a `parent_child` pattern. [[10 CSS Tips](http://christopher-scott.org/blog/10-css-tips-you-might-not-have-known-about)<sup>[67](#67)</sup><sup>[43](#43)</sup><sup>[21](#21)</sup>]
* **Name your classes and IDs properly, according to their semantics.** “We want to avoid names that imply presentational aspects. Otherwise, if we name something right-col, it’s entirely possible that the CSS would change and our “right-col” would end up actually being displayed on the left side of our page. That could lead to some confusion in the future, so it’s best that we avoid these types of presentational naming schemes. [[Garrett Dimon](http://www.digital-web.com/articles/markup_as_craft/)<sup>[22](#22)</sup>]
* **Group selectors with common CSS declarations.** “Group selectors. When several element types, classes, or id:s share some properties, you can group the selectors to avoid specifying the same properties several times. This will save space – potentially lots of it.” [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]
* **Isolate single properties that you are likely to reuse a lot.** “If you find yourself using a single property a lot, isolate it to save yourself repeating it over and over again and also enabling you to change the display of all parts of the site that use it.” [[5 Steps to CSS Heaven](http://www.pingmag.jp/2006/05/18/5-steps-to-css-heaven/)<sup>[89](#89)</sup><sup>[24](#24)</sup><sup>[13](#13)</sup>]
* **Move ids and class naming as far up the document tree as you can.** Leverage [contextual selectors](http://www.456bereastreet.com/archive/200509/css_21_selectors_part_1/)<sup>[25](#25)</sup> as much as possible. Don’t be afraid to be verbose in your selectors. Longer selectors can make css documents easier to read while also cutting down the chances of developing class- or [divitis](http://juicystudio.com/article/div-mania.php)<sup>[26](#26)</sup>. [[Chric Casciano](http://placenamehere.com/article/156/TenSimpleCSSTips)<sup>[76](#76)</sup><sup>[41](#41)</sup><sup>[27](#27)</sup>]
* **Learn to exploit the cascading nature of CSS.** “Say you have two similar boxes on your website with only minor differences – you could write out CSS to style each box, or you could write CSS to style both at the same time, then add extra properties below to make one look different.” [[5 Steps to CSS heaven](http://www.pingmag.jp/2006/05/18/5-steps-to-css-heaven/)<sup>[28](#28)</sup>]
* **Use Your Utility Tags: `<small>`, `<em>` and `<strong>`.** “Many times you’ll have a section in your design that calls for various typographical weights/looks all on the same line, or very close to each other. drop in random divs and classes because I feel they’re not semantic and defeat the purpose of your nice XHTML everywhere else.” Instead, use semantic tags. [[Mike Rundle’s 5 CSS Tips](http://businesslogs.com/design_and_usability/my_5_css_tips.php)<sup>[29](#29)</sup>]

### 1.4\. Workflow: Use shorthand notation

* **Shorten hexadecimal colour notation.** “In CSS, when you use hexadecimal colour notation and a colour is made up of three pairs of hexadecimal digits, you can write it in a more efficient way by omitting every second digit: `#000 is the same as #000000, #369 is the same as #336699` [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]
* **Define pseudo classes for links in the LoVe/HAte-order**: Link, Visited, Hover, Active. “To ensure that you see your various link styles, you’re best off putting your styles in the order “link-visited-hover-active”, or “LVHA” for short. If you’re concerned about focus styles, they may go at the end– but wait until you’ve read this explanation before you decide.” [[Eric Meyer](http://meyerweb.com/eric/css/link-specificity.html)<sup>[31](#31)</sup>]

```css
a:link { color: blue; }
a:visited { color: purple; }
a:hover { color: purple; }
a:active { color: red; }
```

* **Define element’s margin, padding or border in TRouBLed-order**: Top, Right, Bottom, Left. “When using shorthand to specify an element’s margin, padding or border, do it clockwise from the top: Top, Right, Bottom, Left.” [[Roger Johansson](http://www.456bereastreet.com/lab/developing_with_web_standards/css/)<sup>[44](#44)</sup><sup>[32](#32)</sup>]
* **You can use [shorthand properties](http://www.456bereastreet.com/archive/200502/efficient_css_with_shorthand_properties/)<sup>[33](#33)</sup>.** “Using shorthand for `margin`, `padding` and `border` properties can save a lot of space.

```css
margin: top right bottom left;
margin: 1em 0 2em 0.5em;
(margin-top: 1em; margin-right: 0; margin-bottom: 2em; margin-left: 0.5em;)
```

```css
border: width style color;
border: 1px solid #000;
```

```css
background: color image repeat attachment position;
background: #f00 url(background.gif) no-repeat fixed 0 0;
```

```css
font: font-style (italic/normal) font-variant (small-caps) font-weight font-size/line-height font-family;
font: italic small-caps bold 1em/140% "Lucida Grande",sans-serif;
```

### 1.5\. Workflow: Setting Up Typography

* **To work with EMs like with pxs, set **font-size** on the body-tag with 62.5%**. Default-value of the `font-size` is 16px; applying the rule, you’ll get one Em standing for roughly ten pixels (16 x 62.5% = 10). “I tend to put a font-size on the body tag with value: 62.5%. This allows you to use EMs to specify sizes while thinking in PX terms, e.g. 1.3em is approximately 1.3px. ” [[Jonathan Snook](http://snook.ca/archives/html_and_css/top_css_tips/)<sup>[68](#68)</sup><sup>[34](#34)</sup><sup>[18](#18)</sup><sup>[16](#16)</sup>]
* **Use universal character set for encoding**. “[..] The answer is to use a single universal character set that’s able to cover most eventualities. Luckily one exists: UTF-8, which is based on Unicode. Unicode is an industry standard that’s designed to enable text and symbols from all languages to be consistently represented and manipulated by computers. UTF- 8 should be included in your web page’s head like this. [[20 pro tips](http://www.netmag.co.uk/zine/design-tutorials/20-pro-tips)<sup>[77](#77)</sup><sup>[71](#71)</sup><sup>[36](#36)</sup><sup>[35](#35)</sup>]

```html
<meta http-equiv="content-type" content="text/ html;charset=utf-8" />
```

* **You can change capitalisation using CSS.** If you need something written in capitals, such as a headline, rather than rewriting the copy, let CSS do the donkey work. The following code will transform all text with an h1 attribute into all capitals, regardless of format”. [[20 pro tips](http://www.netmag.co.uk/zine/design-tutorials/20-pro-tips)<sup>[77](#77)</sup><sup>[71](#71)</sup><sup>[36](#36)</sup><sup>[35](#35)</sup>]

```css
h1 { text-transform: uppercase; }
```

* **You can display text in small-caps automatically.** The `font-variant` property is used to display text in a small-caps font, which means that all the lower case letters are converted to uppercase letters, but all the letters in the small-caps font have a smaller font-size compared to the rest of the text.

```css
h1 { font-variant: small-caps; }
```

* **Cover all the bases – define generic font-families.** “When we declare a specific font to be used within our design, we are doing so in the hope that the user will have that font installed on their system. If they don’t have the font on their system, then they won’t see it, simple as that. What we need to do is reference fonts that the user will likely have on their machine, such as the ones in the font-family property below. It is important that we finish the list with a generic font type. [[Getting into good coding habits](http://www.communitymx.com/content/article.cfm?cid=FAF76&print=true)<sup>[37](#37)</sup>]

```css
p { font-family: Arial, Verdana, Helvetica, sans-serif; }
```

* **Use 1.4em – 1.6em for `line-height`.** “`line-height:1.4`” for readable lines, reasonable line-lengths that avoid lines much longer than 10 words, and colors that provide contrast without being _too_ far apart. For example, pure black on pure white is often too strong for bright CRT displays, so I try to go with an off-white (`#fafafa` is a good one) and a dark gray (`#333333`, another good one).” [[Christian Montoya](http://www.christianmontoya.com/2007/02/01/css-techniques-i-use-all-the-time/)<sup>[38](#38)</sup><sup>[14](#14)</sup>]
* **Set 100.01% for the `html`-element.** This odd 100.01% value for the font size compensates for several browser bugs. First, setting a default body font size in percent (instead of em) eliminates an IE/Win problem with growing or shrinking fonts out of proportion if they are later set in ems in other elements. Additionally, some versions of Opera will draw a default font-size of 100% too small compared to other browsers. Safari, on the other hand, has a problem with a font-size of 101%. The current “best” suggestion is to use the 100.01% value for this property.” [[CSS: Getting into good habits](http://www.communitymx.com/content/article.cfm?cid=FAF76&print=true)<sup>[39](#39)</sup>]

### 1.6\. Workflow: Debugging

* **Add borders to identify containers.** “Use plenty of test styles like extra borders or background colors when building your documents or debugging layout issues. `div { border:1px red dashed; }` works like a charm. There are also [bookmarklets that apply borders](http://www.squarefree.com/bookmarklets/webdevel.html)<sup>[40](#40)</sup> and do other things for you.” You can also use `* { border: 1px solid #ff0000; }`. [[Chric Casciano](http://placenamehere.com/article/156/TenSimpleCSSTips)<sup>[76](#76)</sup><sup>[41](#41)</sup><sup>[27](#27)</sup>]. Adding a border to specific elements can help identify overlap and extra white space that might not otherwise be obvious. [[CSS Crib Sheet](http://www.mezzoblue.com/css/cribsheet/)<sup>[69](#69)</sup><sup>[42](#42)</sup>]

```css
* { border: 1px solid #f00; }
```

* **Check for closed elements first when debugging.** “If you ever get frustrated because it seemed like you changed one minor thing, only to have your beautiful holy-grail layout break, it might be because of an unclosed element. [[10 CSS Tips](http://christopher-scott.org/blog/10-css-tips-you-might-not-have-known-about)<sup>[67](#67)</sup><sup>[43](#43)</sup><sup>[21](#21)</sup>]

### 2.1\. Technical Tips: IDs, Classes

* **1 ID per page, many classes per page.** “Check your IDs: Only one element in a document can have a certain value for the id attribute, while any number of elements can share the same class name. [..] Class and id names can only consist of the characters [A-Za-z0-9] and hyphen (-), and they cannot start with a hyphen or a digit (see CSS2 syntax and basic data types).” [[Roger Johansson](http://www.456bereastreet.com/lab/developing_with_web_standards/css/)<sup>[44](#44)</sup><sup>[32](#32)</sup>]
* **Element names in selectors are case sensitive.** “Remember case sensitivity. When CSS is used with XHTML, element names in selectors are case sensitive. To avoid getting caught by this I recommend always using lowercase for element names in CSS selectors. Values of the class and id attributes are case sensitive in both HTML and XHTML, so avoid mixed case for class and id names.” [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]
* CSS classes and IDs must be valid. “I.e. [beginning with a letter](http://www.w3.org/TR/html401/types.html#type-id)<sup>[46](#46)</sup>, not a number or an underscore. IDs must be unique. Their names should be [generic](http://www.w3.org/QA/Tips/goodclassnames "W3C Quality Assurance Tip: Use ")<sup>[47](#47)</sup>, describe functionality rather than appearance.” [[CSS Best Practices](http://learningtheworld.eu/2006/best-practices/#css)<sup>[48](#48)</sup>]
* **You can assign multiple class names to a given element.** “You can assign multiple class names to an element. This allows you to write several rules that define different properties, and only apply them as needed.” [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]

### 2.2\. Technical Tips: Use the power of selectors

Roger Johansson has written an **extremely** useful series of articles about [CSS 2.1 Selectors](http://www.456bereastreet.com/archive/200509/css_21_selectors_part_1/)<sup>[50](#50)</sup>. These articles are **highly recommended** to read – some useful aspects can be found in the list below. Note that selectors ‘>’ and ‘+’ aren’t supported in IE6 and earlier versions of Internet Explorer (_updated_).

* **You can use child selectors.** “A child selector targets an immediate child of a certain element. A child selector consists of two or more selectors separated by a greater than sign, “>”. The parent goes to the left of the “>”, and whitespace is allowed around the combinator. This rule will affect all strong elements that are children of a div element. [[Roger Johansson](http://www.456bereastreet.com/archive/200510/css_21_selectors_part_2/)<sup>[52](#52)</sup><sup>[51](#51)</sup>]

```css
div > strong { color:#f00; }
```

* **You can use adjacent sibling selectors.** An adjacent sibling selector is made up of two simple selectors separated by a plus sign, “+”. Whitespace is allowed around the adjacent sibling combinator. The selector matches an element which is the next sibling to the first element. The elements must have the same parent and the first element must immediately precede the second element. [[Roger Johansson](http://www.456bereastreet.com/archive/200510/css_21_selectors_part_2/)<sup>[52](#52)</sup><sup>[51](#51)</sup>]

```css
p + p { color:#f00; }
```

* **You can use attribute selectors.** Attribute selectors match elements based on the presence or value of attributes. There are four ways for an attribute selector to match:

```css
[att] Matches elements that have an att attribute, regardless of its value.
[att=val] Matches elements that have an att attribute with a value of exactly “val”.
[att~=val] Matches elements whose att attribute value is a space-separated list that contains “val”. In this case “val” cannot contain spaces.
[att|=val] Matches elements whose att attribute value is a hyphen-separated list that begins with “val”. The main use for this is to match language subcodes specified by the lang attribute (xml:lang in XHTML), e.g. “en”, “en-us”, “en-gb”, etc.
```

* The selector in the following rule matches all `p` elements that have a `title` attribute, regardless of which value it has:

```css
p[title] { color:#f00; }
```

* The selector matches all div elements that have a class attribute with the value error:

```css
div[class=error] { color:#f00; }
```

* Multiple attribute selectors can be used in the same selector. This makes it possible to match against several different attributes for the same element. The following rule would apply to all blockquote elements that have a class attribute whose value is exactly “quote”, and a cite attribute (regardless of its value):

```css
blockquote[class=quote][cite] { color:#f00; }
```

* **You should use descendant selectors.** “Descendant selectors can help you eliminate many class attributes from your markup and make your CSS selectors much more efficient. ” [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]

### 2.3\. Technical Tips: Styling Links

* **Be careful when styling links if you’re using anchors.** “If you use a classic anchor in your code (`<a name="anchor">`) you’ll notice it picks up `:hover` and `:active` pseudo-classes. To avoid this, you’ll need to either use `id` for anchors instead, or style with a [slightly more arcane](http://dbaron.org/css/1999/09/links "Notes on suggesting link styles")<sup>[54](#54)</sup> syntax: `:link:hover, :link:active`” [[Dave Shea](http://www.mezzoblue.com/css/cribsheet/)<sup>[55](#55)</sup>]
* **Define relationships for links.** “The rel attribute is supposed to indicate a semantic link relationship from one resource to another.

```css
a[rel~="nofollow"]::after { content: "2620"; color: #933; font-size: x-small; }
a[rel~="tag"]::after { content: url(http://www.technorati.com/favicon.ico); }
```

* “These make use of the attribute selector for space separated lists of values. Any a element with a relationship containing those values will be matched. Links with the nofollow relationship will be followed by a dark red skull and crossbones (?) and those with the tag relationship will be followed by the Technocrati icon.” [[Handy CSS](http://lachy.id.au/log/2005/04/handy-css)<sup>[90](#90)</sup><sup>[57](#57)</sup><sup>[56](#56)</sup>]
* You can mark external links automatically. Many people make use of the non-standard `rel="external"` relationship to indicate a link to an external site. However, adding that to each and every link is time consuming and and unnecessary. This style rule will place an north east arrow after any link on your site to an external site. [[Handy CSS](http://lachy.id.au/log/2005/04/handy-css)<sup>[90](#90)</sup><sup>[57](#57)</sup><sup>[56](#56)</sup>]

```css
a[href^="http://"]:not([href*="smashingmagazine.com"])::after { content: "2197"; }
```

* **You can remove dotted links with `outline: none;`**. To [remove dotted links](http://sonspring.com/journal/removing-dotted-links)<sup>[58](#58)</sup> use `outline: none;`:

```css
a:focus { outline: none; }
```

### 2.4\. Technical Tips: CSS-Techniques

* **You can specify body tag ID.** “In most cases placing an ID in the body tag will allow you manipulate CSS presentational items and markup elements by page by page basis. Not only will you be able to organize your sections you will be able to create multiple CSS presentations without changing your markup from template to template or page to page.” [Ryan Parr, [Invasion of Body Switchers](http://alistapart.com/articles/bodyswitchers)<sup>[59](#59)</sup>]
* **You can create columns with equal heights with CSS.** [Equal Height Technique](http://www.positioniseverything.net/articles/onetruelayout/equalheight)<sup>[60](#60)</sup>: a method to make all columns appear to be the same height. But without the need for faux column style background images. [Faux Columns](http://www.alistapart.com/articles/fauxcolumns/)<sup>[61](#61)</sup>: with background images.
* **You can align vertically with CSS.** “Say you have a navigation menu item whose height is assigned 2em. Solution: specify the line height to be the same as the height of the box itself in the CSS. In this instance, the box is 2em high, so we would insert line-height: 2em into the CSS rule and the text now floats in the middle of the box!” [[Evolt.org](http://evolt.org/article/rdf/17/60369/)<sup>[62](#62)</sup>]
* **You can use pseudo-elements and classes to generate content dynamically.** [Pseudo-classes and pseudo-elements](http://www.456bereastreet.com/archive/200510/css_21_selectors_part_3/)<sup>[63](#63)</sup>. Pseudo-classes and pseudo-elements can be used to format elements based on information that is not available in the document tree. For example, there is no element that refers to the first line of a paragraph or the first letter of an element’s text content. You can use :first-child, :hover, :active, :focus, :first-line, :first-letter, :before, :after and more.
* **You can set `<hr>` to separate posts beautifully.** “Restyling the horizontal rule (<hr>) with an image can be a beautiful addition to a web page. [[CSS: Best Practices](http://www.richardkmiller.com/blog/archives/2006/08/css-best-practices)<sup>[64](#64)</sup>]
* **You can use the same navigation (X)HTML-code on every page.** “Most websites highlight the navigation item of the user’s location in the website. But it can be a pain as you’ll need to tweak the HTML code behind the navigation for each and every page. So can we have the best of both worlds?” [[Ten More CSS Tricks you may not know](http://www.sitepoint.com/article/top-ten-css-tricks)<sup>[65](#65)</sup>]

```html
<ul>
   <li><a href="#" class="home">Home</a></li>
   <li><a href="#" class="about">About us</a></li>  
   <li><a href="#" class="contact">Contact us</a></li>
</ul>
```

* Insert an `id` into the `<body>` tag. The id should be representative of where users are in the site and should change when users move to a different site section.

```css
#home .home, #about .about, #contact .contact { commands for highlighted navigation go here }
```

* **You can use `margin: 0 auto;` to horizontally centre the layout.** “To horizontally centre an element with CSS, you need to specify the element’s width and horizontal margins.” [Roger Johansson]

```html
 <div id="wrap"> <!-- Your layout goes here --> </div>
 ```

```css
#wrap { width:760px; /* Change this to the width of your layout */ margin:0 auto; }
```

* **You can add CSS-styling to RSS-feeds.** “You can do a lot more with an XSL stylesheet (turn links into clickable links, etc), but CSS can make your feed look much less scary for the non-technical crowd. [Pete Freitag]

```html
<?xml version="1.0" ?> <?xml-stylesheet type="text/css" href="http://you.com/rss.css" ?> ...
```

* **You can hide CSS from older browsers.** “A common way of hiding CSS files from old browsers is to use the `@import` trick. [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/)<sup>[66](#66)</sup><sup>[53](#53)</sup><sup>[49](#49)</sup><sup>[45](#45)</sup><sup>[30](#30)</sup><sup>[23](#23)</sup><sup>[6](#6)</sup>]

```css
@import "main.css";
```

* **Always declare margin and padding in block-level elements.** [[10 CSS Tips](http://christopher-scott.org/blog/10-css-tips-you-might-not-have-known-about)<sup>[67](#67)</sup><sup>[43](#43)</sup><sup>[21](#21)</sup>]
* **Set a width OR margin and padding.** “My rule of thumb is, if I set a width, I don’t set margin or padding. Likewise, if I’m setting a margin or padding, I don’t set a width. Dealing with the box model can be such a pain, especially if you’re dealing with percentages. Therefore, I set the width on the containers and then set margin and padding on the elements within them. Everything usually turns out swimmingly.” [[Jonathan Snook](http://snook.ca/archives/html_and_css/top_css_tips/)<sup>[68](#68)</sup><sup>[34](#34)</sup><sup>[18](#18)</sup><sup>[16](#16)</sup>]
* **Avoid applying padding/borders and a fixed width to an element.** “IE5 gets the box model wrong, which really makes a mess of things. There are ways around this, but it’s best to side-step the issue by applying the padding to the parent element instead of the child that gets a fixed-width. [[CSS Crib Sheet](http://www.mezzoblue.com/css/cribsheet/)<sup>[69](#69)</sup><sup>[42](#42)</sup>]
* **Provide print styles.** “You can add a print stylesheet in exactly the same way that you would add a regular stylesheet to your page:


```html
<link rel="stylesheet" type="text/css" href="print.css" media="print">
```
Or
```html
<style type=”text/css” media=”print”> @import url(print.css); </style>
```

* This ensures that the CSS will only apply to printed output and not affect how the page looks on screen. With your new printed stylesheet you can ensure you have solid black text on a white background and remove extraneous features to maximise readability. [More about CSS-based print-Layouts](http://www.smashingmagazine.com/2007/02/21/printing-the-web-solutions-and-techniques/)<sup>[70](#70)</sup>. [[20 pro tips](http://www.netmag.co.uk/zine/design-tutorials/20-pro-tips)<sup>[77](#77)</sup><sup>[71](#71)</sup><sup>[36](#36)</sup><sup>[35](#35)</sup>]

### 2.5\. Technical Tips: IE Tweaks

* **You can force IE to apply transparence to PNGs.** “In theory, PNG files do support varied levels of transparency; however, an Internet Explorer 6 bug prevents this from working cross-browser.” [[CSS Tips, Outer-Court.com](http://blog.outer-court.com/archive/2007-03-30-n51.html)<sup>[72](#72)</sup>]

```css
#regular_logo { background: url('test.png'); width:150px; height:55px; }
* html #regular_logo {
   background:none;
   float:left;
   width:150px;
   filter:progid:DXImageTransform.Microsoft.AlphaImageLoader(src='test.png', sizingMethod='scale');
}
```

* **You can define `min-width` and `max-width` in IE.** You can use Microsoft’s dynamic expressions to do that. [[Ten More CSS Trick you may not know](http://www.sitepoint.com/article/top-ten-css-tricks)<sup>[73](#73)</sup>]

```css
#container {
   min-width: 600px;
   max-width: 1200px;
   width:expression(document.body.clientWidth < 600? "600px" : document.body.clientWidth > 1200? "1200px" : "auto");
}
```

* **You can use Conditional Comments for IE.** “The safest way of taking care of IE/Win is to use conditional comments. It feels more future-proof than CSS hacks – is to use Microsoft’s proprietary conditional comments. You can use this to give IE/Win a separate stylesheet that contains all the rules that are needed to make it behave properly. ” [[Roger Johansson](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_2/)<sup>[74](#74)</sup>]

```html
<!--[if IE]> <link rel="stylesheet" type="text/css" href="ie.css" /> <![endif]-->
```

### Workflow: Get Inspired

* **Play, experiment with CSS.** “Play. Play with background images. Play with floats.” [[Play with positive and negative margins](http://chunkysoup.net/article/12/AbusingMargins)<sup>[75](#75)</sup>. Play with inheritance and cascading rules. Play. [[Chric Casciano](http://placenamehere.com/article/156/TenSimpleCSSTips)<sup>[76](#76)</sup><sup>[41](#41)</sup><sup>[27](#27)</sup>]
* **Learn from others.** Learn from great sites built by others. Any site’s HTML is easily accessible by viewing a page’s source code. See how others have done things and apply their methods to your own work. [[20 pro tips](http://www.netmag.co.uk/zine/design-tutorials/20-pro-tips)<sup>[77](#77)</sup><sup>[71](#71)</sup><sup>[36](#36)</sup><sup>[35](#35)</sup>]

### Sources and Related Posts

* [CSS Tips and Tricks](http://www.456bereastreet.com/archive/200503/css_tips_and_tricks_part_1/) by _Roger Johansson_
* [(The Only) Ten Things To Know About CSS](http://blog.jm3.net/2007/03/16/the-only-ten-things-to-know-about-css/) by _John Manoogian_
* [CSS Crib Sheet](http://www.mezzoblue.com/archives/2003/11/19/css_crib_she/) by _Dave Shea_
* [My Top Ten CSS Tricks [CSS Tutorials]](http://www.sitepoint.com/article/top-ten-css-tricks) by _Trenton Moss_
* [CSS Tips](http://blog.outer-court.com/archive/2007-03-30-n51.html) by _Philipp Lenssen_
* [Top CSS Tips](http://www.snook.ca/archives/html_and_css/top_css_tips/) by _Jonathan Snook_
* [Ten CSS tricks — corrected and improved](http://tantek.com/log/2004/09.html#d07t1434) by _Tantek Çelik_
* [Ten More CSS Trick you may now know](http://www.webcredible.co.uk/user-friendly-resources/css/more-css-tricks.shtml) by _Trenton Moss_
* [CSS techniques I use all the time](http://www.christianmontoya.com/2007/02/01/css-techniques-i-use-all-the-time/) by _Christian Montoya_
* [CSS Tip Flags](http://www.stopdesign.com/log/2005/05/03/css-tip-flags.html) by _Douglas Bowman_
* [My 5 CSS Tips](http://businesslogs.com/design_and_usability/my_5_css_tips.php) by _Mike Rundle_
* [5 Steps to CSS Heaven](http://www.pingmag.jp/2006/05/18/5-steps-to-css-heaven/) by _Ping Mag_
* [Handy CSS](http://lachy.id.au/log/2005/04/handy-css) by _Lachlan Hunt_
* [Erratic Wisdom: 5 Tips for Organizing Your CSS](http://erraticwisdom.com/2006/01/18/5-tips-for-organizing-your-css) by _Thame Fadial_
* [15 CSS Properties You Probably Never Use (but perhaps should)](http://www.seomoz.org/blog/css-properties-you-probably-never-use) by _SeoMoz_
* [10 CSS Tips You Might Not Have Known About](http://christopher-scott.org/blog/10-css-tips-you-might-not-have-known-about) by _Christopher Scott_
* [A List Apart: Articles: 12 Lessons for Those Afraid of CSS and Standards](http://www.alistapart.com/articles/12lessonsCSSandstandards) by _Ben Henick_
* [Tips for a better design review process](http://www.dkeithrobinson.com/entry/tips_for_a_better_design_review_process/) by _D. Keith Robinson_
* [20 pro tips – .net magazine](http://www.netmag.co.uk/zine/design-tutorials/20-pro-tips) by _Jason Arber_
* [CSS Best Practices](http://www.richardkmiller.com/blog/archives/2006/08/css-best-practices) by _Richard K Miller_
* [10 Quick Tips for an Easier CSS Life](http://www.search-this.com/2007/03/26/10-quick-tips-for-an-easier-css-life/) by _Paul Ob_
* 10 CSS Tips from a Professional CSS Front-End Architect by _72 DPI in the shade team blog_
* [Web Design References: Cascading Style Sheets](http://www.d.umn.edu/itss/support/Training/Online/webdesign/css.html#tips) by _Laura Carlson_
* [Getting Into Good Coding Habits](http://www.communitymx.com/content/article.cfm?cid=FAF76&print=true) by _Adrian Senior_

This article is by **Vitaly Friedman** from [smashingmagazine.com](http://smashingmagazine.com).

