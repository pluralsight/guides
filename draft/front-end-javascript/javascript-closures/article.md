Closures are a very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public by default. However, closures mechanism provides to objects possibility to have private members, and not only that. In this tutorial we will learn what the closures are and what are benefits of using them in the JavaScript code.

So, what is the closure? Douglas Crockford, author of the book *JavaScript: The Good Parts*, wrote an excellent definition of the closure, which says: 

>The closure is an inner function which always has access to the variables and parameters of its outer function, even the outer function has returned.

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
 In the JavaScript language, functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions and assigned to variables. In our example, we have created the `counter` function, which returns the `increment` function and assignes it to the `incrementCounter` variable, so that variable contains a reference to the `increment` function (the objects in the JavaScript are being copied by reference, not by value). That reference enables us to call the `increment` function outside of the `counter` function's scope, so in our case, we called it from the global scope. Beside that, the line `var incrementCounter = counter(0);` initializes the `currentValue` variable to zero.  
 
The line `incrementCounter(1);` actually invokes the `increment` function with parameter 1. Since that function is a closure and it still has access to the variables and parameters of its outer function, although we called it outside of its outer function, the `currentValue` will be increased by 1 and its value will become 1. In the next function calls, its value will be increased by 2 and 3, respectively, so the output of the code will be:    

```
currentValue = 1
currentValue = 3
currentValue = 6
```

In the JavaScript, local variables of a function will be destroyed after the function returns, unless there is at least one reference on them. In our example, the `currentValue` is referenced in the `increment` function, which is referenced in the global scope. Therefore, the `currentValue` and `increment` will not be destroyed until the whole script execution is being terminated. That's why we can invoke the `increment` function from the global scope (using the reference to it, stored in the `incrementCounter`) after the `counter` function returned.

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

The returned object has `increment` and `decrement` properties, which values are `increment` and `decrement` functions, respectively. In that way, we exposed those functions to the global scope, so we can call them from it:

```JavaScript

var myCounter = counter(0);

myCounter.increment(1);
myCounter.increment(2);
myCounter.decrement(1);
myCounter.increment(3);
myCounter.decrement(2);
```

For sure, when just one function is being returned, it can also be wrapped into an object, but there is no need for that. If more than one function are being returned, they must be wrapped into an object, like in the example above. We could also have functions inside of the `counter` functions which we don't want to expose to the global (outer) scope. For instance, we could have a "private" function for logging the `currentValue`:

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

In this case, the `logCurrentValue` function cannot be accessed from the global scope, since it wasn't returned from the `counter` function. It can be used just within the `increment` and `decrement` functions. Also, the `currentValue` and `initValue` variables are private for the `counter` function object.  So, that's how private members can be emulated in the JavaScript.

Let's see how would this mechanism work if we created more than one function object. For instance, let's create `myCounter1` and `myCounter2` objects and use them in the following way:

``` JavaScript
var myCounter1 = counter(0);
var myCounter2 = counter(3);

myCounter1.increment(2);
myCounter2.increment(2);
myCounter1.decrement(1);
myCounter2.decrement(1);
```
In the first two lines of the code, we have actually created 2 different objects, `myCounter1` and `myCounter2`. These objects have the same properties, `increment` and `decrement`, which are references to the `increment` and `decrement` functions. The `myCounter1` object is created by calling `counter` function with parameter `0`, and the `myCounter2` by calling it with parameter `3`. This means that `increment` and `decrement` functions will have different values of the `currentValue` variable in the outer scope, so their calls on those two objects will produce different results, although there are called with the same parameters, like in the example above. Actually, we have 2 different `increment` and `decrement` functions, because we created 2 different objects and each pair of those functions has its own scope. 

The output of the code above will be:

```
currentValue = 2
currentValue = 5
currentValue = 1
currentValue = 4
```

## Use Cases

Closures are mostly used in callbacks (e.g. timeouts, event handlers) and modules.

### Timeouts

Let's consider the following code sample:

``` JavaScript
for (var i = 0; i < 5; i++) {

    console.log(i);    // 0 1 2 3 4 
}
```

The output of this code are numbers from 0 to 4, sequencially.

However, the output of the following code will be completely different, the number 5 will be logged 5 times:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i); // 5 5 5 5 5
            
    }, 3000);
}
```
Let's analyse this code. The function, which is passed as an argument to the `setTimeout` function, will be executed 3s after the first loop iteration. This practically means that the value of the variable `i`, which is in its outer scope, will become `5` before its execution, because all 5 iterations will certainly be done within those 3 seconds. Then, it will be executed 5 times, 1 time for every iteration, and it will log `5` five times.

In order to get the desired output, we will need to use the closure mechanism, so our code should look like this:

``` JavaScript
for (var i = 0; i < 5; i++) {

    setTimeout((function outer(i) {

        return function inner() {
            console.log(i); // 0 1 2 3 4
        }

    })(i), 3000);
}
```
In the first iteration the value of the variable `i` is `0`. We have invoked the `outer` function immediately with parameter `0` and it returned the `inner` function, so the `inner` function was actualy provided as an argument to the `setTimeout` function instead of the `outer` function. Therefore, the `inner` function will be called after 3s and it will have access to its own outer scope. The value of the variable `i` in its scope is `0`, so the `inner` function will log `0` at the momment of its execution.

The same will happen in the second iteration, the `inner` function will be executed 3s later, but this time the variable `i` will be `1`, and so on. Finally, we will get desired output in the console.

However, there is a simpler soulution for this problem. The EcmaScript 2015 (or ES6) introduced some very useful new features and some of them are *let* and *const* keywords, which create block scoped variables. If we just replace the keyword `var` with `let` in this example, we will get completely different output:

``` JavaScript
for (let i = 0; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i); // 0 1 2 3 4
            
    }, 3000);
}
```
The keyword `let` created a block scope for the variable `i`, for every loop iteration particularly. Therefore, we didn't have to put the line `console.log(i)` into a closure in order to create a new scope for `i`, because the `let` keyword did it for us.

### Event Handlers

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

This code works fine, but we had to define a global variable for counting user clicks on the button. That's generally not recommended, since we are polluting the global scope. We use the `counter` variable just in the event handler, so why wouldn't we declare it inside the handler? However, if we just move declaration to the handler's scope, it will not work as expected, because every time the handler is called (i.e. every time the user clicks on the button), the `counter` will be reset to zero. The solution is to use a closure, like in the following code:

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
In this code, we have defined the `outer` function which returns the `inner` function. The `outer` function is immediately invoked and after its execution, the `inner` function is being assigned to the `onclick` handler. This means that whenever the user clicks on the button, the `inner` function will be called instead of the `outer` function. Since it is a closure, it has access to the `outer` function's scope, so it can easily change the value of the `counter` variable. Therefore, the line `var counter = 0` will be executed just one time, before returning the `outer` function. After that, the `outer` function will be called every time the user clicks on the button and it will increment the `counter` by 1, so the `counter` will always contain the accurate number of button clicks.

### The Module Pattern

One of the most popular patterns in the JavaScript, the Module pattern, contains closures in its implementation. For instance, we could define the `counter` module in the following way:

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
The module pattern is a special case of the singletone pattern, so there can be just one instance of the `counter` module. This is accomplished by immediate invocation of function which returns an object that contains references to the public functions. We can use the `counter` module in the same way as we used the `myCounter` object.

### Function Factory

``` JavaScript
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}

var add5 = makeAdder(5);
var add10 = makeAdder(10);

console.log(add5(2));  // 7
console.log(add10(2)); // 12
```

## Conclusion
