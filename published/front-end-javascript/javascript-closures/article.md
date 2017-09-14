Closures are a very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public by default. However, the closure mechanism enables objects to have private members and more. In this tutorial, we will learn about closures and the benefits of using them in your JavaScript code.

# Closures

So, what is a closure? Douglas Crockford, author of the book *JavaScript: The Good Parts*, used some great examples of the closure mechanism [here](https://www.safaribooksonline.com/library/view/javascript-the-good/9780596517748/ch04s10.html). His definition of a closure is similar to the following: 

> The closure is an inner function which always has access to the variables and parameters of its outer function, even when the outer function has returned.

## Higher-order functions

Let's consider the following code:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var increment = function(step) {
        
        currentValue += step; 
        console.log('currentValue = ' + currentValue);
    };
    
    return increment;
}

var incrementCounter = counter(0);

incrementCounter(1);
incrementCounter(2);
incrementCounter(3);
```
 In the JavaScript language, functions are objects, so they can be passed as arguments to other functions, and they can be returned from other functions and assigned to variables. 
 
 In our example, we have created the `counter` function, which returns the `increment` function and assigns it to the `incrementCounter` variable, so that variable contains a reference to the `increment` function (the objects in the JavaScript are being copied by reference, not by value). 
 
 That reference enables us to call the `increment` function outside of the `counter` function's scope, so in our case, we called it from the global scope. Besides that, the line `var incrementCounter = counter(0);` initializes the `currentValue` variable. 
 
 Every call after this, `incrementCounter` serves as a reference for the inner function (referred to by `increment`). As such, the line `incrementCounter(1)` actually invokes the `increment` function with parameter 1. Since that function is a closure, **it still has access to the variables and parameters of its outer function**, although we called it outside of its outer function. Thus, the `currentValue` will be increased by 1 and its value will become 1. In the subequent function calls, `currentValue` will be increased by 2 and 3, respectively, so the output of the code will be:    

```
currentValue = 1
currentValue = 3
currentValue = 6
```

In JavaScript, local variables of a function will be destroyed after the function returns, unless there is at least one reference on them. In our example, the `currentValue` is referenced in the `increment` function, which is referenced in the global scope. Therefore, the `currentValue` and `increment` will not be destroyed until the entire script has executed. That's why we can invoke the `increment` function from the global scope even after the `counter` function has returned; **`increment` is being called by reference because it is stored in the `incrementCounter` function.** 
## Closure via objects

The `counter` function can also return more than one function, wrapped into an object. If we had `decrement` function and wanted to return it, as well, that could be accomplished with the following code:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var increment = function(step) {
        
        currentValue += step; 
        console.log('currentValue = ' + currentValue);
    };
    
    var decrement = function(step) {
    
        currentValue -= step;
        console.log('currentValue = ' + currentValue);
    }
    
    return {increment: increment,
            decrement: decrement};
}
```

The returned item has `increment` and `decrement` properties whose values are `increment` and `decrement` functions, respectively. In that way, we can expose those inner functions to the global scope, so we can call them from it:

```JavaScript

var myCounter = counter(0); // myCounter now refers to the object returned by counter

myCounter.increment(1); // call the appropriate "property" function using dot notation
myCounter.increment(2);
myCounter.decrement(1);
myCounter.increment(3);
myCounter.decrement(2);
```

If more than one function is being returned, they must be wrapped into an object, like in the example above. Even when a single inner function is being returned by an outer function, the output can be wrapped into an object, but that is typically overkill. 

