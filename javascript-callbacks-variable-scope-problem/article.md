The Callback Variable Scope Problem
-----------------------------------

As a HackHands expert I've often seen this JavaScript problem which people don't
understand how to fix.

Consider this example example:

```javascript
var array = [ ... ]; // An array with some objects
for( var i = 0; i < array.length; ++i )
{
  $.doSthWithCallbacks( function() {
    array[i].something = 42;
  });
}
```

While this code looks perfectly fine, it is an example of misunderstanding a
very basic JavaScript concept. Now for an expert like myself this error is quite
easy to spot and I am rather used to similar problems, but often that's not the
case with most people, who can literally spend hours trying to figure out why
their code isn't working.

Explanation
-----------

Remember those first JavaScript tutorials where it said that JavaScript is
asynchronus... This means that under some circumstances code might not be
executed sequentially. This is usually the case when using internal APIs that
depend on an external event. For example processing a response after an HTTP
request is completed or after some other processing is done. So what happens is
that the `doSthWithCallbacks` _(general expression for all JavaScript function
that use a callback)_ schedules the callback function to be executed at a later
stage. But the `for` loop isn't just scheduling one callback. It's scheduling an
`array.length` worth of callbacks and they most certainly won't be completed
within the same `for` loop iteration. Each of those callbacks will be executed
at an unpredictable time later, when multiple `for` iterations had gone through,
the value if `i` is different and multiple other callbacks had also been
scheduled. What usually happens is that the callbacks aren't executed until the
`for` loop has completed at which point `i` is exactly equal to
`array.length - 1`. So every time any of the callbacks is executed it will be
modifying the last value of the array instead of the value of the `for` loop
iteration it was scheduled on. Of course as I said it is unpredictable when the
callbacks will be executed and depends on multiple factors the JavaScript
interpreter used, the function invoking the callbacks and it's input data. An
example is an HTTP request with a success callback that won't be executed before
the server sends a response, which could be any time interval between several
milliseconds and several minutes.

How to work around it
----------------------

What you would like to do is create separate callback functions that have their
own copy of the value of `i` in a scope only available to them. This brings me
to my most favorite JavaScript hack. This is done by declaring a self called
anonymous function which generally looks like this:

```javascript
(function() {
  // Something declared here will only be available to the function below.
  // Code here is executed only once upon the creation of the inner function
  return function( callbackArguments ) {
    // Actual callback here
  }
}) (); // The last brackets execute the outer function
```

Note that the outer function is only used for encapsulating the inner function
and creating a separate variable scope for the inner function. Also the outer
function returns a value of type `Function` which is the exact type a callback
should be.

So applying this to the previous example we arrive at this:

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

If for example you have to do some asynchronous processing and there should be
some aggregate code that should only be ran after all the callbacks had been
completed, this is how you will do it:

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

Now at this point people are often confused. Is the `++count` operation atomic?
A [Race Condition][wiki-race-condition] could occur and the code might be
executed multiple times or worse - not executed at all. Some consider something
like a mutex or a semaphore. Wrong! While JavaScript is asynchronous, it is not
multithreaded. In fact while it is impossible to predict when a callback will be
executed, it is guaranteed that a Race Condition will not occur since JavaScript
only runs in a single thread. _(As a side note, that doesn't mean there isn't a
way to run multiple threads in JavaScript. See
[Web Workers API][mdn-js-workers-api] for the Web kind of JavaScript)_.

Practice Problem
----------------

I've prepared a [Practice Problem][jsfiddle-practice-problem] demonstrating the
issue and if you would like, you can give it a try. When attempting it, only
edit the code within the designated section. There are multiple ways to do it,
but one of the best is using the technique that I showed you.

About the author
----------------

![Itay Grudev](https://gravatar.com/avatar/37aeb9f5242f93cec35e98e464ed7424?s=200)

Itay Grudev is a student currently pursuing a degree in _Computer Science and
Physics at the University of Aberdeen, United Kingdom_.

Itay is mostly interested in Linux, Security, Electronics and Amateur Radio. He
loves Open Source and Free Software. He is a talented developer and one of those
people spending time to change your `i++` to `++i`, crazy about efficiency and
beautiful code. His favorite technologies are `C++`, `Qt` and `Ruby on Rails`.

[mdn-js-workers-api]: https://developer.mozilla.org/en/docs/Web/API/Worker
[wiki-race-condition]: https://en.wikipedia.org/wiki/Race_condition
[jsfiddle-practice-problem]: https://jsfiddle.net/ItayGrudev/hmw0gk4c/

