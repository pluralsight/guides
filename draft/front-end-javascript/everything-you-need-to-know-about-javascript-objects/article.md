# Basics

JavaScript is an object oriented programming language. Almost everything in JavaScript is an object, except these primitive data types: *number*, *string*, *boolean*, *null* and *undefined*. For instance, *functions* and *arrays* are objects, which have their own properties and methods. We will discuss about them later in this tutorial. Let's see, for beginning, what the objects are and how they can be created.

## What is an object?

Object is a collection of key/value pairs. Keys are also known as properties. An object can be empty, if it doesn't contain any key/value pair.

## How can it be instantiated?

There are several ways of creating new objects in JavaScript. We will start from the simplest one.

### Object literals

The most simple way of object creation is using the literal notation. Here are some examples:

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

Objects can be nested into each other, like the `contactData` in the `employee` object. Even an empty object actually has access to  methods of the [Object.prototype](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/prototype). One of those methods is the `toString` method, so, we could execute, for instance, this line of code:
``` JavaScript
console.log(emptyObject.toString());
```
and get the following output in the console:
```
[object Object]
```

The `[object Object]` means that the `emptyObject` is an `object` which prototype is the `Object.prototype`. There are different types of objects in JavaScript (functions, arrays, dates etc.), so we should get different results in the console for those types of objects.