
Closures are very powerful mechanism in the JavaScript programming language. All members of an object in the JavaScript are public. Closures give an object possibility to have private and privileged members, as well, and not only that. In this tutorial we will learn what are closures and what are benefits of using them in the JavaScript code.

So, what is the closure? Douglas Crockford, author of the book *JavaScript: The Good Parts*, wrote an excellent definition of the closure, which says: 

>The closure is an inner function which always has access to the vars and parameters of its outer function even the outer function has returned.

Let's consider the following code sample:

```JavaScript
function counter(step) {
    
    var currentValue = 0;
    
    var increment = function() {
        
        currentValue += step; 
    }
    
    return increment;
}

var a = counter(2);
```
 In the JavaScript functions are objects, so they can be passed as arguments to other functions and they can be also returned from other functions. In our example, we created the function *counter*, which returns the function *increment*.
 
 After execution of the last line of the code, the var a will contain 

