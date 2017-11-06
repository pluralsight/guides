# Introduction
You probably have used assertion methods when unit testing through frameworks like [JUnit](http://junit.org) or third-party assertion libraries like [AssertJ](http://joel-costigliola.github.io/assertj/).

But, do you know that Java has a built-in assert mechanism? If so, have you used it before?

Java's assertions were introduced way back in Java 1.4, and like the assertion methods of unit testing frameworks, they take a boolean expression to throw an exception if it evaluates to false.

However, they can be used anywhere in the code (with a few restrictions due to some good practices) and with different semantics than the other assertion methods.

> Assertions test things that you *assume* are true.

It may be things that you consider impossible to happen, things that must be present after executing a piece of code or something that must be true so your code to work.

This may sound like a simple concept, but it's the key to understanding and using assertions properly.

In this guide, I'll show you how to use the Java assertion mechanism, the syntax and options to enable and disable assertions, good assertion practices, the Design by Contract style of programming using assertions, and how assertions differentiate to exceptions and unit tests.

# Using assertions

Let's start with the basics of how to use assertions.

Assertions use the reserved `assert` keyword. It has the following syntax:
```java
assert booleanExpression;
```

Here's an example:
```java
private double calculateInterest(double amount) {
  assert amount > 0 && amount <= 1_000;
  
  double interest = 0.0;
  
  // calculate interest
  
  return interest;
}
```

If the condition evaluates to `false`, an exception of type `java.lang.AssertionError` will be thrown.

This means that the above example is equivalent to the following:
```java
private double calculateInterest(double amount) {
  if(amount > 0 && amount < 1_000) {
    throw new AssertionError();
  }
  
  double interest = 0.0;
  
  // calculate interest
  
  return interest;
}
```

As you might have guessed from its name, `AssertionError` is a subclass of `Error`:
![AssertionError class hierarchy](https://raw.githubusercontent.com/pluralsight/guides/master/images/7451a25c-8fb9-4c32-be15-bfe13c46d9ab.png)


Why is `AssertionError` a subclass of `Error` rather than `RuntimeException`?

Because we are **not** supposed to catch `Error` and its subclasses.

It doesn't make sense to try to recover from assertion errors. Such an error would mean that the program is not operating under conditions assumed by the programmer to be true. This would make it very likely that continuing the execution will lead to more errors down the line.

So the error will cause the program to stop and a stack trace will be printed, for example:
```
Exception in thread "main" java.lang.AssertionError
	at com.example.AssertTest.calculateInterest(AssertTest.java:6)
	at com.example.AssertTest.main(AssertTest.java:16)
```

That may not be very useful.

However, an assert statement can also take a message:
```java
assert booleanExpression : "Message about the error";
```
For example, if the following assertion fails:
```java
private double calculateInterest(double amount) {
  assert 
    amount > 0 && amount < 1_000 
    : 
    "Amount should be between 0 and 1,000: " 
        + amount;
  
  // ...
}
```

Something like the following will be printed to the standard output:
```
Exception in thread "main" java.lang.AssertionError: Amount should be between 0 and 1,000: -11.0
	at com.example.AssertTest.calculateInterest(AssertTest.java:6)
	at com.example.AssertTest.main(AssertTest.java:16)
```

Once again, the above example is equivalent to:
```
  if(amount > 0 && amount < 1_000) {
    throw new AssertionError(
      "Amount should be between 0 and 1,000: " 
        + amount
    );
  }
  
  // ...
}
```

Most of the time, you'll use a `String`, but if you look at the constructors of `AssertionError`, you'll see that it can also take other types that will be converted to a string:
```java
AssertionError(boolean detailMessage)
AssertionError(char detailMessage)
AssertionError(double detailMessage)
AssertionError(float detailMessage)
AssertionError(int detailMessage)
AssertionError(long detailMessage)
AssertionError(Object detailMessage)
AssertionError(String message, Throwable cause)
```

Which means that if it is enough for you, you can only have something like:
```java
private double calculateInterest(double amount) {
  assert amount > 0 && amount < 1_000 : amount;
  
  // ...
}
```

If the boolean expression evaluates to `true`, nothing will happen.

However, assertions are not enabled by default, so even if the assertion fails, if you don't run your program with a special flag, nothing will happen.

Assertions are enabled with either the `-ea` or `enableassertions` flags:
```
java –ea AssertTest
// Or
java –enableassertions AssertTest
```

This would enable assertions in all the classes of our program except for Java classes (system classes).

If you want to enable assertions in Java classes, you can use the `-esa` or `enablesystemassertions` flags:
```
java –esa AssertTest
// Or
java –enablesystemassertions AssertTest
```

You can also enable assertions for just one class:
```
java –ea:com.example.MyOtherClass AssertTest
```

Or in a specific named package and any of its subpackages:
```
java –ea:com.example... AssertTest
```

Or in the default package in the current working directory:
```
java –ea:... AssertTest
```

There's also an option to disable assertions:
```
java –da AssertTest
// Or
java –disableassertions AssertTest
```

And the corresponding option to disable assertions in Java classes:
```
java –dsa AssertTest
// Or
java –disablesystemassertions AssertTest
```

You might be wondering, why do we have this option if assertions are disabled by default?

Well, you can use the same options of the `ea` and `esa` flags to disable just one class or entire packages.

And you can also combine them. For example, you can enable assertions in all the classes of the package `com.example` except for the class `com.example.Utils`:
```
java –ea:com.example... -da:com.example.Utils AssertTest
```

Now let's review when it's recommended to use assertions and when it's not.

# Best practices for using assertions

In some cases, assertions are the way to go!

#### Use assertions instead of comments.

Something like the following is good for documentation purposes:
```java
void printTotals() {
  // ...
  String amountStr = amountToString(amount);
  // amountStr should not be null
  System.out.println("Amount: " + amountStr);
  // ...
}
```
 
But it's even better to enforce this check when testing or running the program. Assertions can help you with this and highlight potential bugs:
```java
void printTotals() {
  // ...
  String amountStr = amountToString(amount);
  
  // amountStr should not be null
  assert amountStr != null;  
  
  System.out.println("Amount: " + amountStr);
  // ...
}
```

Never again rely on using comments and a debugger. Use assertions instead!

### Use assertions to validate _unreachable_ code. 

This can be especially helpful when you don't want the execution flow to reach a certain point of your program. Like in the following example, you can place an assertion in the `default` clause of a `switch` statement to add an extra protection:
```java
switch(type) {
  case 1:
    //...
    break;
  case 2:
    //...
    break;
  case 3:
    //...
    break;
  default:
    assert false : "Default case reached";
}
```

However, take into account that truly unreachable code can generate a compile-time error:
```java
if(interest < 0) {
  return 0;
} else {
  return interest;
}

assert false : "A valid interest should be returned"; // Compile time error
```

### Use assertions to verify the result of an operation.

Don't think of assertions of something to only check input values or enforce control flow. Remember that you can place an assertion statement anywhere in your program.
 
An assert statement checks a boolean expression, however, you can use methods to check the results of an operation using complex criteria:
 ```java
String defaultPassw = generateDefaultPassword();

assert isValidPassword(defaultPassw);
```
Many beginners choose to use print statements here. Assertions go a step further by actually evaluating code and flagging it if it does not meet expectations.

### Where to avoid assertions

#### Do not use assertions to check arguments in public methods.

At last, a situation where assertions are not the best choice.

In theory, a public method can be used by everyone who has access to its class (although with the introduction of Java 9's module system this may not be completely true). Most likely, they are part of your API.

For this reason, an assert statement is inappropriate; it can be disabled, but the method has to enforce the validation of its arguments at all times. 

In addition, an assertion can only throw AssertionErrors. By manually checking the arguments, you have the options of using other exceptions or returning an error code.

This practice doesn't apply to private methods because in these cases, you have more control over what should happen.

Also, note that this practice doesn't say that you should not use assertions in public methods rather that you should not use them to check inputs to those methods.

#### Do not use assertions to perform an operation required by your application.

Having something like the following is bad practice because the `newElement` won't be added when assertions are disabled:
```java
assert list.add(newElement);
```

The correct way to do it would be:
```java
boolean elementAdded = list.add(newElement);
assert elementAdded;
```
 
### Do not be afraid to fill your code with assertion methods.

The more you check your assumptions, the more confident you'll be that those assumptions are guaranteed to be met when executing your code.

Even if something may seem obviously true to you, remember that bugs can appear in the least expected places. Sometimes it's difficult to see all the possible ways a condition can fail.

Yes, this may bloat your code, but the advantages of fixing bugs earlier and less debugging time (the stack trace of an `AssertionError` can help you pinpoint the source of the error)

And if you're concerned that all your assert statements will harm your performance, **don't be**.

First of all, most of the time, assert operations are simple comparisons that get enabled in development and  disabled in production. Remember, they're *internal* checks to make sure a developer's assumptions are correct. They should not change the behavior of the application.

However, you can make use of a Java compiler optimization by **placing all the assert statements inside an if-block** that tests for a `static final false` value:  
```java
static final boolean assertsEnabled = false;

// ...

if (assertsEnabled) {
  assert expr1; 
  assert expr2; 
  assert expr3; 
}
```

This will eliminate the `if` block from the generated class file because the compiler is smart enough to see that it will never be executed. You can see the explanation of this form of *conditional compilation* at the end of [this section of the Java Language Specification](https://docs.oracle.com/javase/specs/jls/se9/html/jls-14.html#jls-14.21).

For a deeper look at the effects of assertions on performance, look at some of the posts on [this StackOverflow thread](https://stackoverflow.com/questions/4624919/performance-drag-of-java-assertions-when-disabled). 

# Design-by-Contract style and assertions
Java's assert mechanism can be used for an informal design-by-contract style of programming.

[Design by Contract](https://en.wikipedia.org/wiki/Design_by_contract) is an approach to designing software that views software as a set of components whose interactions are based on mutual defined obligations or contracts in the form of:
- Acceptable and unacceptable input and return values or types
- Error and exception condition that can occur
- Side effects
- Preconditions
- Postconditions
- Invariants

This concept was conceived by [Bertrand Meyer](https://bertrandmeyer.com/), who designed the [Eiffel programming language](https://en.wikipedia.org/wiki/Eiffel_(programming_language)) based on this and other concepts of object-oriented programming.

For example, in Eiffel, you can have a routine with the following syntax:
```eiffel
processElement (e : ELEMENT) is
  require
    not e.empty
  do
    -- Perform the operation
  ensure
    e.is_processed
  end
```

Similar to assert, the `require` clause checks the input, or preconditions, while the `ensure` clause checks the output or postconditions. Both of these conditions are part of the contract associated with this routine.

But we can also have conditions that must apply to the whole class all the time (like before and after calling one of its methods), not just at a certain moment. These are known as class invariants:
```eiffel
invariant
  element_count > 0
```

You can think of invariants as a condition that is a precondition and a postcondition at the same time.

This way, [Eiffel's IDE](https://www.eiffel.com/eiffelstudio/) can provide a *short form* of the class in text format, without all the implementation of the class and just retaining the *contract* information:
```eiffel
class MyClass
feature
  processElement (e : ELEMENT) is
    require
      not e.empty
    ensure
      e.is_processed
    end
    
  -- Other features or routines
    
invariant
  element_count > 0
end
```

We don't have clauses or documentation mechanism like these in Java, but those preconditions, postconditions, and invariants are really some forms of assertions that can be implemented with the `assert` keyword.

You have seen examples of preconditions and postconditions (just remember that it's not recommended to use assertions to check the input of public methods), but in the case of invariants, Java assertions don't particularly enforce class or any kind of invariants.

If you want to do something equivalent that Eiffel class invariants, the recommended approach is to create a private method that checks for the invariant conditions:  
```java
private  boolean checkClassInvariants() {
  // ...
}
```

And use this method in assert statements placed at the beginning or/and at end of public methods, constructors, or any other places you think it will make sense:
```java
assert checkClassInvariants();
```

# Misconceptions about assertions

Let's talk about some popular misconceptions about assertions.

### Production code should be run with assertions turned on.

Assertions are a development-time tool **for testing purposes only**. They are not meant for production environments. They validate your assumptions to help the developer prevent errors down the line. 

Why didn't the assumption hold true when testing this or performing that? Answer this question, find the root cause and fix it. But do so in the development stages. The point of using assertions is to prevent those errors from happening by the time code becomes production-worthy. 

Besides, remember, we are not supposed to use assertions to handle errors, which takes me to the next misconception.

### Instead of asserting a condition, I can just throw a RuntimeException.

Sometimes yes, you can throw an exception instead of using an assertion. In fact, assertions are built on top of exceptions, and an AssertionError can be caught like any other Java exception (not that we should do it).

But cases exist in which you cannot substitute an assertion with an exception.

Once again, assertions serve a distinct purpose from exceptions. They are intended to test assumptions and detect bugs. Meanwhile, exceptions indicate *exceptional* conditions &mdash; problems for which the developer may not have accounted.

You can use assertions for internal logic checks within your code, unchecked exceptions for error conditions outside your immediate code's control, and checked exceptions for *business* and recoverable errors.

Besides, don't forget that assertions can be enabled or disabled (globally or for individual packages or classes). 

> If you really care if a particular process happens seamlessly, exceptions may be better.

For example, you can start by using an assumption to check that the execution flow doesn't reach a certain point of your program:
```java
switch(type) {
  case 1:
    //...
    break;
  case 2:
    //...
    break;
  case 3:
    //...
    break;
  default:
    assert false : "Default case reached";
}
```

Which is equivalent to:
```java
switch(type) {
  case 1:
    //...
    break;
  case 2:
    //...
    break;
  case 3:
    //...
    break;
  default:
    throw new AssertionError("Default case reached");
}
```

But with the advantage of having the option to disable the throwing of the exception.

And, as with customizable assert messages, it is better practice to use meaningful exceptions whenever possible:
```java
switch(type) {
  case 1:
    //...
    break;
  case 2:
    //...
    break;
  case 3:
    //...
    break;
  default:
    throw new IllegalStateException("Illegal type: " + type);
}
```

### Unit tests are substitutes for assertions.

In some way, assert statements serve as tests of your code.

However, unit tests should not substitute assertions and assertions should not substitute unit tests.

Instead, assert statements can *complement* your unit tests.

Assertions validate assumptions about the internal state of your program &mdash; things like preconditions, postconditions, and invariants.

In contrast, unit tests check the external behavior of your program. Generally, they follow the structure **Arrange-Act-Assert**. In other words, they set the preconditions (don't test them), perform the operation, and assert the result, all from an external perspective of the class or method being tested.

Yes, there can be some overlap between them, the assertions of the unit tests may be the same than the postconditions assertions, but they have different purposes.

There's a limit of what you can do with assertion statements. Unit tests provide more flexibility and have a bigger scope than assertions since they are specifically tailored to certain tasks. In fact, unit tests often drive the software development process, as seen by the test-driven development model. 

# Conclusion
The idea behind assertions is simple: a boolean condition is checked, it does nothing if it is true but throws an error if it is false.

Although it's a feature that is not used quite often (or not very well-known in some cases), it can turn out to be something helpful.

Assertions are more a development-time tool, they can help you to discover an invalid condition that could be the cause of an error.

In other words, they make sure that your application logic is correct.

They are different from exceptions:
- Exceptions can be caught, assertions are not meant to be caught. 
- Assertions can be disabled, exceptions cannot be disabled. 

They differ from unit tests too:
- Assertions check the internal state of your program. Unit tests check the external behavior of your program.
- They have a bigger scope than assertions. Unit tests can drive the software development process (Test-driven development).

Java assertions can be used to program using design-by-contract style. Eiffel is a programming language that tightly integrates design by contract in its construct. It doesn't use something like an assert keyword, but in Java, you can structure your logic, in the same way, using asserts.

In the JVM ecosystem, Groovy is an example of a language that has a more [powerful support of assertions](http://docs.groovy-lang.org/next/html/documentation/core-testing-guide.html#_power_assertions) than Java. For example, they give a lot more information and they are enabled by default.

At the end of [this Oracle's document about assertions](https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html), you can find a response in a FAQ section that states:
> ... Note also that assertions were contained in James Gosling's original specification for the Java programming language. Assertions were removed from the Oak specification because time constraints prevented a satisfactory design and implementation.

I wonder, would assertions had become more popular among Java programmers if they were more formal (like the Eiffel implementation) or with more features (like Groovy's)?

What do you think?

Let me know in the comments below and favorite this guide if you found it helpful or interesting. Thanks for reading!