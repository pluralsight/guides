# JSON

For those who are unfamiliar, JSON (pronounced just like the name Jason) is short for *JavaScript Object Notation*. It’s a serialisation technology (more on that below), so it pretty much does the same job as XML--but, for most programmers, JSON is generally lighter and more intuitive. JSON has grown in popularity ever since Douglas Crockford first proposed it as an alternative to XML about 15 years ago.

## Serialisation

Serialisation is all about converting values of different types into strings, and converting those strings back into values again. More properly, the term *serialisation* describes a process that generates a textual representation of a runtime value (like a bool, number or array). That ability wouldn’t be much use without *deserialisation*, which reverses the process by parsing a string of serialised data to generate a carbon copy of the original value.

To make things clearer, and to show how simple it is to use serialisation, we’ll step through a few lines of JavaScript code. This first line just sets up a runtime value:

~~~ javascript
let runtimeValue = { name: "Alice", age: 30 };
~~~

If you pass that value to a serialisation function, the function will return a textual representation of the value. Modern browsers provide the oddly named `JSON.stringify` for this, so we could do something like the following:

~~~ javascript
let serialisedValue = JSON.stringify(runtimeValue);
~~~

We call these strings *JSON text*.

Browsers provide the pleasantly named `JSON.parse` for deserialising JSON text, so this code would create a carbon copy of `runtimeValue`:

~~~ javascript
let carbonCopy = JSON.parse(serialisedValue);
~~~

Other languages use different names for the functions, but the API is always the same: You have one function for converting values into strings, and another for converting the strings back into values.

### Why serialisation is helpful

Serialisation allows you to do things with your data that you can’t do with runtime values. The two most common reasons for serialising a value are:

- To save it to disk, possibly in a file or database.
- To send it across a network, possibly over the Web, over a local network or just between processes on the same machine.

### Limitations

As with most tools in our kits, there are limitations. For one, some values can’t be reliably serialised. Functions are the most obvious example. Many languages treat functions as a type of data (*first-class functions*), and functions also have lexical scope, so it matters where the function is defined in the source code. In those languages, you can’t serialise a function because you can’t reliably recreate the environment in which the function was defined. For example, take the following code:

~~~ javascript
let x = 1;
let f = function() { console.log(x) };
~~~

If you serialised `f` and then deserialised the JSON text in some other process, we can only guess what the function would do if it were invoked. It might throw a reference error, because `x` is not defined there, or it might find something named `x` but it would not be the right `x`.

### Specialization

Because serialisation can be complicated, many programming languages have their own approaches, which are optimised for the language. For example, Python has `pickle,` which knows a lot about the Python type system, and can serialise a relatively broad range of Python objects.

Language specific approaches to serialisation allow for greater sophistication, but, in practice, we often need to transfer data between processes that are implemented in different languages. Maybe a JavaScript client wants to share data with a Ruby server. In those cases, we need something that is language agnostic.

## Enter JSON

JavaScript Object Notation is not exactly language agnostic. It uses syntax derived directly from JavaScript to serialise a subset of JavaScript types (Null, Bool, Number, String, Object and Array). However, in practice, JSON can still be used as though it is a language agnostic format because it restricts itself to using six data types that are common to virtually every high level language. Every normal programming language has some equivalence of a null value, true and false values, real numbers, unicode strings, hashes and arrays.

In JavaScript, `[1, 2, 3]` evaluates to an Array type value that contains three Number type values. We could serialise that array, and then save the string to a file. We could then read the file in Python, and deserialise the string using the `json` module from the Python Standard Library. Python does not have Array or Number types, but that is not a problem; it would just use List and Int instead.

Any value that a language can serialise into JSON text is known as a *JSON serialisable value*. For a given language, those values will have data types which are the equivalent of the JavaScript types Null, Bool, Number, String, Object and Array.

Note that what JavaScript calls an *object* is a bit different to the more complex objects used by most object oriented languages. The JavaScript Object type is roughly equivalent to what other languages call a *record*, *struct*, *dictionary*, *hash table*, *keyed list*, or *associative array*. It’s a container that associates keys with values.

The equivalency thing works in every direction. Amongst languages that support JSON, any process can share data with any other process, regardless of the languages being used. That’s why generic formats like JSON and XML are used much more widely than language specific approaches, like `pickle`. Still, that doesn't explain why JSON is displacing XML. Both are effectively language agnostic, and XML was here first.

## JSON thinks like we do

XML syntax is a slightly less forgiving version of HTML syntax. It represents data as nested trees of elements. For example:

~~~ xml
<person>
  <name>Alice</name>
  <age>30</age>
</person>
~~~

That can be made a little less verbose by using attributes:

~~~ xml
<person name="Alice" age="30"></person>
~~~

In a normal programming language, we would use a hash to achieve the same thing:

~~~ javascript
{ type: "person", name: "Alice", age: 30 }
~~~

JSON would represent that structure like this:

~~~ json
{"type":"person","name":"Alice","age":30}
~~~

Note that the number 30 is represented as a number in JSON, while every value in XML is just a string. Values in an XML document do not have types. To solve this problem, XML uses schemas (or schemata, if you prefer). XML requires each document to contain a link to a separate file that defines the schema of the structure, denoting the types of the values.

