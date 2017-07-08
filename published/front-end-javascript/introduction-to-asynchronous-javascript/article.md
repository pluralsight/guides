In JavaScript we often need to deal with asynchronous behavior, which can be confusing for programmers who only have experience with synchronous code. This article will explain what asynchronous code is, some of the difficulties of using asynchronous code, and ways of handling these difficulties.

## What is the difference between synchronous and asynchronous code?

#### Synchronous code

In *synchronous* programs, if you have two lines of code (L1 followed by L2), then L2 cannot begin running until L1 has finished executing.

You can imagine this as if you are in a line of people waiting to buy train tickets. You can't begin to buy a train ticket until all the people in front of you have finished buying theirs. Similarly, the people behind you can't start buying their tickets until you have bought yours.

#### Asynchronous code

In *asynchronous* programs, you can have two lines of code (L1 followed by L2), where L1 schedules some task to be run in the future, but L2 runs before that task completes.

You can imagine as if you are eating at a sit-down restaurant. Other people order their food. You can also order your food. You don't have to wait for them to receive their food and finish eating before you order. Similarly, other people don't have to wait for you to get your food and finish eating before they can order. Everybody will get their food as soon as it is finished cooking. 

The sequence in which people receive their food is often correlated with the sequence in which they ordered food, but these sequences do not always have to be identical. For example, if you order a steak, and then I order a glass of water, I will likely receive my order first, since it typically doesn't take as much time to serve a glass of water as it does to prepare and serve a steak.

Note that *asynchronous* does not mean the same thing as *concurrent* or *multi-threaded*. **JavaScript can have asynchronous code, but it is generally single-threaded.** This is like a restaurant with a single worker who does all of the waiting and cooking. But if this worker works quickly enough and can switch between tasks efficiently enough, then the restaurant seemingly has multiple workers.

### Examples

The `setTimeout` function is probably the simplest way to asynchronously schedule code to run in the future:

```javascript
// Say "Hello."
console.log("Hello.");
// Say "Goodbye" two seconds from now.
setTimeout(function() {
  console.log("Goodbye!");
}, 2000);
// Say "Hello again!"
console.log("Hello again!");
```
If you are only familiar with synchronous code, you might expect the code above to behave in the following way:

 - Say "Hello".
 - Do nothing for two seconds.
 - Say "Goodbye!"
 - Say "Hello again!"

But `setTimeout` does not pause the execution of the code. It only schedules something to happen in the future, and then immediately continues to the next line. 

 - Say "Hello."
 - Say "Hello again!"
 - Do nothing for two seconds.
 - Say "Goodbye!"

#### Getting data from AJAX requests

Confusion between the behavior of synchronous code and asynchronous code is a common problem for beginners dealing with AJAX requests in JavaScript. Often they will write jQuery code that looks something like this:

```javascript
function getData() {
  var data;
  $.get("example.php", function(response) {
    data = response;
  });
  return data;
}

var data = getData();
console.log("The data is: " + data);
```

This does not behave as you would expect from a synchronous point-of-view. Similar to `setTimeout` in the example above, `$.get` does not pause the execution of the code, it just schedules some code to run once the server responds. **That means the `return data;` line will run before `data = response`, so the code above will always print "The data is: undefined".**

Asynchronous code needs to be structured in a different way than synchronous code, and the most basic way to do that is with *callback functions*.

### Callback functions

Let's say you call your friend and ask him for some information, say, a mutual friend's mailing address that you have lost. Your friend doesn't have this information memorized, so he has to find his address book and look up the address. This might take him a few minutes. There are different strategies for how you can proceed:

 - (Synchronous) You stay on the phone with him and wait while he is looking.
 - (Asynchronous) You tell your friend to call you back once he has the information. Meanwhile, you can focus all of your attention on the other tasks you need to get done, like folding laundry and washing the dishes. 

In JavaScript, we can create a *callback function* that we pass in to an asynchronous function, which will be called once the task is completed.

That is, instead of 

```javascript
var data = getData();
console.log("The data is: " + data);
```

we will pass in a callback function to `getData`:

```javascript
getData(function (data) {
  console.log("The data is: " + data);
});
```

Of course, how does `getData` know that we're passing in a function? How does it get called, and how is the `data` parameter populated? Right now, none of this is happening; we need to change the `getData` function as well, so it will know that a callback function is its parameter. 

