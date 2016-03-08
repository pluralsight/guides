
Hi every one! today I'm gonna to show you how to parse HTML files through the amazing DOMDocument.
To tests will be done with the following HTML script.

```
<html>
    <body>
        <div id="header">
            This is a text for header div.
            <form>
                <input type="text" name="first" value="Sousa" />
                <input type="text" name="last" value="Gaspar" />
            </form>
             
            <table class="insideHeader">
                <tr><td>Im fine</td><td>Thanks</td></tr>
            </table>
        </div>
         
        <table id="contentTable" class="contentTable">
            <tr><td>contentA1</td><td>contentB1</td></tr>
            <tr><td>contentA2</td><td>contentB2</td></tr>
            <tr><td>contentA3</td><td>contentB3</td></tr>
            <tr><td>contentA4</td><td>contentB4</td></tr>
            <tr><td>contentA5</td><td>contentB5</td></tr>
        </table>
    </body>
</html>
```

The first step is to load our HTML document with the library.

```

$htmlFile = 'sample.html';
$document = new DONDocument();

$document->loadHTML($htmlFile);

```

At this moment the file is already loaded at variable $document.


**Retreating elements**

The DOMDocument library offer a lot methods the gives us a huge help when the goal is to retrieve elements. Let look to the first one.

```
$document->getElementById('header')->nodeValue;

```
The function **getElementById()** is responsible for searching an element inside the HTML file with the ID we passed as parameter and **nodeValue **is a property of DOMDocument class for exibit the content of the node.


The second way to retrieve HTML elements is by tag name. Let's do it.

```
$tables = $document->getElementsByTagName('table');
//Remember that we have 2 tables in oour HTML file.

$i = 0;

while($table = $tables->item($i))
{

}
```

