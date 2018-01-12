# Basics

JavaScript is an object oriented programming language. Almost everything in JavaScript is an object, except these primitive data types: *number*, *string*, *boolean*, *null* and *undefined*. For instance, *functions* and *arrays* are objects, which have their own properties and methods. We will discuss about them later in this tutorial. Let's see, for beginning, what the objects are and how they can be created.

## What is an object?

Object is a collection of key/value pairs. Keys are also known as properties. An object can be empty, if it doesn't contain any key/value pair. For instance, this is an empty object:

``` JavaScript
var x = {};
```

This is an example of a simple object (`x` and `y` are properties which have values `5` and `"abc"`, respectively):
``` JavaScript
var a = {
    x: 5,
    y: "abc"
};
```

Values of properties can be functions, as well. For instance, the object *a* could have the following properties: 

``` JavaScript
var a = {
    x: 5,
    y: "abc",
    z: function() {
       console.log("This is a method.");
    }
};
```

As you can see, the value of the property *z* is a function, which will be executed if we call it in the following way:
```JavaScript
a.z();
```

A property which value is a function is called **method**.

## What is a prototype?
Every object has its prototype. The prototype is actually a base object which an object is inherited from. Objects `x` and `a` from the above examples are inherited from the [Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype), so they have access to all methods of the `Object.prototype`and access to their own properties, as well.

## How objects can be instantiated?

There are several ways of creating new objects in JavaScript. We will start from the simplest one.

### Object literals

The most simple way of object creation is using the literal notation. The objects in previous examples are created using the literal notation. Here are some more examples:

```JavaScript
var emptyObject = {};

var person = {
    firstName: 'John',
    lastName: 'Smith'
};

var employee = {
    id: '1',
    contactData: {
        email: test@test.com
        phone: 062345678
    },
    isFullTimeEmployee: true
}
```

Objects can be nested into each other, like the `contactData` in the `employee` object.

We could initialize the *person* object in the above example in the following way, as well:
```JavaScript
var person = {};

person['firstName'] = 'John';
person['lastName'] = 'Smith';
```

So, we can use square brackets to access (or create) object properties. They do the same thing as the *dot* operator does, but they are useful if a property needs to be the result of an expression. For instance, this code will also work:

```JavaScript
var nameSuffix = 'Name';

var person = {};

person['first' + nameSuffix] = 'John';
person['last' + nameSuffix] = 'Smith';
```

### Object.create()

Objects can be created by using the `Object.create()` function