```javascript
function getData(callback) {
  $.get("example.php", function(response) {
    callback(response);
  });
}
```

You'll notice that we were already passing in a callback function to `$.get` before, perhaps without realizing what it was. We also passed in a callback to the `setTimeout(callback, delay)` function in the first example.

Since `$.get` already accepts a callback, we don't need to manually create another one in `getData`, we can just directly pass in the callback that we were given:

```javascript
function getData(callback) {
  $.get("example.php", callback);
}
```

Callback functions are used very frequently in JavaScript, and if you've spent any amount of time writing code in JavaScript, it's highly likely that you have used them (perhaps inadvertently). Almost all web applications will make use of callbacks either through events (e.g. `window.onclick`), `setTimeout` and `setInterval`, or AJAX requests.

## Common problems with asynchronous code

### Trying to avoid asynchronous code altogether

Some people decide that dealing with asynchronous code is too complicated to work with, so they try to make everything synchronous. For example, instead of using `setTimeout`, you could create a synchronous function to do nothing for a certain amount of time:

```javascript
function pause(duration) {
  var start = new Date().getTime();
  while(new Date().getTime() - start < duration);
}
```

Similarly, when doing an AJAX call, it is possible to set an option to make the call synchronous rather than asynchronous (although this option is slowly losing browser support). There are also synchronous alternatives to many asynchronous functions in Node.js.

**Trying to avoid asynchronous code and replacing it with synchronous code is almost always a bad idea in JavaScript** because JavaScript only has a single thread (except when using Web Workers). That means the webpage will be unresponsive while the script is running. If you use the synchronous `pause` function above, or a synchronous AJAX call, then the user will not be able to do anything while they are running.

The issue is even worse when using server-side JavaScript: the server will not be able to respond to any requests while waiting for synchronous functions to complete, which means that *every user* making a request to the server will have to wait to get a response.

### Scope issues with callbacks inside loops

When you create a callback inside of a for-loop, you might encounter some unexpected behavior. Think about what you would expect the code below to do, and then try running it in your browser's JavaScript console.

```javascript
for(var i = 1; i <= 3; i++) {
  setTimeout(function() {
    console.log(i + " second(s) elapsed");
  }, i * 1000);
}
```

The code above is likely intended to output the following messages, with a second of delay between each message:

```
1 second(s) elapsed.
2 second(s) elapsed.
3 second(s) elapsed.
```

But the code actually outputs the following:

```
4 second(s) elapsed.
4 second(s) elapsed.
4 second(s) elapsed.
```

The problem is that `console.log(i + " second(s) elapsed");` is in the callback of an asynchronous function. By the time it runs, the for-loop will have already terminated and the variable `i` will be equal to `4`. 

There are various workarounds to this problem, but the most common one is to wrap the call to `setTimeout` in a closure, which will create a new scope with a different `i` in each iteration:

```javascript
for(var i = 1; i <= 3; i++) {
  (function (i) {
    setTimeout(function() {
      console.log(i + " second(s) elapsed");
    }, i * 1000);
  })(i);
}
```

If you are using ECMAScript6 or later, then a more elegant solution is to use `let` instead of `var`, since `let` creates a new scope for `i` in each iteration:

```javascript
for(let i = 1; i <= 3; i++) {
  setTimeout(function() {
    console.log(i + " second(s) elapsed");
  }, i * 1000);
}
```

### Callback hell

Sometimes you have a series of tasks where each step depends on the results of the previous step. This is a very straightforward thing to deal with in synchronous code:

```javascript
var text = readFile(fileName),
    tokens = tokenize(text),
    parseTree = parse(tokens),
    optimizedTree = optimize(parseTree),
    output = evaluate(optimizedTree);
    console.log(output);
```

When you try to do this in asynchronous code, it's easy to run into [callback hell](http://callbackhell.com/), a common problem where you have callback functions deeply nested inside of each other. Node.js code and front-end applications with lots of AJAX calls are particularly susceptible to ending up looking something like this:

```
readFile(fileName, function(text) {
  tokenize(text, function(tokens) {
    parse(tokens, function(parseTree) {
      optimize(parseTree, function(optimizedTree) {
        evaluate(optimizedTree, function(output) {
          console.log(output);
        });
      });
    });
  });
});
```