We could also have functions inside of the `counter` function that we don't want to expose to the global (outer) scope. For instance, we could have a "private" function for logging the `currentValue`:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var logCurrentValue = function() {
        console.log('currentValue = ' + currentValue);
    }
    
    var increment = function(step) {
        
        currentValue += step; 
        logCurrentValue();
    };
    
    var decrement = function(step) {
    
        currentValue -= step;
        logCurrentValue();
    }
    
    return {increment: increment,
            decrement: decrement};
}
```

In this case, the `logCurrentValue` function cannot be accessed from the global scope, since it wasn't returned by the `counter` function. It can be used just within the `increment` and `decrement` functions, but **this function remains private in that it cannot be called externally**. 

Also, the `currentValue` and `initValue` variables are private for the `counter` function object.  In a nutshell, this is how we can emulate private members in JavaScript -- by restricting access using scope.

### Closure calls from multiple object references

Let's see how this mechanism works if we create multiple function objects. For instance, let's use `counter` to create two objects called `myCounter1` and `myCounter2` and call their property functions:

``` JavaScript
var myCounter1 = counter(0);
var myCounter2 = counter(3);

myCounter1.increment(2);
myCounter2.increment(2);
myCounter1.decrement(1);
myCounter2.decrement(1);
```
In the first two lines of the code, we have actually created 2 different objects, `myCounter1` and `myCounter2`. These objects have the same properties, `increment` and `decrement`, which are references to the `increment` and `decrement` functions. The `myCounter1` object is created by calling the `counter` function with parameter `0`, and the `myCounter2` by calling it with parameter `3`. This means that `increment` and `decrement` functions will have **different** values of the `currentValue` variable in the outer scope, so their calls on those two objects will produce different results. In this case, we have 2 different `increment` and `decrement` functions because we created 2 different objects each with a pair of functions that has its own scope.

This is a rather confusing edge case, but it makes the code easier to understand. The two variable references point to separate objects because `myCounter1` and `myCounter2` were instantiated using separate calls to `counter`. Thus, their calls to `increment` and `decrement` are separate as well.

The output of the code above will be:

```
currentValue = 2
currentValue = 5
currentValue = 1
currentValue = 4
```

# Use Cases

Closures are mostly used in callbacks, such as timeouts, event handlers, and so on, as well as in modules. 

## Timeouts

Let's consider the following code:

``` JavaScript
for (var i = 0; i < 5; i++) {

    console.log(i);
}
```

The output of this code are numbers from 0 to 4, sequentially.

Let's change it up a little bit:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i);
            
    }, 3000);
}
```

The output of the code above will be completely different; the number 5 will be logged 5 times.

Let's analyze this code. The function, which is passed as an argument to the `setTimeout` function, will be executed 3s after the first loop iteration. Therefore, the value of the variable `i`, which is in its outer scope (this function is also a closure), will become `5` before its execution because all 5 iterations will certainly be done within those 3 seconds. That function will be executed 5 times, 1 time for every iteration, and it will log `5` five times.

In other words, by the time that the Timeout finishes, the value of `i` will be 5, and this value is the one that will get logged.

### Using closures to fix the issue

In order to get the desired output, we will need to add one more closure, so our code should look like this:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout((function outer(i) {

        return function inner() {
            console.log(i); // 0 1 2 3 4
        }

    })(i), 3000);
}
```
So, now our code contains 2 closures, `outer` and `inner`, which both have access to the variable `i`, but there is a slight difference between them. The outer scope of the `outer` function is created once and the outer scope of the `inner` function is created 5 times, for every iteration separately. The outer scope of the `outer` function contains the variable `i`, created in the `for` loop, while the outer scope of the `inner` function contains this variable, an argument of the `outer` function. So, those are two different variables with the same name.

In the first iteration the value of the variable `i` is `0`. We have invoked the `outer` function immediately with parameter `0` and it returned the `inner` function. Thus, the `inner` function was actually provided as an argument to the `setTimeout` function instead of the `outer` function. Therefore, the `inner` function will be called after 3s and it will have access to its own outer scope. The value of the variable `i` in its scope is `0`, so the `inner` function will log `0` at the moment of its execution.

The same will happen in the second iteration, but this time the variable `i` will be `1`, and so on. Finally, we will get desired output in the console.

### An easier way

However, there is a simpler solution for this problem. The ECMAScript 2015 (or ES6) introduced some very useful new features and some of them are *let* and *const* keywords, which create block scoped variables. If we just replace the keyword `var` with `let` in this example, we will get the completely different output:

``` JavaScript
for (let i = 0; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i); // 0 1 2 3 4
            
    }, 3000);
}
```
The keyword `let` created a block scope for the variable `i`, for every loop iteration particularly. Therefore, we didn't have to put the line `console.log(i)` into a new closure in order to create a new scope for `i`, because the `let` keyword did it for us.

## Event Handlers

Let's consider the following example: there is a button on an HTML page and we want to show users information about how many times the button was clicked. We could write the following code to implement this functionality:

``` HTML
<!DOCTYPE html>
<html>
<head>
    <title></title>
	<meta charset="utf-8" />
