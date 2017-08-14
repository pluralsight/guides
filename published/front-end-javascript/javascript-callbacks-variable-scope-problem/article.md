If you have ever experienced this JavaScript problem, you're not alone - it trips
up many people and can be tricky, at first, to understand exactly how to fix it.
Let's consider this example:

```javascript
var array = [ ... ]; // An array with some objects
for( var i = 0; i < array.length; ++i )
{
  $.doSthWithCallbacks( function() {
    array[i].something = 42;
  });
}
```

While this code looks perfectly fine, it shows the misunderstanding of a very
basic JavaScript concept. Now, if you're well-versed in Javascript, this error
should be easy enough to spot. But for most folks, this isn't case - some people
can literally spend hours trying to figure out why their code isn't working.

Explanation
-----------

Remember those first JavaScript tutorials you read, where it said that
JavaScript is asynchronous? This means that under some circumstances code
might not be executed sequentially. This is usually the case when using internal
APIs that depend on an external event. For example, processing a response after
an HTTP request is completed or after some other processing is done.

So what happens then, is that the `doSthWithCallbacks`
_(general expression for all JavaScript function that use a callback)_ schedules
the callback function to be executed at a later stage. But the `for` loop isn't
just scheduling one callback. It's scheduling an `array.length` worth of
callbacks and they most certainly won't be completed within the same `for` loop
iteration. Each of those callbacks will be executed at an unpredictable time
later on, when multiple `for` iterations have gone through, the value if `i` is
different and multiple other callbacks have also been scheduled.

Usually, the callbacks aren't executed until the `for` loop has completed, at
which point `i` is exactly equal to `array.length - 1`. So, every time any of
the callbacks is executed it will be modifying the last value of the array
instead of the value of the `for` loop iteration it was scheduled on. Of course,
as I said, it's unpredictable when the callbacks will be executed and depends on
multiple factors the JavaScript interpreter used, the function invoking the
callbacks and it's input data. An example is an HTTP request with a success
callback that won't be executed before the server sends a response, which could
be any time interval between several milliseconds and several minutes.

How to work around it
---------------------

I am going to present you with two solutions on how to go around the problem.
Both are very efficient in terms of performance and memory consumptions. The
first method is easier to understand, but requires the definition of a function
in an upper scope, which makes the code a bit less readable since you have to
look up the function. The second solution is my personal favorite but is harder
to understand, especially when you see such constructs for the first time. There
are other solutions, but these at the moment are the fastest and are supported
in all major browsers and JavaScript interpreters.

Essentially both methods do the same task but in different ways. What they do is
creating a separate callback function with their own copy of the value of `i` in
a scope only available to them.

Closure in a function
---------------------

This method is relatively easy to understand and that is why I won't be covering
it in much detail. This isn't the case with inline closures, so I will cover
that in depth. The `callbackClosure` function returns a function that invokes
the actual callback with an explicit copy of `i` as an argument.

```javascript
var array = [ ... ]; // An array with some objects

function callbackClosure(i, callback) {
  return function() {
    return callback(i);
  }
}

for( var i = 0; i < array.length; ++i )
{
  API.doSthWithCallbacks( callbackClosure( i, function(i) {
    array[i].something = 42;
  }) );
}
```

Since each function declares it's own scope, and `i` has a base atomic type
(`int`)  it is not passed as a reference, but rather as a copy (unlike objects)
which ensures that the actual callback will be executed against the correct
value.

Inline closure
--------------

This brings us to my most favorite JavaScript hack. This is done by declaring a
self called anonymous function, which generally looks like this:

```javascript
(function() {
  // Something declared here will only be available to the function below.
  // Code here is executed only once upon the creation of the inner function
  return function( callbackArguments ) {
    // Actual callback here
  }
}) (); // The last brackets execute the outer function
```

Note that the outer function is only used for encapsulating the inner function,
and creating a separate variable scope for the inner function. Also, the outer
function returns a value of type `Function` which is the exact type a callback
should be. So, applying this to the previous example we arrive here:

```javascript
var array = [ ... ]; // An array with some objects
for( var i = 0; i < array.length; ++i )
{
  API.doSthWithCallbacks( (function() {
    var j = i; // j is a copy of i only available to the scope of the inner function
    return function() {
      array[j].something = 42;
    }
  })() );
}
```

If, for example, you have to do some asynchronous processing, and there should
be some aggregate code that should only be run after all the callbacks have been
completed, all you need to do is know how many callback you have scheduled and
count how many of those were completed. If `count` is equal to `length` that
means that you are currently processing the last callback.

```javascript
var array = [ ... ]; // An array with some objects
var count = 0, length = array.length;
for( var i = 0; i < array.length; ++i )
{
  API.doSthWithCallbacks( (function() {
    var j = i; // A copy of i only available to the scope of the inner function
    return function() {
      array[j].something = 42;

      ++count;
      if( count == length ) {
        // Code executed only after all the processing tasks have been completed
      }
    }
  })() );
}
```

Now, at this point it's easy to get confused. Is the `++count` operation atomic?
A [Race Condition][wiki-race-condition] could occur and the code might be
executed multiple times or, worse, not executed at all. Some consider something
like a mutex or a semaphore. But this isn't right.

While JavaScript is asynchronous, it's not multithreaded. In fact, while it's
impossible to predict when a callback will be executed, it is guaranteed that a
Race Condition will not occur since JavaScript only runs in a single thread.
_(As a side note, that doesn't mean there isn't a way to run multiple threads in
JavaScript. See [Web Workers API][mdn-js-workers-api] for the Web kind of
JavaScript)_.

ES6
---

ECMAScript 6 introduces the `let` keyword which allows you to declare a variable
scoped to the nearest enclosing block and not global like `var` does. Thus the 
closure problem can be solved simply by replacing `var` with `let`:

```js
var array = [ ... ]; // An array with some objects
for( let i = 0; i < array.length; ++i )
{
  $.doSthWithCallbacks( function() {
    array[i].something = 42;
  });
}
```

Neat, isn't it? If you can use ES6 your code can be made to look so much better 
without the need to polute it with immedately invoked anonymous inline function
calls just to create a new scope.

Practice problem
----------------

I've prepared a [Practice Problem][jsfiddle-practice-problem] demonstrating the
issue and if you would like, you can give it a try. When attempting it, only
edit the code within the designated section. There are multiple ways to do it,
but one of the best is using the second technique I showed you.

About the author
----------------

Itay Grudev is a student currently pursuing a degree in _Computer Science and
Physics at the University of Aberdeen, United Kingdom_.

Itay is mostly interested in Linux, Security, Electronics and Amateur Radio. He
loves Open Source and Free Software. He is a talented developer and one of those
people spending time to change your `i++` to `++i`, crazy about efficiency and
beautiful code. His favorite technologies are `C++`, `Qt` and `Ruby on Rails`.

[mdn-js-workers-api]: https://developer.mozilla.org/en/docs/Web/API/Worker
[wiki-race-condition]: https://en.wikipedia.org/wiki/Race_condition
[jsfiddle-practice-problem]: https://jsfiddle.net/ItayGrudev/hmw0gk4c/

