Ruby is an object-oriented language. What does that even mean? It has unique quirks and characteristics that we’ll explain clearly. This article assumes that you have no programming experience, not even HTML.

An important skill to have when creating a program is translating — translating the desires of the user into the output they are looking for. In order to do that, you have to be able to think like a developer so that you can take what you know instinctively (as a user) and morph it into what the computer needs to be able to do what you want. So, we’ll help you start thinking like a developer. When you are done, you should have a mental model of how Ruby works and be on your way to becoming a successful Rubyista.

We’ll take you through a variety of the fundamental elements of the Ruby language and explain the whys behind the hows.

For all the code samples we go over, you can test them out on [Try Ruby](http://tryruby.org) (without having to install anything on your computer). You can follow Try Ruby’s tutorial if you want, but you don’t need to in order to understand what we’ll outline below. It’s just a quick way to get your feet wet without the headache of installing anything.

### How Is Your Ruby Code Evaluated by the Computer?

The interpreter for Ruby — basically, the main brain of the programming language that makes sense of the code you write — reads the code from top to bottom and left to right; meaning, it starts at line 1, character 1, literally, and first reads across line 1 to the last character, then goes down to the next line, and repeats this process until it reaches the last line of your program. If you have any syntax errors — i.e. errors in your code, such as misspelled variable names, improper use of constants (we’ll get to constants in a bit), etc. — it will halt execution and show you an error message, usually with a line number corresponding to the code. Remembering this is important because if you encounter an error report while coding, you will need to know how to decipher it. Figuring this out isn’t always straightforward for beginners.

<figure>[![](https://s3.amazonaws.com/static.written.com/matrixFictitious-code1416261243.jpg)](http://www.flickr.com/photos/15267728@N03/3369363558/)

<figcaption> Fictitious code from The Matrix. (Image: [Absolute Chaos](http://www.flickr.com/photos/15267728@N03/3369363558/)) </figcaption>

</figure>

This top-down parsing also affects the control of the flow of logic in your program. Say you want to calculate the balance of someone’s account before showing it to them. You would have to make sure that you put the method and function that does the calculation before the output of the balance; that is, if you are outputting the balance at line 10, then you would have to do the calculations somewhere between line 1 and 9\. We’ll dive into this later.

### Ruby Objects

An object is a thing. It is at the heart of Ruby. Going back to our earlier statement about Ruby being an object-oriented language, that means that Ruby manipulates all data on the assumption that the data is an object. There are many object-oriented languages, but very few put the object at the center of their universe like Ruby does. In Ruby, everything is an object. I mean everything: every variable, every operation. Every object has different characteristics; that’s what makes them different. A string is an object that has built-in characteristics that make it suitable for handling text. For a more technical definition, check out the article “[Object](http://en.wikipedia.org/wiki/Object_(computer_science))” on Wikipedia.

## Ruby Methods

A method is simply a definition of an action that can be performed on an object. Ruby has built-in object definitions and methods. One such method is `capitalize` for the Ruby class `strings` (we will dive into strings later).


``` ruby
string1 = "this string is awesome"
```


If you wrote `string1.capitalize`, the output would look something like this:


``` ruby
"This string is awesome"
```


All that the `capitalize` method tells the Ruby interpreter to do is convert the first character of the string from lowercase to uppercase. [Check out an example](http://ruby-doc.org/core-1.9.3/String.html#method-i-capitalize) directly from the Ruby documentation. [As you can see](http://ruby-doc.org/core-1.9.3/String.html) from the documentation, the `string` object in Ruby has a ton of methods that you can use right out of the box.

Another thing you should have noticed is the way to call a method, `string1.capitalize`, which is basically `{{object name}} . {{method name}}`. In this case, the object is a string variable. If you tried to do `capitalize` on an object that is not a string, Ruby would throw an error.You can create any method for any of your objects. Here is the way to do that:


``` ruby
def method_name
    #Enter code here
end
```


The `#` basically tells the Ruby interpreter that this is a comment for another human and to ignore it. So, the Ruby interpreter skips lines that begin with a `#`.

## Ruby Classes

A class is like a blueprint that allows you to create objects of a particular type and to create methods that relate to those objects. But classes have a special property called “inheritance.” Inheritance means just what you would think. When you inherit something from someone, it likely means a few things:

* That you are related in some way (in most cases, it is parent to children or grandparent to grandchildren);

* That either you are getting a bunch of stuff (land, money, etc.) or you have gotten some biological attribute (say, a nose shape or hair type).

<figure>[![](https://s3.amazonaws.com/static.written.com/blueprint-image1416261244.jpg)](http://www.flickr.com/photos/47325272@N00/2541408630/)

<figcaption> Classes are like a blueprint for objects. (Image: [Todd Ehlers](http://www.flickr.com/photos/47325272@N00/2541408630/)) </figcaption>

</figure>

Those principles are the same in Ruby. There are parent, grandparent and children classes. As a general rule, children classes inherit all of the attributes of a parent or grandparent class.In Ruby, an object’s grandparent class is known as its “superclass.” In other words, if you have an object that is a string — meaning that your object inherits the properties of the `String` class — then the parent class of `String` is `String`’s superclass. Be careful not to miss an important distinction here: the superclass of `String` (which is a class that tells Ruby how to treat `strings`) is _not_ the same as the superclass of a `String` object. Here is a demonstration:


``` ruby
> num1 = "this­ is a strin­g"
=> "this is a string"
> num1.class
=> String
> String.sup­erclass
=> Object
> Object.superclass
=> BasicObject
> BasicObjec­t.supercla­ss
=> nil
```


What we have done is set the local variables `num1` to be a string. When we check the class of `num1`, by calling the `.class` method, it tells us that the class of `num1` is `String`. Then, when we checked the superclass of `String`, it tells us `Object`, and so on.Look at what would happen if we tried `num1.superclass`:


``` ruby
> num1 = "this­ is a strin­g"
=> "this is a string"
> num1.super­class
=> #<NoMethodError: undefined method `superclass' for "this is a string":String>
```


The reason this doesn’t work is because `num1` is an object (a local variable) that has inherited the properties of the class `String`. And `num1` is _not_ a class, so it has no superclass.Here is another way to do what we did earlier:


``` ruby
> num1 = "this­ is a strin­g"
=> "this is a string"
> num1.class
=> String
> num1.class­.superclas­s
=> Object
> num1.class.superclas­s.supercla­ss
=> BasicObject
> num1.class­.superclas­s.superclass.supercl­ass
=> nil
```


The reason the last value is `nil` is because `BasicObject` has no parent. It inherits nothing from another class, so it stops there.One key thing we have done here that is different from before is we have “chained” methods, meaning we have continued applying a method to the current statement. That’s another beautiful thing about Ruby: every time it evaluates something, it returns a copy and allows you to continue evaluating it.Take the last line:


``` ruby
> num1.class­.superclas­s.supercla­ss.supercl­ass
=> nil
```


Basically, Ruby did this:

* What is the class of `num1` ? It’s a string, so return `String`.
* What is the superclass of `String` ? `String` is a child class of `Object`, so return `Object`.
* What is the superclass of `Object` ? `Object` is a child class of `BasicObject`, so return `BasicObject`.
* What is the superclass of `BasicObject` ? `BasicObject` is not a child class of anything, so return `nil`.

All on one line, all in one command. Simple, neat, elegant.The structure of classes and superclasses is the hierarchy of class inheritance.Now the question is, how do you define a class and use one? Glad you asked.


``` ruby
class MyClass
    # some code logic
end
```


That’s it.Basically, you just have the opening keyword, `class`, followed by the name of your class (`MyClass`, in this case). Then you have some code. And when you are done, you close it with the keyword `end`. Make sure that `class` and `end` are always all lowercase (i.e. don’t write `Class` or `End` or you might get errors).That’s all there is to it.If you have a parent class that you want this new class to inherit stuff from, you would define it like this:


``` ruby
class MyChildClass < MyClass
    # some code that is specific to the child class
end
```


Ruby interprets the `<` operator to mean that the class name on the right side is the parent and the class name on the left is the child (therefore, the child should inherit methods and such from the parent).Also, remember that class names usually start with an uppercase letter; and if their name has multiple words, you do what is called “CamelCasing” — i.e. instead of using a space or underscore or hyphen, you just start the new word with an uppercase letter.

### Ruby Class Instances

Now we know how to create a class, which we know is the blueprint of an object type. So, if you think of baking, a class is like a recipe (which contains a list of ingredients and instructions for creating something). But once you create something — say, blueberry muffins — then each muffin may be considered an “instance” of that class.So, each instance or muffin is an object.The way to create an instance is like this:


``` ruby
muffin = BlueberryMuffin.new
```


That’s it.To be technical, the only part of the statement above that actually creates an instance of the `BlueberryMuffin` class is `BlueberryMuffin.new`. In order to use the object, you have to store it somewhere, so we’ve stored it in the local variable `muffin` so that we can reuse this specific instance (or muffin).You will need to do more technical things with a class, like set up an initialization method so that whenever you create an object of the class, Ruby knows how to do that exactly. That is a bit beyond the scope of this article — just understand what a class is, how it relates to objects, how to create new objects, etc.To read up on classes, check out the article about them [on Learn Ruby The Hard Way](http://ruby.learncodethehardway.org/book/ex45.html).

## Data Structures in Ruby

**How is data structured?**
At the core of programming is the manipulation of data. Computer scientists have come up with a way to manipulate data in a structured way by inventing things called “data structures.” A data structure is simply a container for a particular type of data. Words are handled differently than formulas; likewise, characters and letters are handled differently than numbers — in most cases.

### Variables

**What’s a variable?**
A variable is the name of the most basic type of container that you will store data in. Each variable name has to be unique to its scope (i.e. the area in which the variable is allowed to exist). Think of it as a Venn diagram, in which each variable is only valuable in the circle or square within which it is contained.Say you wanted to create a program (or a part of a program) that is responsible for adding two numbers. From the coder’s point of view, you would need to set up a container for each of those numbers, and then set up the mathematical function between the containers. The reason to do this is because you don’t want the user to have to edit the source code every single time they want to calculate the sum. Although you could do that, the solution is neither practical nor efficient. Most users know what a calculator looks like, so they can just press the buttons or enter the numbers. But editing source code is a no-no.In Ruby, each of those containers is a variable. So, you would do something like this:


``` ruby
sum = num1 + num2
```


As opposed to something like this:


``` ruby
sum = 19 + 20
```


Ruby and many other languages have many types of variables. We’ll go over just a few to be brief and not confuse you too much.

* **Local**
    This is a variable that can be used only in a finite part of the program, such as a method or function (we’ll go over what these are later). Once you have exited that part of the program, those variables are destroyed. In fact, say you have a program that has three methods; you could have the same variable — say, `num1` — that is used in three different ways in each of those methods and that stores three different values. Going back to the Venn diagram, suppose there are three shapes within the diagram: Circle 1, Circle 2, Square. Also suppose that Circle 1 and Circle 2 are not connected, but both are within Square. A local variable would be confined to its respective circle and would not be able to affect anything outside of its circle. The way to use these variables is to just use them. If you want to use a local variable called `sum` that stores the sum of the values of `num1` and `num2`, you would simply write `sum = num1 + num2`.
* **Global**
    This is a variable that can be used throughout the entire program. Back to the Venn diagram, these variables would be within the square. This way, if you are inside any of the circles that are within the square, you can access a variable that is outside of the circles but within the square. You use these in Ruby by putting a `$` before the name. So, suppose you want to calculate multiple dimensions of a circle, and you want to define the radius beforehand. You would do something like this: `$radius = 20`. Then, at any other time throughout the program, regardless of whether you are in a subcircle of the square or in the square alone, you can reference `$radius`. Now, using global variables has a good side and bad side. The good side is that you can read the value of a global variable in any method or function within your program. The bad side is that you can also write to a global variable in any method or function within your program. If you change the value, forgetting that another method or function depends on the previous value could really screw things up. As a rule, then, stay away from global variables unless you are confident that you know where they will be used and how changes would affect the rest of the program.
* **Constants**
    These are “sacred” global variables. The values of these variables are _supposed_ to remain constant for the life of your program. Say you wanted to specify a mathematical constant such as pi that you could easily use throughout your program. You would do something like this: `PI = 3.14`. Constants have to begin with an uppercase letter, and more often than not they are all uppercase, but they don’t have to be. Note that I said that the values of constants are _supposed_ to be constant throughout your entire program, but they can be changed. Ruby doesn’t forbid you from changing the value, but when you do, it gives you a warning because it doesn’t like it. Going back to the Venn diagram, think of `PI` as being set outside of the square, and it can be used anywhere within the square and anywhere within the circles within the square.
* **Class**
    These are variables whose scope is limited to the class that they are defined in. Class variables are defined with `@@` at the beginning of the name of the variable.
* **Instance**
    These are variables whose scope is limited to one particular instance of a class. They are defined with `@` at the beginning of the name of the variable.

Here’s a recap on how to use the variable types:

* **Local**
    `sum = num1 + num2`
    Local variable names should start with a lowercase letter or an underscore.
* **Global**
    `$radius = 20`
    Global variable names should start with a `$`.
* **Constants**
    `PI = 3.14`
    Constants should start with an uppercase letter, but they are commonly written in full caps.
* **Class**
    `@@length = 10 #`
    This denotes the length of a side of an object in a class. I’ve used an imaginary class, called `Square`, and defined the length of each side for demonstration purposes. What’s important to note here is that all “squares” would have a “length” of `10` by default.
* **Instance**
    `@length = 5 #`
    This denotes the length of a side of a particular object. Suppose you wanted to create a red square that had a length of `5` instead of the default `10`. You could use this instance variable to specify the length of this particular square, your “Red Square.”

Note that these rules are by no means comprehensive. Some words you can’t use as variable names. They are called “reserved words,” which Ruby uses internally to identify various elements of the language.

<figure style="padding-left: 30px;"> [![](https://s3.amazonaws.com/static.written.com/Ruby-Reserved-Words-300x1231416261245.png)](http://www.smashingmagazine.com/wp-content/uploads/2012/04/Ruby-Reserved-Words.png)</figure>

To find out more about variables and other do’s and don’ts, check out the following resources:

* “[The Ruby Language](http://www.ruby-doc.org/docs/ProgrammingRuby/html/language.html),” Programming Ruby: The Pragmatic Programmer’s Guide

* “[Variables](http://www.rubyist.net/~slagell/ruby/variables.html),” Ruby User’s Guide

* “[Ruby Programming/Syntax/Variables and Constants](https://en.wikibooks.org/wiki/Ruby_Programming/Syntax/Variables_and_Constants),” Wikibooks

### Ruby Strings

**What is a string?**

A string is a series or sequence of characters — i.e. a “word” or sequence of words. You might say a sentence, but a string is not just a sentence. For instance:


``` ruby
string1 = 'a'
string2 = 'This is a string'
```


Two things are happening here. The first is that we are using local variables, and the second thing is that we are using single quotes to define the content of the variable. Even though `string1` contains just one letter, it is still a string because it is declared in single quotes. Ruby knows how to treat a variable by the way it is declared. You can use double quotes, but you have to be consistent. You can’t start the string’s declaration with a double quote and end with a single quote, like this: `string1 = "This is a string'`. But you can do this: `string1 = "This is a string"`, or `string2 = 'This too is a string'`. Both are valid, and it’s just a matter of taste.


``` ruby
num1 = 9
```


This sets `num1` to the numerical value of 9\. So, if you did `num1 + 1`, the result would be `10`.But if you used single quotes around the `9`, like this…


``` ruby
num1 = '9'
```


… then that would say that `9` is actually a string, not a number. So, if you wrote `num1 + 1`, it would throw an error along the lines of: `=> #`. The Ruby interpreter is basically saying that you have given it a number and a string and that it doesn’t know how to add them.To take that one step further, if you did this…


``` ruby
num1 = '9'
num2 = '1'
num1 + num2
```


… the result would be this:


``` ruby
"91"
```


Because Ruby would take the two strings and literally squish them together. When you specify a value in quotes (either single and double quotes), you are telling the Ruby interpreter, “Don’t translate this. Just take the exact content between the beginning and end quotes.” It treats the `9` like any other letter. So, as far as Ruby is concerned…


``` ruby
num1 = '9'
```


… is more or less the same as this:


``` ruby
num2 = 'a'
```


As a matter of fact, if you did `num1 + num2`, the result would be `9a`.In summary, a string is just a combination of letters, numbers and special characters.

## Ruby Collections

So far, we have covered individual pieces of data, such as one or a handful of items that can be stored in a local variable, or a single object created as an instance of a class.But what happens if we want to work with many pieces of data — that is, a collection, such as a series of numbers that we need to put in ascending order, or a list of names sorted alphabetically. How does Ruby manage that?Ruby gives us two tools: hashes and arrays.

### Arrays

The easiest way to explain an array is to show an image of what a “typical” one looks like.

<figure> [![](https://s3.amazonaws.com/static.written.com/Array21416261245.gif)](http://www.smashingmagazine.com/wp-content/uploads/2012/04/Array2.gif)</figure>

Rather than having six different variables for the six food types, we have just one food array that stores each food item in its own container or element. The numbers to the right of the diagram above are the “index” or “keys” (i.e. addresses) of each element (`[0] = chicken`, `[1] = rice`, etc). Note that the keys are always integers (whole numbers) and always start at 0 and go up from there. So, the first element is always `[0]`, and `[1]` is always the second element, etc. So, you will know that the range of keys of any array is always `[0]` to `(length-1)` — meaning that the last element is always total length of the array minus 1, because we started at `[0]`.To create the above in Ruby, we would do something like this:


``` ruby
food = ['chicken', 'rice', 'steak', 'fish', 'shrimp', 'beef']
=> ['chicken', 'rice', 'steak', 'fish', 'shrimp', 'beef']
> food.count
=> 6
```


Notice that for each element, we use single quotes (we could have used double quotes instead) because we are storing strings in each element. Ruby’s `array` class has some methods that we can use right out of the box, such as `count`, as used above. It simply counts the total number of elements in the array and outputs that value. Thus, even though the index goes up to 5, there are 6 elements because the index started at 0.Now that we have created a food array, we can access each item by invoking the name of the array that we created, followed by the index number.


``` ruby
> food[0]
=> "chicken"
> food[1]
=> "rice"
> food[2]
=> "steak"
> food[6]
=> nil
```


The reason we get `nil` at `food[6]` is because there is no `[6]` — or, rather, nothing is stored in `food[6]`, so Ruby automagically sets `food[6]`, `food[7]`, `food[8]` and so on to `nil`. To add another food item to this array, all you would have to do is set the next element to whatever value you wanted, like so:


``` ruby
> food[6] = 'carr­ots'
=> "carrots"
> food
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots"]
> food.count
=> 7
```


There is another way to add elements to your array in Ruby. You use the append operator, `<<`, which basically sticks something at the end of the array. The difference here is that we don’t have to specify an index position when using the append operator. We just do this:


``` ruby
> food << "irish potato"
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots", "irish potato"]
> food << 42
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots", "irish potato", 42]
```


Everything that comes after the `<<` is added to the array. This is pretty convenient because you can append variables and other objects to an array without worrying about the content itself. For instance:


``` ruby
> sum = 10 + 23
=> 33
> food << sum
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots", "irish potato", 42, 33]
```


All we did here was create a local variable named `sum`, and then push the value of `sum` to the end of the array. We can even add arrays to the end of other arrays:


``` ruby
> name_and_a­ge = ["Marc", "Gayle", 28]
=> ["Marc", "Gayle", 28]
> food
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots", "irish potato", 42, 33]
> food.count
=> 10
> food << name_­and_age
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots", "irish potato", 42, 33, ["Marc", "Gayle", 28]]
> food.last
=> ["Marc", "Gayle", 28]
> food.count
=> 11
```


Even though the last element is an array with three elements — `Marc`, `Gayle`, `28` — it still counts as just one element (i.e. one array) inside the food array. So, the count figure goes from 10 (before `name_and_age` is added) to 11.If we wanted to find out how many elements were inside the last element of the food array, we could do something like this:


``` ruby
> food.last.count
=> 3
```


A few other interesting methods that Ruby allows us to use right out of the box are `first`, `last`, `length`, `include?` (followed by the object you want to check for), `empty?`, `eql?` and `sort`.


``` ruby
> food
=> ["chicken", "rice", "steak", "fish", "shrimp", "beef", "carrots"]
> food.first
=> "chicken"
> food.last
=> "carrots"
> food.length
=> 7
> food.count
=> 7
> food.include?("chicken")
=> true
> food.inclu­de?("filet­ migno­n")
=> false
> food.empty­?
=> false
> food[0]
=> "chicken"
> food[0].eq­l?("chicke­n")
=> true
> food[0].eq­l?("beef")
=> false
> food.sort
=> ["beef", "carrots", "chicken", "fish", "rice", "shrimp", "steak"]
```


In the brackets right after `eql?`, we put the string in double quotes because we are dealing with a string. Also, `sort` arranges alphabetically on strings and from lowest to highest for numbers.We can store anything in each element, not just strings. We can even mix; some elements can be strings, others can be numbers.Say we wanted an array of numbers. We would do something like this:


``` ruby
numbers = [1, 2, 3, 4, 5, 6]
=> [1, 2, 3, 4, 5, 6]
```


Remember what we said earlier about always starting the index at `0`. You can see here why that is so important. In order to reference the number `1` in this array, the array reference has to be `[0]` because that is the first element in the array.


``` ruby
> numbers[0]
=> 1
> numbers[1]
=> 2
> numbers[6]
=> nil
> numbers.fi­rst
=> 1
> numbers.la­st
=> 6
> numbers.co­unt
=> 6
> numbers.le­ngth
=> 6
> numbers.in­clude?(3)
=> true
> numbers.in­clude?(10)
=> false
> numbers.em­pty?
=> false
> numbers[1]
=> 2
> numbers[1]­.eql?(1)
=> false
> numbers[1]­.eql?(2)
=> true
```


Because we are evaluating numbers, the objects in the brackets should not be wrapped in double quotes. In fact, if we did use double quotes, Ruby wouldn’t find the items because it would be looking for a string and not a number. Be careful with those quotes!


``` ruby
> numbers.in­clude?("3")
=> false
> numbers[1]­.eql?("2")
=> false
```


To see what other Ruby methods are included in the `array` class, check the [documentation on “Array.”](http://www.ruby-doc.org/core-1.9.3/Array.html)Everything we’ve just discussed covers one-dimensional arrays (i.e. arrays with just one column). These are best used to store lists of items.As you can imagine, there are multi-dimensional arrays. We’ll just touch on a 2-D array. Once you understand how to use them, you can then extrapolate to 3-D and beyond (if you ever want to go there).A 2-D array looks like this:

<figure> [![](https://s3.amazonaws.com/static.written.com/2D-Array11416261245.gif)](http://www.smashingmagazine.com/wp-content/uploads/2012/04/2D-Array1.gif)</figure>

We are storing two things: the name of the dish, along with a price related to that item.As the diagram suggests, in order to access each element, you would use both keys.This is how we would declare this array:


``` ruby
> food2 = [["ch­icken", 10], ["ric­e", 5], ["ste­ak", 20], ["fis­h", 15], ["shr­imp", 18], ["bee­f", 9]]
=> [["chicken", 10], ["rice", 5], ["steak", 20], ["fish", 15], ["shrimp", 18], ["beef", 9]]
```


A few key differences should jump out at you. Essentially, `food2` is an array of arrays (meaning that it is an array whose elements are themselves arrays). Huh? Well, look at each element.


``` ruby
> food2[0]
=> ["chicken", 10]
> food2[1]
=> ["rice", 5]
> food2[2]
=> ["steak", 20]
> food2[3]
=> ["fish", 15]
```


When you access each “single” element, you notice that each has an array inside of it; `["chicken", 10]` is an array that has a string (`chicken`) in the first element and a number (`10`) in the second element.So, to access each individual element, we would do something like this:


``` ruby
> food2[0]
=> ["chicken", 10]
> food2[0][0]
=> "chicken"
> food2[0][1]
=> 10
```


First, `food2[0][0]` is saying, “Show me the first element of the first element of the array `food2`.” And `food2[0][1]` is saying, “Show me the second element of the first element of the array `food2`.”You can also use the same methods of the Ruby class `array` on subarrays.


``` ruby
> food2
=> [["chicken", 10], ["rice", 5], ["steak", 20], ["fish", 15], ["shrimp", 18], ["beef", 9]]
> food2.coun­t
=> 6
> food2[0]
=> ["chicken", 10]
> food2[0].count
=> 2
> food2.last
=> ["beef", 9]
> food2.first
=> ["chicken", 10]
```


Keep in mind one important distinction for multi-dimensional arrays: Ruby will check whatever you call the method on.For instance, if you wanted to check whether `chicken` is in the `food2` array, you could not do this:


``` ruby
> food2.incl­ude?("chic­ken")
=> false
```


The reason is that `food2` is just an array of arrays. So, you would have to do something like this:


``` ruby
> food2
=> [["chicken", 10], ["rice", 5], ["steak", 20], ["fish", 15], ["shrimp", 18], ["beef", 9]]
> food2[0].include?("chicken")
=> true
```


We had to specify the particular element (`[0]`) that we wanted to check for the string `chicken`.In this case, we knew that the string `chicken` was stored in `food2[0]` because we put it there. How would we find it if we didn’t know? We’d have to use an iterator.

## Ruby Iterators

An iterator is a mechanism in Ruby that enables you to cycle through data structures that store multiple elements (such as an array) and examine each element. One of the most commonly used methods is named `each`. Each is a method in the array class that comes with Ruby.Let’s start simple. Suppose we wanted to print a list of all of our food items stored in the `food` array. How would we do this?


``` ruby
> food
=> ["chicken", "rice", "steak", "fish", "beef"]

food.each do |x|
puts x
end

chicken
rice
steak
fish
beef
```


A few things to be aware of here:

1. You can only call `each` on a collection of data.

2. Once you call `each`, you have to pass a block to it. A block is just a contained bit of code. Basically, you are saying to apply the code contained within the block to `each` element that you look at.

### Block

There are two ways to use a block. The first is similar to the example above, where you just do this:


``` ruby
do |variable| #some code end
```


Note that you have to use a block with an iterator. You can define a block outside of an iterator, but in order to execute the block, you have to use it in conjunction with an iterator. That’s why we called `do |x|` after `food.each` earlier.You can use one or more variables in your block. Those variables are local to the block alone, so they will be destroyed once you leave. Thus, if you had two blocks, you could use the variable `x` in both, and one wouldn’t affect the other.In the example above about food, we have said, for each element in the array `food`, print it to the screen.Another way to use a block is on one line, like this:


``` ruby
food.each { |x| puts x }
```


In this case, the opening curly brace (`{`) replaces the `do`, and the closing curly brace replaces the `end`. If your operation is just one line, then this way is convenient, although I have found that rereading such code in future is sometimes harder; so, I usually just use `do` and `end`, but that’s a personal preference. Do whatever makes you most comfortable.The reason that blocks use variables is because the elements of the collection are actually not modified — unless you specifically chose to do so. Basically, what happens is that for every single iteration through the array, a copy of the new element is stored in `x`, and then `x` is used in the block.Going through the `food` array, the local block variable `x` would look something like the following.

**First iteration:**


``` ruby
food[0] = 'chicken'
x = food[0]
x = 'chicken'
```


**Second iteration:**


``` ruby
food[1] = 'rice'
x = food[1]
x = 'rice'
```


**Third iteration:**


``` ruby
food[2] = 'steak'
x = food[2]
x = 'steak'
```


Using numbers would more clearly illustrate that the values aren’t changed in the original array:


``` ruby
> numbers = [1, 2, 3, 4, 5]
=> [1, 2, 3, 4, 5]
> numbers.each do |x|
… x = x + 2
… puts x
… end

3
4
5
6
7

> numbers
=> [1, 2, 3, 4, 5]
```


Here we’ve printed out the numbers `3, 4, 5, 6, 7` (i.e. `1+2, 2+2, 3+2`, etc.); but at the end, the `numbers` array is the same.

## Ruby Hashes

A [hash](http://www.ruby-doc.org/core-1.9.3/Hash.html) is another collection type. It is a collection of “key-value” pairs. A key-value pair is a combination of the name of a container (i.e. the key) and the contents of the container (i.e. the value).


``` ruby
a => "Marc"
```


In the key-value pair above, the key is `a`, and the value is `Marc`.A hash, then, is basically a list of these key-value pairs, separated by commas. A hash looks like this:


``` ruby
a =>"Marc", b => "Cheyenne", c => "Alexander", d=> "Mia"
```


Hashes and arrays have some key differences, though, and some things to note:

* The keys are not integer keys. They can be characters, integers, strings, etc. — basically, any Ruby object type.

* The keys are not ordered. So, you couldn’t say that `a` is “first” or that it “comes before” `b` in the example above, because Ruby does not look at the order of keys in hashes.

* Even though the keys are not ordered, if you were iterating through a hash (which we will do shortly), Ruby would go through them in the order in which they were added to the hash. In our example, if we were printing out each value, Ruby would print out `Marc`, `Cheyenne`, etc. But don’t confuse this with the way in which array keys are ordered.

There are multiple ways to initialize (or initially create) a hash, but the most popular ways look something like the following.To create an empty hash (i.e. a hash with no values):


``` ruby
> day = Hash.­new
=> {}
```


To create a hash with particular values:


``` ruby
> names = Hash[­"a" => "Marc­", "b" => "Chey­enne", "c" => "Alexander", "d" => "Mia"­]
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia"}
> names2 = {"a" => "Marc­", "b" => "Chey­enne"}
=> {"a" => "Marc", "b" =>"Cheyenne"}
```


You will notice that to create the hash, you don’t have to use the keyword `Hash` or square brackets (`[]`). You can use them if you like, or you can just use `= { }`.For the keys and values, you also don’t need to put the keys in quotes. You need to do that only if you want to use strings as the key. Ruby also requires a `=>` (pronounced “rocket”) to assign the value on the right side of the rocket to the key on the left side.If you tried to do `names2` without the quotes around the keys, you would likely get an error like this:


``` ruby
> names2 = { a => "Marc­", b => "Chey­enne"}
=> #<NameError: undefined local variable or method `a' for main:Object>
```


To access values within the hash, you have to specify the name of the hash, along with the key for the value you are trying to access.


``` ruby
> names
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia"}
> names["a"]
=> "Marc"
> names["c"]
=> "Alexander"
> names[a]
=> #<NameError: undefined local variable or method `a' for main:Object>
```


Because we didn’t use quotes for `names[a]`, the Ruby interpreter thinks that `a` is a local variable or a method and so can’t find a value for it, thus throwing an error.If you tried to access a seemingly legitimate value via a legitimate key that has not been assigned a value, then Ruby would usually return `nil`.


``` ruby
> day["a"]
=> nil
> day[9]
=> nil #For you Day9 fans, don't worry… I am a fan too :)
```


Suppose you wanted to create a hash in which every value has a “default” value. You could do something like this:


``` ruby
> year = Hash.­new("2012"­)
=> {}
> year[0]
=> "2012"
> year[12]
=> "2012"
```


All we’ve done was call the method `new` on the Ruby class `Hash` and pass the default value of `2012` into that method. So, when trying to access a value that doesn’t exist, instead of returning `nil`, Ruby would return the default value (`2012`).You can use a number of methods with hashes:


``` ruby
> names.keys
=> ["a", "b", "c", "d", "e"]
> names.values
=> ["Marc", "Cheyenne", "Alexander", "Mia", "Christopher"]
```


As you can guess, the `keys` just returns all of the keys in the hash, and the `values` returns all of the values.


``` ruby
> names.leng­th
=> 5
> names.has_­key?("a")
=> true
> names.has_­key?("z")
=> false
> names.has_­key("a")
=> #<NoMethodError: undefined method `has_key' for #<Hash:0x55c797d7>>
```


Note that the name of the `has_key` method is actually `has_key?`. If you left out the `?`, it would throw an error like the one above.All that `has_key?` is doing is checking the hash to see whether any key matches whatever is in the brackets. If it finds a match, then it returns `true`; if it doesn’t, it returns `false`.


``` ruby
> f_names = names
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher"}
> l_names = {"g" => "Gayl­e", "h" => "Gayl­e", "j" => "Jack­son", "m" => "Brow­n"}
=> {"g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
> f_names.me­rge(l_name­s)
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher", "g"=>"Gayle", "h"=>"Gayle","j"=>"Jackson", "m"=>"Brown"}
> f_names
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher"}
> l_names
=> {"g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
```


All we’ve done above was create a new hash, `f_names`, by assigning it the existing names `hash`. Then, we created another hash, `l_names`, that has a few last names. Then, we just merged the two hashes to create a master hash. However, because we just ran the `merge` method without assigning the result to any variable, it wasn’t stored. If you check the values of `f_names` and `l_names` after, you will see that they look exactly the same as before we ran `merge`.If we wanted to store the value of the merge, we would have had to do something like this:


``` ruby
> master_has­h = f_nam­es.merge(l­_names)
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher", "g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
```


Another approach is to do a “destructive” merge. This is an interesting feature of Ruby. For many (perhaps most) methods, if you add an exclamation point to the end of the method’s call, you actually replace the value of the method’s caller with the returned value. For example:


``` ruby
> f_names
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher"}
> l_names
=> {"g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
> f_names.me­rge!(l_names)
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher", "g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
> f_names
=> {"a"=>"Marc", "b"=>"Cheyenne", "c"=>"Alexander", "d"=>"Mia", "e"=>"Christopher", "g"=>"Gayle", "h"=>"Gayle", "j"=>"Jackson", "m"=>"Brown"}
```


As you can see, the `f_names` value after we ran the destructive merge method (`merge!`) is now the same value as the merged hash.Another method that you can use with hashes is `each`. But it is slightly different. With arrays, you just have to pass in one variable to the block (which essentially represents the index of the array). With hashes, you have to pass in two variables: one that represents the key, and another that represents the value.


``` ruby
> f_names.ea­ch do |key,­ value­|
.. puts "#{ke­y} is #{val­ue}"
.. end
=> "a is Marcb is Cheyennec is Alexanderd is Miae is Christopherg is Gayleh is Gaylej is Jacksonm is Brown"
```


This looks a little messy. Here is what’s happening:

* Reading from left to right, Ruby reads the left-most and oldest value first, and it stores those values in `key` and `value`. So, after the first iteration, `key` would be `a`, and `value` would be `Marc`.
* Then, Ruby goes inside the block and executes top down. The first command is `puts`, followed by a string. In other words, it will print everything in quotes to the screen.
* What is that strange syntax in the quotes after the `puts`? That’s called “[string interpolation](http://en.wikipedia.org/wiki/String_interpolation).” It basically says, stick the value of this variable into my string at this exact position. Thus, after the first iteration, `puts` would do this:
    1. Look for the key variable.
    2. Print the key variable to the screen (i.e. `a`).
    3. Then print a space (because we put a space between the key and the `is`).
    4. Print the next word (`is`).
    5. Then print another space.
    6. Then print the value variable (`Marc`). (The entire string, after the first iteration, would be `a is Marc`.)
    7. Go to the next command because this `puts` command is done.
    8. Sees `end`, so goes back to the beginning of the block to see whether any more elements are in this hash object.
* Because it is in a block, it just repeats this entire process for every key-value pair in the hash until there are no more.
* Because we didn’t add a space before the last double quotes on the `puts` line (and we didn’t put a space after the first quote on the `puts` line), no space will be between the last character of the first iteration and the first character of the second iteration.
* In other words, if `puts` looks like `puts " #{key} is #{value}"`, then the resulting string might make more sense: `a is Marc b is Cheyenne c is Alexander` etc.

I intended for the output to make sense, but when I saw the result, I realized that this has tripped me up many times in my career, so I figured to highlight it.You can [use a lot more methods on hashes](http://www.ruby-doc.org/core-1.9.3/Hash.html), many of which you should be familiar with because they look like others we have covered here, such as `value?` (note the `?` — I’m not asking a question here), and they look similar to the methods we went over in the arrays section, such as `include?`, `empty?`, `eql?`, `size`, etc.The last element of Ruby that you should be familiar with is an object type called a symbol.

### Symbols

A symbol is an object type that resembles a string, but is not quite one. The major difference between a symbol and a string is that a symbol always begins with a colon (like `:name`). (For more information, see the “[Symbol](http://www.ruby-doc.org/core-1.9.3/Symbol.html)” article in the Ruby documentation and “[The Ruby_Newbie Guide to Symbols](http://www.troubleshooters.com/codecorn/ruby/symbols.htm)” on Troubleshooters.com)Symbols work nicely with hashes because you can use them as the keys instead of strings.


``` ruby

> f_names
=> {:a =>"Marc", :b =>"Cheyenne", :c =>"Alexander", :d =>"Mia", :e =>"Christopher"}
> f_names[:a­]
=> "Marc"

```


The good thing about this is that you no longer have to worry about all of those quotes for both the keys and the values… but you can still remember the words for the keys.


``` ruby

> pets = {:dog­ => "Cook­ie", :cat => "Snow­y", :fish­ => "Gold­ie"}
=> {:dog=>"Cookie", :cat=>"Snowy", :fish=>"Goldie"}
> pets[:dog]
=> "Cookie"
> pets[:fish]
=> "Goldie"

```


Symbols make dealing with hashes much simpler than using strings as keys. You can, of course, use hashes for anything else in the Ruby language; their main function is to store values and make retrieval easier on the interpreter (since handling strings has many rules).

## Conclusion

I hope you have learned a lot here. Remember that this guide to Ruby is not comprehensive, but simply an introduction tailored to those with little or no programming experience. It’s not written in the typical programming tutorial style because I’ve always found that to be a bit difficult. I need to understand the whys behind the whats, so I’ve taken that approach here. I also don’t profess to be a Ruby ninja; I just wanted to learn how to build Web products myself, so I taught myself Ruby and Rails.You now have the foundation to play with Try Ruby some more or to install Ruby on your system and get started (Google it).Good luck, and remember that true learning often happens when you are struggling with a problem. When you spend one week stuck on a “very simple” problem and you eventually figure it out, you are guaranteed not to make that mistake again. And when you get stuck, don’t panic. Just take a break; maybe Google it and see what solutions others have had. But don’t just copy and paste code. Figure out why it does what it does and how it can help you. That’s how you learn.If I was unclear with anything, please let me know in the comments.

If you'd like additional practice with ruby using a hands-on approach, [check out this hack.hands( ) article about building a module to send yourself text message alerts from your ruby program completely for free!](https://hackhands.com/build-sms-alert-module-ruby-mechanize/)

### Additional Reading

There are fabulous books on Ruby to help get you started. Here are some of my favorites.

* [Why’s (Poignant) Guide to Ruby](http://mislav.uniqpath.com/poignant-guide/book/) This wonderful comic has become a classic in the Ruby community. In fact, its author (Why the Lucky Stiff — yes, that’s his name) disappeared a few years ago, which created a somewhat cultish mystique around the work that he did. His wife let the world know that he is fine, but he is no longer an active member of the Ruby community (Google him if you are interested in the saga).
* _[Humble Little Ruby Book](http://www.humblelittlerubybook.com/) _Buy the PDF or read it free online. The writing style is engaging.
* _[Eloquent Ruby](http://www.amazon.com/Eloquent-Ruby-Addison-Wesley-Professional-Series/dp/0321584104) _This book really helped me wrap my brain around “the Ruby way” of programming. It is a little more advanced than the two resources above, but once you have some of the basics down (i.e. once you have a solid understand of everything we’ve covered in this series), you should be able to learn a lot from this book. Russ’ tone is engaging and his writing easy to understand.
* [Programming Ruby: The Pragmatic Programmer’s Guide](http://www.ruby-doc.org/docs/ProgrammingRuby/) This is a little drier in presentation and tone, but rich in content. It is also known as Ruby Pickaxe and the Ruby Bible. A solid encyclopedia of all aspects of the Ruby programming language. The reason it is called Pickaxe is because it had a picture of a Pickaxe on the cover. The first version is free to read, although it is a bit outdated.
* [Programming Ruby 1.9: The Pragmatic Programmer’s Guide](http://pragprog.com/book/ruby3/programming-ruby-1-9) A more up-to-date Programming Ruby (aka Pickaxe). While not free, this one is a must have for all Rubyistas.

_Note: this article has been provided courtesy of Smashing Magazine._