</head>
<body>
    <button id="test-button">Test</button>
    <script>
        var counter = 0;
        var element = document.getElementById('test-button');

        //event handler
        element.onclick = function () {

            counter++;
            alert('Number of clicks:' + counter);
        };
    </script>
</body>
</html>
```

This code works fine, but we had to define a global variable for counting user clicks on the button. That's generally not recommended since we are polluting the global scope. We use the `counter` variable only in the event handler, so why not declare it inside the handler? However, if we just move its declaration to the handler's scope, it will not work as expected; every time the handler is called (i.e. every time the user clicks on the button), the `counter` will be reset to zero. 

The solution is to use a closure, like in the following code:

``` HTML
<!DOCTYPE html>
<html>
<head>
    <title></title>
	<meta charset="utf-8" />
</head>
<body>
    <button id="test-button">Test</button>
    <script>
        var element = document.getElementById('test-button');

        //event handler
        element.onclick = (function outer() {

            var counter = 0;
            
            return function inner() {

                counter++;
                alert('Number of clicks: ' + counter);
            };            
        })();
    </script>
</body>
</html>
```
In this code, we have defined the `outer` function which returns the `inner` function. The `outer` function is immediately invoked and after its execution, the `inner` function is being assigned to the `onclick` handler. Now, whenever the user clicks on the button, the `inner` function will be called instead of the `outer` function. Since it is a closure, it has access to the `outer` function's scope, so it can easily change the value of the `counter` variable. Therefore, the line `var counter = 0` will be executed just once, before returning the `outer` function. After that, the `outer` function will be called every time the user clicks on the button and it will increment the `counter` by 1, so the `counter` will always contain the accurate number of button clicks.

## The Module Pattern

One of the most popular patterns in JavaScript, the Module pattern, contains closures in its implementation. For instance, we could define the `counter` **module** in the following way:

``` JavaScript
var counter = (function() {

    var currentValue = 0;
    
    var logCurrentValue = function() {
        console.log('currentValue = ' + currentValue);
    }
    
    var increment = function (step) {

        currentValue += step;
        logCurrentValue();
    };

    var decrement = function (step) {

        currentValue -= step;
        logCurrentValue();
    }

    return {
        increment: increment,
        decrement: decrement
    };
})();
```
The module pattern is a special case of the singleton pattern, so there can be just one instance of the `counter` module. This is accomplished by the immediate invocation of function (IIFE) which returns an object that contains references to the public functions and assigns it to a global variable, in our case `counter`. We can call functions of the `counter` module like we called them on the `myCounter` object (substituting `counter` for `myCounter`). So the following code would work:

``` JavaScript
counter.increment(1);
counter.increment(2);
counter.decrement(1);

```
This will produce the following output:

```
currentValue = 1
currentValue = 3
currentValue = 2
```

In this case, the `currentValue` has been initialized to zero during the module initialization.

## Conclusion

This tutorial covered the most important things to remember when using closures. Hopefully, it gave you a better understanding of how this mechanism works and the kinds of problems that closures can solve. 
____

If you still need more clarification about the closures, feel free to leave questions or comments below. If you found this guide informative or interesting, please 'favorite' it. Thank you very much for reading this tutorial!