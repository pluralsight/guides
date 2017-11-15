# Basics

JavaScript is an object oriented programming language. Almost everything in JavaScript is an object, except these primitive data types: *number*, *string*, *boolean*, *null* and *undefined*. For instance, *functions* and *arrays* are objects, which have their own properties and methods.

# Creating objects

There are several ways of creating new objects in JavaScript. We will consider each of them.

## Object literals

The most simple way of object creation is using the literal notation. Here are some examples:

```JavaScript
var emptyObject = {};

var person = {
    firstName: 'John',
    lastName: 'Smith',
    tel: '+38168123876'
};

var employee = {
    id: '1',
    employeeData: person,
    isFullTimeEmployee: true
}
```