This kind of code is difficult to read and can be a real pain to try to reorganize whenever you need to make changes to it. If you have deeply nested callbacks like this, it is usually a good idea to arrange the code differently. There are several different strategies for refactoring deeply nested callbacks.

#### Split the code into different functions with appropriate names

You can give names to the callback functions so that you can reference them by their names. This helps to make the code more shallow, and it also naturally divides the code into small logical sections.

```javascript
function readFinish(text) {
  tokenize(text, tokenizeFinish);
}
function tokenizeFinish(tokens) {
  parse(tokens, parseFinish); 
}
function parseFinish(parseTree) {
  optimize(parseTree, optimizeFinish);
}
function optimizeFinish(optimizedTree) {
  evalutate(optimizedTree, evaluateFinish);
}
function evaluateFinish(output) {
  console.log(output);
}
readFile(fileName, readFinish);
```

#### Create a function to run a pipeline of tasks

This solution is not as flexible as the one above, but if you have a simple pipeline of asynchronous functions you can create a utility function that takes an array of tasks and executes them one after another.

```javascript
function performTasks(input, tasks) {
  if(tasks.length === 1) return tasks[0](input);
  tasks[0](input, function(output) {
    performTasks(output, tasks.slice(1));           //Performs the tasks in the 'tasks[]' array
  });
}
performTasks(fileName, 
  [readFile, token, parse, optimize, evaluate, function(output) {
    console.log(output);
  }]);
```

## Tools for dealing with asynchronous code

### Async libraries

If you are using lots of asynchronous functions, it can be worthwhile to use an asynchronous function library, instead of having to create your own utility functions. [Async.js](https://github.com/caolan/async) is a popular library that has many useful tools for dealing with asynchronous code.

###  Promises

[Promises](https://www.promisejs.org/) are a popular way of getting rid of callback hell. Originally it was a type of construct introduced by JavaScript libraries like [Q](https://github.com/kriskowal/q) and [when.js](https://github.com/cujojs/when), but these types of libraries became popular enough that promises are now [provided natively in ECMAScript 6](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise).

The idea is that instead of using functions that accept input and a callback, we make a function that returns a promise object, that is, an object representing a value that is intended to exist in the future.

For example, suppose we start with a `getData` function that makes an AJAX request and uses a callback in the usual way:

```
function getData(options, callback) {
  $.get("example.php", options, function(response) {
    callback(null, JSON.parse(response));
  }, function() {
    callback(new Error("AJAX request failed!"));
  });
}

// usage
getData({name: "John"}, function(err, data) {
  if(err) {
    console.log("Error! " + err.toString())
  } else {
    console.log(data);
  }
});
```

We can change the `getData` function so that it returns a promise. We can create a promise with `new Promise(callback)`, where `callback` is a function with two arguments: `resolve` and `reject`. We will call `resolve` if we successfully obtain the data. If something goes wrong, we will call `reject`.

Once we have a function that returns a promise, we can use the `.then` method on it to specify what should happen once `resolve` or `reject` is called.

```
function getData(options) {
  return new Promise(function(resolve, reject) {                    //create a new promise
    $.get("example.php", options, function(response) {
      resolve(JSON.parse(response));                                //in case everything goes as planned
    }, function() {
      reject(new Error("AJAX request failed!"));                    //in case something goes wrong
    });
  });
}

// usage
getData({name: "John"}).then(function(data) {
  console.log(data)
}, function(err) {
  console.log("Error! " + err);
});
```

The error handling feels a bit nicer, but it's difficult to see how we are making things any better given the size of the function. The advantage is clearer when we rewrite our [callback hell](#callbackhell) example using promises:

```javascript
readFile("fileName")
  .then(function(text) {
    return tokenize(text);
  }).then(function(tokens) {
    return parse(tokens);
  }).then(function(parseTree) {
    return optimize(parseTree);
  }).then(function(optimizedTree) {
    return evaluate(optimizedTree);
  }).then(function(output) {
    console.log(output);
  });
```

## Conclusion

At this point, you should be familiar with strategies for confronting some of the difficulties that arise when using asynchronous code. 

In this article I only gave a quick introduction to promises, but it is a large subject and there are many details that I left out. If you decide to use promises I would recommend reading a [more in-depth introduction](http://www.html5rocks.com/en/tutorials/es6/promises/) to them. 

I hope you enjoyed this tutorial. If you have any questions or feedback, feel free to leave a comment for me below.