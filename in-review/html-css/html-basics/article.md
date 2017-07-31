## What is HTML?
---
**HTML is a markup language used for describing web documents (or web pages).**

* HTML stands for Hyper Text Markup Language.
* A markup language is a set of markup tags, which look like this: `<...> </...>` (Replace `...` with the type of markup tag that is being used)
* HTML documents are described by [HTML tags](#HTML Tags)
* Each HTML tag describes different document content


>A sample HTML document:

```html
<!DOCTYPE html>
<html>
<head>
<title>Page Title</title>
</head>
<body>

<h1>My First Heading</h1>
<p>My first paragraph.</p>

</body>
</html>
```
### Example Explained

Let's go through the lines of HTML code in the example above.

- The ```DOCTYPE``` declaration defines the document type to be HTML
- The text between ```<html> and </html>``` describes an HTML document
- The text between ```<head> and </head>``` provides information about the document
- The text between ```<title> and </title>``` provides a title for the document
- The text between ```<body> and </body>``` describes the visible page content
- The text between ```<h1> and </h1>``` describes a heading
- The text between ```<p> and </p>``` describes a paragraph

## HTML Tags
The most of HTML tags are __keywords__ (tag names) surrounded by __angle brackets__:

     <tagname>content</tagname>
<br>
* HTML tags usually come in pairs like ```<p> and </p>```
* The first tag in a pair is the __start tag__, the second tag is the **end tag**.
* The end tag is written like the start tag, but with a __slash__ before the tag name

There are also self-closing HTML tags, like the `<br />` tag, which inserts a single line break into a HTML document. For example, this HTML code

     Line 1 <br/> Line 2

will be shown in a browser as:

    Line 1
    Line 2
---

## Web Browsers
The purpose of a web browser (Chrome, IE, Firefox, Safari) is to read HTML documents and display them. You may have noticed that a browser does not display the HTML tags. Instead, it uses them to determine how to create, organize, and display the document:


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6c0aa61f-583a-495a-b182-bfa0c825f210.png)
---
## Write HTML Using Notepad
##### HTML can be edited by using professional HTML editors, such as the ones listed below:

* [Atom.io](https://atom.io/)
* [Visual Studio Code](https://code.visualstudio.com/b?utm_expid=101350005-21.ckupCbvGQMiML5eJsxWmxw.1&utm_referrer=https%3A%2F%2Fcode.visualstudio.com%2Fb)
* [Sublime](https://www.sublimetext.com/)

###### Follow the 4 steps below to create your first web page with Notepad.

#### **Step 1: Open Your Editor**

#### **Step 2: Write Some HTML**

Write or copy HTML into your editor.

```html
<!DOCTYPE html>
<html>
<body>

<h1>My First Heading</h1>

<p>My first paragraph.</p>

</body>
</html>
```
As seen in the image above, this HTML creates an HTML document with a heading display saying "First Heading" and a paragraph titled "First paragraph."

#### **Step 3: Save the HTML Page**

  Save the file on your computer.

  Select File > Save as in the Notepad menu.

  Name the file "index.html" or any other name ending with html or htm.

  UTF-8 is the preferred encoding for HTML files.

  ANSI encoding covers US and Western European characters only. 
  
  
![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/37c002d3-9acf-4d11-b1b5-c733799f748c.png)

#### **Step 4: View HTML Page in Your Browser.**

Open the saved HTML file in your favorite browser. The result will look like [this](https://raw.githubusercontent.com/pluralsight/guides/master/images/6c0aa61f-583a-495a-b182-bfa0c825f210.png).


> If you want to learn more,you can visit:
* [The w3 HTML Tutorial](http://www.w3schools.com/html/default.asp)
* [Codecademy](https://www.codecademy.com/courses/web-beginner-en-HZA3b/0/1)
* [The HTML page on Wikipedia](https://en.wikipedia.org/wiki/HTML)

Hopefully this tutorial helped you get started on creating an HTML document. As you can see, web design has very simple roots. However, learning more advanced design is key to building fresh, modern-looking web pages. Stay tuned for more tutorials on HTML basics.

Thank you for reading. Please leave your comments and feedback in the discussion section below!
