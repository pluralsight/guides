Closures are very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public. Closures give an object possibility to have private and privileged members, as well, and not only that. In this tutorial we will learn what are closures and what are benefits of using them in the JavaScript code.

So, what is the closure? Douglas Crockford, author of the book *JavaScript: The Good Parts*, wrote an excellent definition of the closure, which says: 

>The closure is an inner function which always has access to the vars and parameters of its outer function even the outer function has returned.

Let's consider the following code:

```JavaScript
function counter(initValue) {
    
    var currentValue = initValue;
    
    var increment = function(step) {
        
        currentValue += step; 
        console.log('New value: ' + currentValue);
        
    };
    
    return increment;
}

var incrementCounter = counter(0);

incrementCounter(1);
incrementCounter(2);
incrementCounter(3);
```
 In the JavaScript functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions and assigned to variables. In our example, we created the function *counter*, which returns the function *increment* and assignes it to the *incrementCounter* variable, so that variable contains a reference to the *increment* function. That reference enables us to call the *increment* function outside of the *counter* function's scope, so in our case, we call it from the global scope. If you are unfamiliar with JavaScripty scopes, please read [this](https://toddmotto.com/everything-you-wanted-to-know-about-javascript-scope/) great article before continuing with this tutorial.  
 
 
 
 According to the definition, the *increment* function is a closure. 
 
 
 
The output of the code above will be:
```
New value: 1
New value: 3
New value: 6
New value: 10
```

