Closures are very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public. Closures give an object possibility to have private and privileged members, as well, and not only that. In this tutorial we will learn what are closures and what are benefits of using them in the JavaScript code.

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
 In the JavaScript functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions and assigned to variables. In our example, we created the function `counter`, which returns the function `increment` and assignes it to the `incrementCounter` variable, so that variable contains a reference to the *increment* function. That reference enables us to call the `increment` function outside of the `counter` function's scope, so in our case, we call it from the global scope. If you are unfamiliar with JavaScript scopes, please read [this](https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/) great article before continuing with this tutorial. Beside that, the line `var incrementCounter = counter(0);` initializes the `currentValue` variable to zero.  
 
The line `incrementCounter(1);` actually invokes the `increment` function with paramether 1. Since that function is a closure and it still has access to the variables and parameters of its outer function, although we call it outside of its outer function, the `currentValue` will be increased by 1 and its value will be changed to 1. In the next function calls, its value will be increased by 2 and 3, respectively, so the output of the code will be:    

```
currentValue = 1
currentValue = 3
currentValue = 6
```

In the JavaScript, local variables of a function will be destroyed after the function returns, unless there is at least one reference on them. In our example, the `currentValue` is referenced in the `increment` function, which is referenced in the global scope (in the `incrementCounter` variable, which points to that function). Therefore, the `currentValue` and `increment` will not be destroyed until the whole script execution is being terminated. That's why we can invoke the `increment` function from the global scope (using the reference to it, stored in the `incrementCounter`) after the `counter` function returned.

The `counter` function can return more than one function, wrapped in a object. If we want to return the `decrement` function, as well, the counter function will look like this:

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

The returned object has `increment` and `decrement` properties, which values are `increment` and `decrement` functions, respectively. In that way, we exposed those functions to the outer scope, so we can use them in the following way:

```JavaScript

var myCounter = counter(0);

myCounter.increment(1);
myCounter.increment(2);
myCounter.decrement(1);
myCounter.increment(3);
myCounter.decrement(2);
```

For sure, when just one function is being returned, it can also be wrapped into an object, but it doesn't have to. If more than one function are being returned, they must be wrapped into an object, like in the example above. We could also have functions inside of the `counter` functions which we didn't want to expose to the global (outer) scope. For instance, we could have a separate function for logging the `currentValue`:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var logCurrentValue() {
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

In this case, the `logCurrentValue` function cannot be accessed from the global scope, since it wasn't returned from the `counter` function. Therefore, the `logCurrentValue` is a private function and the other two functions are public. The `counter` function actually represents the module pattern, one of the most used patterns in the JavaScript.

## Use Cases

Closures are mostly used in callbacks (timers, event handlers, ...) and modules.

### Callbacks

Let's consider the following code sample:

``` JavaScript
for (var i = 0; i < 5; i++) {

    console.log(i);    // 0 1 2 3 4 
}
```

The output of this code are numbers from 0 to 4, sequencially.

However, the output of the following code will be completely different, the number 5 will be logged 5 times:

``` JavaScript
for (var i = 1; i < 5; i++) {

    setTimeout(function() { 
    
        console.log(i); 
            
    }, 5000);     // 5 5 5 5 5
}
```

That is because the `function() {console.log(i)}` will be called 5s (5000ms) after the first loop iteration, then it will be called 5s after the second loop iteration and so on. That practically means that all 5 iterations will be executed before the first call of the function, so the value of `i` will be 5 and it will be logged 5 times.

In order to get the desired output, we will need to use the closure mechanism, so the code should look like this:

``` JavaScript
for (var i = 1; i < 5; i++) {

    setTimeout((function(i) { 
    
        console.log(i); 
            
    })(i), 5000);     // 5 5 5 5 5
}
```