Schemas do not just fix the absence of data types, they also allow the validity of an XML document to be established without us having to write validation code, potentially in multiple languages. That can be advantageous. However, programmers do not normally think in terms of schemas, especially in loose, dynamically typed languages. Furthermore, and perhaps more importantly, maintaining schemas is a massive pain.

In JSON, we represent data the same way that we represent data in programs, using syntax to distinguish between values of different types, and that is JSON's biggest strength. Naturally, if you serialise an array of ints, you would ideally end up with something you can easily think of as a serialised array of ints, not an ambiguous document, along with a corresponding schema that explains that the document represents an array of ints.

## Using JSON

JSON is a simple thing to use. It has a few technicalities that you need to be aware of if you’re implementing a parser, but they’re not a concern for normal users. Still, there are a couple of things you need to understand to use JSON competently.

### The type system

There are six JSON types:

- Null: Represents the absence of a meaningful value.
- Bool: Represents absolute truth or absolute falsehood.
- Number: Represents a signed decimal with high precision.
- String: Represents a sequence of zero or more characters.
- Object: Represents a collection of zero or more key, value pairs.
- Array: Represents a sequence of zero or more values.

### The rules

Objects and arrays can recursively contain other objects and arrays, so JSON can serialise structures nested to any depth. The values contained by objects and arrays, must also be JSON serialisable. JSON can’t serialise an array of functions for the same reason that it can’t serialise a function.

The original JSON spec required that the value being serialised was either an object or an array. The values within that container could be of any JSON type, but the outer value had to be a container. Amendments to the spec mean that you can now serialise any JSON type directly. Just keep in mind that older JSON libraries may still adhere to the original spec.

## Never evaluate JSON text

JSON syntax is derived from JavaScript syntax, though they are not exactly the same. The differences relate to escaping unicode, and are only a concern for the authors of JSON libraries. However, one consequence of the syntactic similarity is that, in JavaScript, you can usually pass a JSON text to the `eval` function and it will return the correct value, as though you passed the text to `JSON.parse`. Using `eval` like that is a really bad idea.

If you can’t trust the source of a JSON text, then the text may contain something malicious, and `eval` would happily execute the malicious payload. The `JSON.parse` function does not have this issue, and will only parse valid JSON, which can’t contain anything executable.

## Containers, references and values

You now know how to use JSON, and how not to use it. However, to fully appreciate one of the subtleties of serialisation, you need to understand the difference between a reference and a value in JavaScript.

While objects and arrays contain values when they’re serialised, runtime objects and arrays do not. A runtime object contains keys, which reference values in memory. Those values are not inside the object; only the keys are. The values are just floating about in memory, and may simultaneously be referenced by names, and by other containers. Likewise, a runtime array contains a number of zero-indexed integers that each reference some value in memory. The array does not contain the actual values. Take the following code for example:

~~~ javascript
let value = 1;
let x = value;
let y = [value];
let z = {key: value};
~~~

That code creates four references to the same value. The expressions `value`, `x`, `y[0]` and `z.key` all evaluate to the exact same value in memory. Note that there is also nothing to stop you having multiple references to the same value inside the same container:

~~~ javascript
let value = 1;
let array = [value, value, value];
~~~

The difference between references and values has a number of consequences that can be very confusing until you appreciate the distinction, especially when dealing with mutable values. One consequence that is relevant to JSON is that you can’t serialise circular references.

### Infinite recursion

This code just sets up a perfectly valid circular structure:

~~~ javascript
let hash = {};
hash.key = hash;
~~~

That code is fine. However, if you then attempted to serialise `hash`, you would get a TypeError, complaining that you can’t convert a circular structure to JSON.

If you ever need to serialise circular structures, there are libraries available on GitHub. [This one][1] is quite popular.

[1]: https://github.com/WebReflection/circular-json

## An elegant solution

[JSON](https://app.pluralsight.com/library/courses/json-csharp-jsondotnet-getting-started/table-of-contents) is an especially elegant solution to serialisation. It’s effectively language agnostic, yet it directly works with values that we actually have in our programs. The JSON format is derived from JavaScript syntax, which is a lot like C syntax, so programmers generally find it easy to read and edit the serialised data directly, and we don’t have to reference schemas to do it.

### JSON everywhere

JSON is rapidly replacing XML for doing AJAX. Crockford once joked that the X in AJAX stands for JSON. Some people are now calling it "AJAJ." Like most people, I still say "AJAX," and just use JSON. It’s widely used for storing all kinds of structured data in files, and has become a de facto standard for configs.

A growing number of non-relational databases are based on JSON, including MongoDB and CouchDB. Other databases, such as DynamoDB from Amazon and the Google App Engine Datastore, have built-in support for working with JSON too. Many Web APIs only use JSON, and those that still use XML usually have the option to request JSON instead. HTML5 added localStorage and sessionStorage for client-side persistency. This only allows us to store strings; the logic being that we can just use JSON to store other types of data.

Of course, we can’t predict the future, but JSON is a safe bet. It already has widespread, solid support across the entire software ecosystem, and as we move towards a world based on cloud services, and one where everything happens in parallel, JSON has become our lingua franca.

### About the author

[Carl](https://hackhands.com/carlsmith/) is a self-taught, full-stack web developer. He began using Python back in 2009, mainly for scripting cloud services, and he is especially familiar with Google App Engine. He has extensive experience of browser technologies, is a big fan of jQuery, and a bit of a CoffeeScript evangelist. Carl has led the development of commercial applications, and he has published a couple of open source projects too.
