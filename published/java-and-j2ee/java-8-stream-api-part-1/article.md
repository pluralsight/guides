Java is a programming language used commonly throughout the world of software development. As of 2013, over 3 billion devices used Java, with the language being used primarily in web applications and Android applications. 

Nonetheless, people complain about the language being more verbose and syntactically demanding than its peers (such as Ruby and Python.) Some even say it is an outdated language.

Luckily, Java 8 brought many refreshing changes designed to mold Java into something more simple and modern. Better yet, at the time of writing, version 9 is coming with more changes soon.

One of the key changes is the [Stream interface](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html) which relies on a new Java component, lambda expressions.

This guide introduces lambda expressions and the Stream interface and highlights the most common Stream operations on collections.

In [part two](http://tutorials.pluralsight.com/java-and-j2ee/java-8-stream-api-part-2), you'll learn about more advanced methods (like reducing and collecting) and parallel streams.

# What is a lambda expression?

Let's begin by answering the question, what are lambda expressions in the context of java?

Lambda expressions make code more functional and less object-oriented, thus shortening its length. How about an example? 

Instead of writing something like:
```java
List<Toy> usedToys = findToys(toys,
     new Searchable() {
        public boolean test(Toy toy) {
           return toy.getType().equals(
                     ToyTypes.USED);
        }
});
```

Lambda expressions enable you to write:
```java
List<Toy> usedToys = findToys(toys,
     Toy toy ->
        toy.getType().equals(ToyTypes.USED);
```

The term lambda expression comes from lambda calculus, written as λ-calculus, where λ is the Greek letter lambda. This form of calculus deals with defining and applying functions.

As a result, lambdas simplify code in a way called functional programming, a different paradigm than object-oriented programming.

A lambda expression has three parts:

### A list of parameters

A lambda expression can have zero (represented by empty parentheses), one or more parameters:
```java
() -> System.out.println("Hi");
(String s) -> System.out.println(s);
(String s1, String s2) -> System.out.println(s1 + s2);
```

The type of the parameters can be declared explicitly, or it can be inferred from the context:
```java
(s) -> System.out.println(s);
```

If there is a single parameter, the type is inferred and is not mandatory to use parentheses:
```java
s -> System.out.println(s);
```

If the lambda expression uses as a parameter name the same as a variable name of the enclosing context, a compile error is generated:
```java
// This doesn't compile
String s = ""; s -> System.out.println(s);
```

### An arrow

Formed by the characters `-` and `>` to separate the parameters and the body.

### A body

The body of the lambda expressions can contain one or more statements.

If the body has one statement, curly brackets are not required and the value of the expression (if any) is returned:
```java
() -> 4; (int a) -> a*6;
```

If the body has more than one statement, curly brackets are required, and if the expression returns a value, it must be returned with a return statement:
```java
() -> {
     System.out.println("Hi");
     return 4;
}
(int a) -> {
     System.out.println(a);
     return a*6;
}
```

Returning is not necessary with lambda expressions. For example, the following are equivalent:
```java
() -> System.out.println("Hi");
() -> {
     System.out.println("Hi");
     return;
}
```

The signature of the abstract method of a functional interface provides the signature of a lambda expression (this signature is called a *functional descriptor*).

This means that **to use a lambda expression, you first need a functional interface**, which is just a fancy name for an interface with one method. For example:
```java
interface Searchable {
     boolean test(Car car);
}
```

In fact, lambda expressions don't contain the information about which functional interface they are implementing. The type of the expression is deduced from the context in which the lambda is used. This type is called the *target type*. 

So lambda expressions are an **alternative** to [anonymous classes](https://docs.oracle.com/javase/tutorial/java/javaOO/anonymousclasses.html), but they are not the same. 

They have some similarities:
- Local variables (variables or parameters defined in a method) can only be used if they are declared final or are effectively final.
- You can access instance or static variables of the enclosing class.
- They must not throw more exceptions than specified in the throws clause of the functional interface method. Only the same type or a supertype.

Some significant differences between lambdas and anonymous classes:

- For an anonymous class, the `this` keyword resolves to the anonymous class itself. For a lambda expression, `this` resolves to the enclosing class where the lambda is written.
- Default methods of a functional interface cannot be accessed from within lambda expressions. Anonymous classes can.
- Anonymous classes are compiled into inner classes, while lambda expressions are converted into private, static (in some cases) methods within their enclosing class. Using the `invokedynamic` instruction (added in Java 7), they are bound dynamically. Simply put, since there's no need to load another class, lambda expressions are more efficient than anonymous classes.

With this in mind, let's introduce the Stream interface.

# What is a Stream?

First of all, streams are **not** collections.

A simple definition is that streams are **wrappers for collections and arrays**. They wrap an existing collection (or another data source) to support operations expressed with lambdas, so you specify what you want to do, not how to do it.

### Characteristics of streams
- Streams work perfectly with lambdas.
- Streams don't store their elements.
- Streams are immutable.
- Streams are not reusable.
- Streams don't support indexed access to their elements.
- Streams are easily parallelizable.
- *Stream operations are lazy when possible.*

One thing that allows this laziness is the way their operations are designed. Most of them return a new stream, allowing operations to be chained and form a pipeline that enables this kind of optimizations.

To set up this pipeline you:
1. Create the stream.
2. Apply zero or more intermediate operations to transform the initial stream into new streams.
3. Apply a terminal operation to generate a result or a *side-effect*.

## Creating streams

A stream is represented by the `java.util.stream.Stream<T>` interface. This works with objects only.

There are also specializations to work with primitive types, such as `IntStream`, `LongStream`, and `DoubleStream`. Also, there are many ways to create a stream. Let's see the three most popular.

The first one is creating a stream from a `java.util.Collection` implementation using the `stream()` method:
```java
List<String> words = Arrays.asList(new String[]{"hello", "hola", "hallo", "ciao"});
Stream<String> stream = words.stream();
```

The second one is creating a stream from individual values:
```java
Stream<String> stream = Stream.of("hello","hola", "hallo", "ciao");
```

The third one is creating a stream from an array:
```java
String[] words = {"hello", "hola", "hallo", "ciao"};
Stream<String> stream = Stream.of(words);
```

## Intermediate operations

You can easily identify intermediate operations; they always return a new stream. This allows the operations to be connected.

```java
Stream<String> s = Stream.of("m", "k", "c", "t")
    .sorted()
    .limit(3)
```

An important feature of intermediate operations is that they don't process the elements until a terminal operation is invoked; in other words, they're lazy.

Intermediate operations are further divided into stateless and stateful operations.

**Stateless** operations retain no state from previous elements when processing a new element so each can be processed independently of operations on other elements.

Some examples are:
- `Stream<T> filter(Predicate<? super T> predicate)` 
    - Returns a stream of elements that match the given predicate.
- `<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)` 
    - Returns a stream with the content produced by applying the provided mapping function to each element. There are versions for `int`, `long` and `double` also.
- `<R> Stream<R> map(Function<? super T,? extends R> mapper)`  
    - Returns a stream consisting of the results of applying the given function to the elements of this stream. There are versions for `int`, `long` and `double` also.
- `Stream<T> peek(Consumer<? super T> action)`
    - Returns a stream with the elements of this stream, performing the provided action on each element.

**Stateful** operations, such as distinct and sorted, may incorporate state from previously seen elements when processing new elements.

Some examples are:

- `Stream<T> distinct()`. Returns a stream consisting of the distinct elements.
- `Stream<T> limit(long maxSize)`. Returns a stream truncated to be no longer than `maxSize` in length.
- `Stream<T> skip(long n)`. Returns a stream with the remaining elements of this stream after discarding the first `n` elements.
- `Stream<T> sorted()`. Returns a stream sorted according to the natural order of its elements.
- `Stream<T> sorted(Comparator<? super T> comparator)`. Returns a stream with the sorted according to the provided `Comparator`.

## Terminal operations

You can also easily identify terminal operations because they always return something other than a stream.

After the terminal operation is performed, the stream pipeline is consumed and can't be used anymore. For example:
```java
int[] digits = {0, 1, 2, 3, 4 , 5, 6, 7, 8, 9};
IntStream s = IntStream.of(digits);
long n = s.count();
System.out.println(s.findFirst()); // An exception is thrown
```

If you need to traverse the same stream again, you must return to the data source to get a new one. For example:
```java
int[] digits = {0, 1, 2, 3, 4 , 5, 6, 7, 8, 9};
long n = IntStream.of(digits).count();
System.out.println(IntStream.of(digits).findFirst()); // OK
```

The following methods represent terminal operations:
- `boolean allMatch(Predicate<? super T> predicate)`
    - Returns whether all elements of this stream match the provided predicate.
- `boolean anyMatch(Predicate<? super T> predicate)`
    - Returns whether any elements of this stream match the provided predicate.
- `boolean noneMatch(Predicate<? super T> predicate)`
    - Returns whether no elements of this stream match the provided predicate.
- `Optional<T> findAny()`
    - Returns an `Optional` describing some element of the stream.
- `Optional<T> findFirst()` 
    - Returns an `Optional` describing the first element of this stream.
- `<R,A> R collect(Collector<? super T,A,R> collector)`
    - Performs a mutable reduction operation on the elements of this stream using a `Collector`.
- `long count()`
    - Returns the count of elements in this stream.
- `void forEach(Consumer<? super T> action)`
    - Performs an action for each element of this stream.
- `void forEachOrdered(Consumer<? super T> action)`
    - Performs an action for each element of this stream, in the encounter order of the stream if the stream has a defined encounter order.
- `Optional<T> max(Comparator<? super T> comparator)`
    - Returns the maximum element of this stream according to the provided `Comparator`.
- `Optional<T> min(Comparator<? super T> comparator)`
    - Returns the maximum element of this stream according to the provided `Comparator`.
- `T reduce(T identity, BinaryOperator<T> accumulator)`
    - Performs a reduction on the elements of this stream, using the provided identity value and an associative accumulation function, and returns the reduced value.
- `Object[] toArray()`
    - Returns an array containing the elements of this stream.
- `<A> A[] toArray(IntFunction<A[]> generator)`
    - Returns an array containing the elements of this stream, using the provided generator function to allocate the returned array.
- `Iterator<T> iterator()` 
    - Returns an iterator for the elements of the stream.
- `Spliterator<T> spliterator()`
    - Returns a `Spliterator` for the elements of the stream.

## Operations on Collections
Usually, when you have a list, you'd want to iterate over its elements. A common way is to use a `for` block.

Either with an index:
```java
List<String> words = ...
for(int i = 0; i < words.size(); i++) {
    System.out.println(words.get(i));
}
```

Or with an iterator:
```java
List<String> words = ...
for(Iterator<String> it = words.iterator(); it.hasNext();) {
    System.out.println(it.next());
}
```

Or with the so-called `for-each` loop:
```java
List<String> words = ...
for(String w : words) {
    System.out.println(w);
}
```

The Stream interface provides a corresponding `forEach` method:
```java
void forEach(Consumer<? super T> action)
```

Since this method doesn't return a stream, it is a terminal operation.

Using it is not different from using the other methods:
```java
Stream<String> stream = words.stream();
// As an anonymous class
stream.forEach((new Consumer<String>() {
    public void accept(String t) {
        System.out.println(t);
    }
});
// As a lamba expression
stream.forEach(t -> System.out.println(t));
```

Of course, the advantage of using streams is that you can chain operations.
```java
words.sorted()
    .limit(2)
    .forEach(System.out::println);
```

Remember that because this is a terminal operation, you cannot do things like this:
```java
words.forEach(t -> System.out.println(t.length()));
words.forEach(System.out::println);
```

If you want to do something like that, either create a new stream each time:
```java
Stream.of(wordList).forEach(t -> System.out.println(t.length()));
Stream.of(wordList).forEach(System.out::println);
```

Or wrap the code inside one lambda:
```java
Consumer<String> print = t -> {
    System.out.println(t.length());
    System.out.println(t);
};
words.forEach(print);
```

Also, you can't use `return`, `break` or `continue` to terminate an iteration either. `break` and `continue` will generate a compilation error since they cannot be used outside of a loop and `return` doesn't make sense when we see that the `foreach` method is implemented basically as:
```java
for (T t : this) {
   // Inside accept, return has no effect
   action.accept(t);
}
```

Another common requirement is to filter (or remove) elements from a collection that don't match a particular condition.

You normally do this either by copying the matching elements to another collection:
```java
List<String> words = ...
List<String> nonEmptyWords = new ArrayList<String>();
for(String w : words) {
    if(w != null && !w.isEmpty()) {
        nonEmptyWords.add(w);
    }
}
```

Or by removing the non-matching elements in the collection itself with an iterator (only if the collection supports removal):
```java
List<String> words = new ArrayList<String>();
// ... (add some strings)
for (Iterator<String> it = words.iterator(); it.hasNext();) {
    String w = it.next();
    if (w == null || w.isEmpty()) {
        it.remove();
    }
}
```

For these cases, you can use the `filter` method of the Stream interface:
```java
Stream<T> filter(Predicate<? super T> predicate)
```

That returns a new stream consisting of the elements that satisfy the given predicate.

Since this method returns a stream, it represents an intermediate operation, which basically means that you can chain any number of filters or other intermediate operations:
```java
List<String> words = Arrays.asList("hello", null, "");
words.stream()
    .filter(t -> t != null) // ["hello", ""]
    .filter(t -> !t.isEmpty()) // ["hello"]
    .forEach(System.out::println);
```

Of course, the result of executing this code is:
```
hello
```

# Data Search
Searching is a common operation for when you have a set of data.

The Stream API has two types of operation for searching.

Methods starting with *Find*:
```java
Optional<T> findAny()
Optional<T> findFirst()
```

`find` methods search for an element in a stream. Since there's a possibility that an element can't be found (if the stream is empty, for example), `find` methods return an `Optional`.

The other way to search is through methods ending with *Match*:
```java
boolean allMatch(Predicate<? super T> predicate)
boolean anyMatch(Predicate<? super T> predicate)
boolean noneMatch(Predicate<? super T> predicate)
```

`match` methods indicate whether a certain element **matches** the given predicate. They return a boolean.

> Since all these methods return a type different than a stream, they are considered **terminal** operations.

`findAny()` and `findFirst()` practically do the same, they return the first element they find in a stream:
```java
IntStream stream = IntStream.of(1, 2, 3, 4, 5, 6, 7);
stream.findFirst()
    .ifPresent(System.out::println); // 1

IntStream stream2 = IntStream.of(1, 2, 3, 4, 5, 6, 7);
stream2.findAny()
    .ifPresent(System.out::println); // 1
```

Again, if the stream is empty, these return an empty `Optional`:
```java
Stream<String> stream = Stream.empty();
System.out.println(
    stream.findAny().isPresent()
); // false
```

`java.util.Optional<T>` is a new class also introduced in Java 8. *(If you want to know more about it, check out my tutorial [here](http://ocpj8.javastudyguide.com/ch14.html).)*

When to use `findAny()` and when to use `findFirst()`?

When working with parallel streams, it's harder to find the first element. In this case, it's better to use `findAny()` if you don't really mind which element is returned.

On the other hand, we have the `*Match` methods.

`anyMatch()` returns true if any of the elements in a stream matches the given predicate:
```java
IntStream stream = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream.anyMatch(i -> i%3 == 0)
); // true
```

If the stream is empty or if there's no matching element, this method returns `false`:
```java
IntStream stream = IntStream.empty();
System.out.println(
    stream.anyMatch(i -> i%3 == 0)
); // false

IntStream stream2 = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream2.anyMatch(i -> i%10 == 0)
); // false
```

`allMatch()` returns true only if **all** elements in the stream match the given predicate:
```java
IntStream stream = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream.allMatch(i -> i > 0)
); // true

IntStream stream2 = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream2.allMatch(i -> i%3 == 0)
); // false
```

If the stream is empty, this method returns **true** without evaluating the predicate:
```java
IntStream stream = IntStream.empty();
System.out.println(
    stream.allMatch(i -> i%3 == 0)
); // true
```

`noneMatch()` is the opposite of `allMatch()`, it returns true if **none** of the elements in the stream match the given predicate:
```java
IntStream stream = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream.noneMatch(i -> i > 0)
); // false

IntStream stream2 = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream2.noneMatch(i -> i%3 == 0)
); // false

IntStream stream3 = IntStream.of(1, 2, 3, 4, 5, 6, 7);
System.out.println(
    stream3.noneMatch(i -> i > 10)
); // true
```

If the stream is empty, this method returns also **true** without evaluating the predicate:
```java
IntStream stream = IntStream.empty();
System.out.println(
    stream.noneMatch(i -> i%3 == 0)
); // true
```

An important thing to consider is that all of these operations use something similar to the short-circuiting of `&&` and `||` operators.

Short-circuiting means that the evaluation stops once a result is found. Thus `find*` operations stop at the first found element.

With `*Match` operations, however, why would you evaluate all the elements of a stream when, by evaluating the third element (for example), you can know if all or none (again for example) of the elements will match?

# Sorting a Stream

Sorting a stream is simple.
```java
Stream<T> sorted()
```

The method above returns a stream with the elements sorted according to their natural order. For example:
```java
List<Integer> list = Arrays.asList(57, 38, 37, 54, 2);
list.stream()
    .sorted()
    .forEach(System.out::println);
```

Will print:
```
2
37
38
54
57
```

The only requirement is that the elements of the stream implement `java.lang.Comparable` (that way, they are sorted in natural order). Otherwise, a `ClassCastException` may be thrown.

If we want to sort using a different order, there's another version of this method that takes a `java.util.Comparator` (this version is not available for primitive stream like `IntStream`):
```java
Stream<T> sorted(Comparator<? super T> comparator)
```

For example:
```java
List<String> strings =
    Arrays.asList("Stream","Operations","on","Collections");
strings.stream()
    .sorted( (s1, s2) -> s2.length() - s1.length() )
    .forEach(System.out::println);
```

This method will print the following on execution:
```
Collections
Operations
Stream
on
```

# Data and Calculation Methods

The Stream interface provides the following data and calculation methods:
```java
long count()
Optional<T> max(Comparator<? super T> comparator)
Optional<T> min(Comparator<? super T> comparator)
```

The primitive versions of the Stream interface have the following methods:

**IntStream**
```java
OptionalDouble average()
long count()
OptionalInt max()
OptionalInt min()
int sum()
```

**LongStream**
```java
OptionalDouble average()
long count()
OptionalLong max()
OptionalLong min()
long sum()
```

**DoubleStream**
```java
OptionalDouble average()
long count()
OptionalDobule max()
OptionalDouble min()
double sum()
```

`count()` returns the number of elements in the stream or zero if the stream is empty:
```java
List<Integer> list = Arrays.asList(57, 38, 37, 54, 2);
System.out.println(list.stream().count()); // 5
```

`min()` returns the minimum value in the stream wrapped in an `Optional` or an empty one if the stream is empty.

`max()` returns the maximum value in the stream wrapped in an `Optional` or an empty one if the stream is empty.

When we talk about primitives, it is easy to know which the minimum or maximum value is. But when we are talking about objects (of any kind), **Java needs to know how to compare them** to know which one is the maximum and the minimum. That's why the Stream interface needs a `Comparator` for `max()` and `min()`:
```java
List<String> strings =
    Arrays.asList("Stream","Operations","on","Collections");
strings.stream()
    .min( Comparator.comparing(
                 (String s) -> s.length())
    ).ifPresent(System.out::println); // on
```

`sum()` returns the sum of the elements in the stream or zero if the stream is empty:
```java
System.out.println(
    IntStream.of(28,4,91,30).sum()
); // 153
```

`average()` returns the average of the elements in the stream wrapped in an `OptionalDouble` or an empty one if the stream is empty:
```java
System.out.println(
    IntStream.of(28,4,91,30).average()
); // 38.25
```	

# Conclusion

That's it for now. As you saw, the Stream interface is powerful and not very complicated.

In the [second part](http://tutorials.pluralsight.com/java-and-j2ee/java-8-stream-api-part-2), I'll cover more advanced methods like `map()`, `merge()` and `flatMap()`, and take a look at parallel streams.

____
*If you found this tutorial informative, please hit the "Favorites" button. Feel free to post comments and feedback in the discussion section below.*