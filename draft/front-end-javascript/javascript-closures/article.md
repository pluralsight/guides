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
 In the JavaScript functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions and assigned to variables. In our example, we created the function *counter*, which returns the function *increment* and assignes it to the *incrementCounter* variable, so that variable contains a reference to the *increment* function. That reference enables us to call the *increment* function outside of the *counter* function's scope, so in our case, we call it from the global scope. If you are unfamiliar with JavaScript scopes, please read [this](https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/) great article before continuing with this tutorial. Beside that, the line `var incrementCounter = counter(0);` initializes `currentValue` to zero.  
 
The line `incrementCounter(1);` actually invokes the `increment` function with paramether 1. Since that function is a closure and it still has access to the variables and parameters of its outer function, although we call it outside of its outer function, the `currentValue` will be increased by 1 and its value will be changed to 1. In the next function calls, its value will be increased by 2 and 3, respectively, so the output of the code will be:    

```
currentValue = 1
currentValue = 3
currentValue = 6
```

In the JavaScript, local variables of a function will be destroyed after the function returns, unless there is at least one reference on them. In our example, the `currentValue` is referenced in the `increment` function, which is referenced in the global scope (in the `incrementCounter` variable, which points to that function). Therefore, the `currentValue` and `increment` will not be destroyed until the whole script execution is being terminated. That's why we can invoke the `increment` function from the global scope (using the reference to it, stored in the `incrementCounter`) after the `counter` function was executed.