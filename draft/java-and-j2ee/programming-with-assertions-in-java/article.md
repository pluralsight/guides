# Introduction
Probably, you have used the assertion methods of unit testing frameworks like [JUnit](http://junit.org) or third-party assertion libraries like [AssertJ](http://joel-costigliola.github.io/assertj/).

But, do you know that Java has a built-in assert mechanism? Have you used it?

Java's assertions were introduced in Java 1.4, and like the assertion methods of unit testing frameworks, they take a boolean expression to throw an exception if it evaluates to false.

However, they can be used anywhere in the code (with a few restrictions due to some good practices) and with different semantics than the other assertion methods.

Assertions are primary for testing things that you *assume* are true.

It may be things that you consider impossible to happen, things that must be present after executing a piece of code or something that must be true so your code to work.

This may sound like a simple concept, but it's the key to understand and use assertions properly.

In this guide, I'll show you how to use the Java assertion mechanism, the syntax and options to enable and disable assertions, some good practices, how to use assertion to program in the style of Design by Contract, and how they differentiate to exceptions and unit tests.

Let's start with the basics of how to use assertions.

# Using assertions

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

Because we are not supposed to catch `Error` and its subclasses.

It doesn't make sense to try to recover from assertion errors. Such an error would mean that the program is not operating in the expected conditions and it would be very likely that continuing the execution would lead to other errors.

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

# Assertions' best practices
There are some situations where it's better to use assertions.

### Do use assertions instead of comments.

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
 
But it cannot really be enforced when testing or running the program. Assertions can help you with this:
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

### Do use assertions to validate _unreachable_ code. 

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

However, take into account that truly unreachable code actually generates a compile-time error:
```java
if(interest < 0) {
  return 0;
} else {
  return interest;
}

assert false : "A valid interest should be returned"; // Compile time error
```

### Do use assertions to verify the result of an operation.

Don't think of assertions of something to only check input values or enforce control flow. Remember that you can place an assertion statement anywhere in your program.
 
An assert statement checks a boolean expression, however, you can use methods to check the results of an operation using complex criteria:
 ```java
String defaultPassw = generateDefaultPassword();

assert isValidPassword(defaultPassw);
```

And there are situations where assertions are not the best choice.

### Do not use assertions for argument checking in public methods.

In theory, a public method can be used by everyone who has access to its class (although with the introduction of Java 9's module system this may not be completely true). Most likely, they are part of your API.

For this reason, an assert statement is inappropriate, it can be disabled but the method has to enforce the validation of its arguments at all times. 

In addition, an assertion can only throw AssertionErrors. By manually checking the arguments, you have the options of using other exceptions or return an error code.

This practice doesn't apply to private methods because in these cases, you have more control over what should happen.

Also, note that this practice doesn't say you shouldn't use assertions in public methods. It's just about checking the input of those methods.

### Do not use assertions to perform an operation required by your application.

Having something like this:
```java
assert list.add(newElement);
```

It's a bad practice because the element won't be added when assertions are disabled.

This would be the correct way to do it:
```java
boolean elementAdded = list.add(newElement);
assert elementAdded;
```
 
### Do not be afraid to fill your code with assertion methods.

The more you check your assumptions, the more confident you'll be that those assumptions are guaranteed to be met when executing your code.

Even if something may seem obviously true to you, remember that bugs can appear in the least expected places. Sometimes it's difficult to see all the possible ways a condition can fail.

Yes, this may bloat your code, but the advantages of fixing bugs earlier and less debugging time (the stack trace of an `AssertionError` can help you pinpoint the source of the error)

If you're concerned about performance for having lot's of assert statements, don't be.

First of all, most of the time, assert operations will be simple comparisons. Then, assert statements should be enabled in development but disabled in production. Remember, they're internal checks to make sure your assumptions are correct, and they should not change the behavior of the application.

However, you can make use of an optimization the Java compiler does by placing all the assert statements inside an if block that tests for a `static final false` value:  
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

# Using design-by-contract style programming with assertions
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

The `require` clause checks the input, or preconditions, while the `ensure` clause checks the output or postconditions. Both of these conditions are part of the contract associated with this routine.

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

### You should run production code with assertions turned on.

Assertions are a development-time tool, they are not meant for production environments.

Assertions are a tool to validate your assumptions so you can take other measures to prevent errors. 

Why didn't the assumption hold true when testing this or performing that?

Find the root cause and fix it.

The point of using assertions is to prevent those errors from happening in production. They shouldn't be used as a barrier to avoid problems in production.

Besides, remember, we are not supposed to use or handle exceptions of type Error or its subclasses, which takes me to the next misconception.


### Instead of asserting a condition, I can just throw a RuntimeException.

Sometimes yes, you can throw an exception instead of using an assertion. In fact, assertions are built on top of exceptions, and an AssertionError can be caught like any other Java exception (not that we should do it).

But not in all cases you can substitute an assertion by an exception.

The purpose of assertions is different than the purpose of exceptions.

Assertions are intended to test assumptions and detect bugs. On the other hand, exceptions indicate *exceptional* conditions.

You can use assertions for internal logic checks within your code, unchecked exceptions for error conditions outside your immediate code's control, and checked exceptions for *business* and recoverable errors.

Besides, don't forget that assertions can be enabled or disabled (globally or for individual packages or classes). If you really care about something happening, maybe it's better to use an exception instead.

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

However, if you find that the assumption of the `default` clause being *unreachable*, most of the time doesn't hold true, it's better to use something like a meaningful exception:
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

### Unit tests substitute assertions.

In some way, assert statements serve as tests of your code.

However, neither unit tests substitute assertions nor assertions substitute unit tests.

Instead, assert statements can *complement* your unit tests.

Assertions validate assumptions about the internal state of your program. Things like preconditions, postconditions, and invariants.

In contrast, unit tests check the external behavior of your program. Generally, they follow the structure Arrange-Act-Assert. In other words, they set the preconditions (don't test them), perform the operation, and assert the result, all from an external perspective of the class or method being tested.

Yes, there can be some overlap between them, the assertions of the unit tests may be the same than the postconditions assertions, but they have their own and different purposes.

There's a limit of what you can do with assertion statements. Unit tests provide more flexibility and have a bigger scope than assertions. In fact, unit tests can drive the software development process (Test-driven development).

# Conclusion
The idea behind assertions is simple, a boolean condition is checked, it does nothing if it is true but throws an error if it is false.

Although it's a feature that is not used quite often (or not very well-known in some cases), it can turn out to be something helpful.

Assertions are more a development-time tool, they can help you to discover an invalid condition that could be the cause of an error.

In other words, they make sure that your application logic is correct.

They are different than exceptions:
- Exceptions can be caught, assertions are not meant to be caught. 
- Assertions can be disabled, exceptions cannot be disabled. 

They are different than unit tests:
- Assertions check the internal state of your program. Unit tests check the external behavior of your program.
- They have a bigger scope than assertions. Unit tests can drive the software development process (Test-driven development).

Java assertions can be used to program using design-by-contract style. Eiffel is a programming language that tightly integrates design by contract in its construct. It doesn't use something like an assert keyword, but in Java, you can structure your logic, in the same way, using asserts.

In the JVM ecosystem, Groovy is an example of a language that has a more [powerful support of assertions](http://docs.groovy-lang.org/next/html/documentation/core-testing-guide.html#_power_assertions) than Java. For example, they give a lot more information and they are enabled by default.

At the end of [this Oracle's document about assertions](https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html), you can find a response in a FAQ section that states:
> ... Note also that assertions were contained in James Gosling's original specification for the Java programming language. Assertions were removed from the Oak specification because time constraints prevented a satisfactory design and implementation.

I wonder, would assertions had become more popular among Java programmers if they were more formal (like the Eiffel implementation) or with more features (like Groovy's)?

What do you think?