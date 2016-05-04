## Have you ever written the following code?

```Java
try {
    doSomething();
} catch(InterruptedException swallowed) {
    // BAD BAD PRACTICE, TO IGNORE THIS EXCEPTION
    // just logging is also not a useful option here....
}
```

I have! I newer knew what the heck to do with those annoying `InterruptedException` when I simply wanted to call `Thread.sleep(..)`. What is this `InterruptedException`? Why is it thrown? Why can't I ignore it?

# This is what you should do!

Before I explain the whys, here is what you should do (if you don't re-throw):

```Java
try {
    doSomething();
} catch(InterruptedException e) {
    // Restore the interrupted status
    Thread.currentThread().interrupt();
}
```

## What is InterruptedException?

There is no way to simply stop a running thread in Java (don't even consider using the deprecated method `stop()`). Stopping threads is cooperative in Java. Calling `Thread.interrupt()` is a way to tell the thread to stop what it is doing. If the thread is in a blocking call, the blocking call will throw an `InterruptedException,` otherwise the interrupted flag of the thread will be set. A Thread or a `Runnable` that is interruptible should check from time to time `Thread.currentThread().isInterrupted()`. If it returns true, the thread should clean-up and return.

## Why is InterruptedException thrown?

The problem is that blocking calls like `sleep()` and `wait()`, can take very long till the check `Thread.currentThread().isInterrupted()` can be done. Therefore they throw an `InterruptedException`. However the `isInterrupted` is cleared when the `InterruptedException` is thrown! (I have some vague idea why this is the case, but for whatever reason this is done, that is how it is!)

## Why can't InterruptedException be simply ignored?

It should be clear by now: ignoring an `InterruptedException` means resetting the interrupted status of the thread. For example, worker threads take `Runnable` from a queue and execute, and check the interrupted status periodically. If you swallow the exception, the thread would not know that it was interrupted and would happily continue to run.

## Some more thoughts
Unfortunately it is not specified that `Thread.interrupt()` can only be used for cancellation. It can be used for anything that requires to set a flag on a thread. So, ending your task or `Runnable` early might be the wrong choice if the interrupted status is used for something else. Even though common practice is to use it for cancellation, if it is used for something else, your code does not have the right to reset the interrupt flag (unless you are the owner of the thread).

To learn more read the nice article [Dealing with `InterruptedException` by Brian Goetz](http://www.ibm.com/developerworks/library/j-jtp05236/).

Or read [Java Concurrency in Practice](https://books.google.de/books/about/Java_Concurrency_in_Practice.html?id=EK43StEVfJIC) by Brian Goetz, Tim Peierls, Joshua Bloch, Joseph Bowbeer, David Holmes, and Doug Lea. It's a great book!

##Summary

**If you don't know what to do with an `InterruptedException`, call `Thread.currentThread().interrupt()`**

####Note

This is based on my blog entry I have written 10 years ago ["Don't swallow InterruptedException. Call Thread.currentThread().interrupt() instead."](http://michaelscharf.blogspot.de/2006/09/dont-swallow-interruptedexception-call.html)
