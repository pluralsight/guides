In the [first part of this tutorial on the Java 8 Stream API](http://tutorials.pluralsight.com/java-and-j2ee/java-8-stream-api-part-1), we covered what streams are and some of their most common operations.

Without further ado, let's continue with the methods used to program streams in a functional style. After that, we'll take a look at **parallel streams**.

# Map

`map()` is used to transform the value or the type of the elements of a stream:
```java
<R> Stream<R> map(Function<? super T,? extends R> mapper)
IntStream mapToInt(ToIntFunction<? super T> mapper)
LongStream mapToLong(ToLongFunction<? super T> mapper)
DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper)
```

As you can see, `map()` takes a `Function` to convert the elements of a stream of type `T` to type `R`, returning a stream of that type `R`:
```java
Stream.of('a', 'b', 'c', 'd', 'e')
    .map(c -> (int)c)
    .forEach(i -> System.out.format("%d ", i));
```

The output:
```
97 98 99 100 101
```

There are versions for transforming to primitive types. For example:
```java
IntStream.of(100, 110, 120, 130 ,140)
    .mapToDouble(i -> i/3.0)
    .forEach(i -> System.out.format("%.2f ", i));
```

Will output:
```
33.33 36.67 40.00 43.33 46.67
```

# FlatMap

`flatMap()` is used to *flatten* (or combine) the elements of a stream into one (new) stream:
```java
<R> Stream<R> flatMap(Function<? super T,
                       ? extends Stream<? extends R>> mapper)
DoubleStream flatMapToDouble(Function<? super T,
                       ? extends DoubleStream> mapper)
IntStream flatMapToInt(Function<? super T,
                       ? extends IntStream> mapper)
LongStream flatMapToLong(Function<? super T,
                       ? extends LongStream> mapper)
```

From its signature (and the signature of the primitive versions) we can see that, in contrast to `map()` which returns a single value, `flatMap()` must return a Stream. If `flatMap()` maps to `null`, the return value will be an empty stream, not `null` itself.

Let's see how this works. Suppose we have a stream comprising lists of characters:
```java
List<Character> aToD = Arrays.asList('a', 'b', 'c', 'd');
List<Character> eToG = Arrays.asList('e', 'f', 'g');
Stream<List<Character>> stream = Stream.of(aToD, eToG);
```

We want to convert all the characters to their int representation. Notice through the code below that we we can't use `map()` anymore; c represents an object of type `List<Character>`, not `Character`:

```java
stream .map(c -> (int)c)
```

Instead, we need to get the elements of the lists into one stream and then convert each character to an int. Fortunately, we have `flatMap()` to combine the list elements into a single Stream object:
```java
stream
    .flatMap(l -> l.stream())
    .map(c -> (int)c)
    .forEach(i -> System.out.format("%d ", i));
```

This outputs the following:
```
97 98 99 100 101 102 103
```
> `flatMap()` returns a stream while `map()` returns an element.

Using `peek()` (which just executes the provided expression and returns a new stream with the same elements of the original one) after `flatMap()` may clarify how the elements are processed:
```java
stream
    .flatMap(l -> l.stream())
    .peek(System.out::print)
    .map(c -> (int)c)
    .forEach(i -> System.out.format("%d ", i));
```

As you can see from the output, the stream returned from `flatMap()` is passed through the pipeline, as if we were working with a stream of single elements rather than a stream of lists of elements:
```
a97 b98 c99 d100 e101 f102 g103
```

This way, with `flatMap()` you can convert a `Stream<List<Object>>` to `Stream<Object>`. However, the important concept is that this method returns a stream and not a single element (as `map()` does).

# Reduction

A reduction is an operation that takes many elements and combines them to reduce them into a single value or object. Reduction is done by applying an operation multiple times.

Some examples of reductions include summing `N` elements, finding the maximum element of `N` numbers, or counting elements.

In the following example, we use a `for` loop to reduce an array of numbers to their sum:
```java
int[] numbers = {1, 2, 3, 4, 5, 6};
int sum = 0;
for(int n : numbers) {
    sum += n;
}
```

Of course, making reductions with streams instead of loops has benefits, such as easier parallelization and improved readability.

The Stream interface has two methods for reduction:
```java
collect()
reduce()
```

We can implement reductions with both of these methods, but `collect()` helps us implement a type of reduction called **mutable reduction**, where a container (like a `Collection`) is used to accumulate the result of the operation.

The other reduction operation, `reduce()`, has three versions:
```java
Optional<T> reduce(BinaryOperator<T> accumulator)

T reduce(T identity,
         BinaryOperator<T> accumulator)

<U> U reduce(U identity,
             BiFunction<U,? super T,U> accumulator,
             BinaryOperator<U> combiner)
```

Remember that a `BinaryOperator<T>` is equivalent to a `BiFunction<T, T, T>`, where the two arguments and the return type are all of the same types.

Let's start with the version that takes one argument. This is equivalent to:
```java
boolean elementsFound = false;
T result = null;
for (T element : stream) {
    if (!elementsFound) {
        elementsFound = true;
        result = element;
    } else {
        result = accumulator.apply(result, element);
    }
return elementsFound ? Optional.of(result)
                     : Optional.empty();
```

This code just applies a function for each element, accumulating the result and returning an `Optional` wrapping that result, or an empty `Optional` if there were no elements.

Let's see a concrete example. We just see how a sum is a reduce operation:
```java
int[] numbers = {1, 2, 3, 4, 5, 6};
int sum = 0;
for(int n : numbers) {
    sum += n;
}
```

Here, the accumulator operation is:
```java
sum += n;   //or sum = sum + n
```
This translates to:
```java
OptionalInt total = IntStream.of(1, 2, 3, 4, 5, 6)
                      .reduce( (sum, n) -> sum + n );
```

Notice how the primitive version of Stream uses the primitive version of `Optional`.

This is what happens step by step:

1. An internal variable that accumulates the result is set to the first element of a stream (1).
2. This accumulator and the second element of the stream (2) are passed as arguments to the `BinaryOperator` represented by the lambda expression `(sum, n) -> sum + x`.
3. The result (3) is assigned to the accumulator.
4. The accumulator (3) and the third element of the stream (3) are passed as arguments to the `BinaryOperator`.
5. The result (6) is assigned to the accumulator.
6. Steps 4 and 5 are repeated for the next elements of the stream until there are no more elements.

However, what if you need to have an initial value? For cases like that, we have the version that takes two arguments:
```java
T reduce(T identity, BinaryOperator<T> accumulator)
```

The first argument is the initial value, and it is called the identity because, strictly speaking, this value must be an identity for the accumulator function. In other words, for each value v, accumulator.apply(identity, v) must be equal to v.

This version of `reduce()` is equivalent to:
```java
T result = identity;
for (T element : stream)  {
    result = accumulator.apply(result, element);
} 
return result;
```

Notice that this version does not return an `Optional` object because if the stream empty, the identity value is returned.

For example, the sum example can be rewritten as:
```java
int total = IntStream.of(1, 2, 3, 4, 5, 6)
               .reduce( 0,
                       (sum, n) -> sum + n ); // 21
```

Or using a different initial value:
```java
int total = IntStream.of(1, 2, 3, 4, 5, 6)
                .reduce( 4,
                        (sum, n) -> sum + n ); // 25
```

However, notice that in the example above, the first value **cannot** be considered an identity because, for instance, `4 + 1` is not equal to `4`.

This can bring some problems when working with parallel streams, which we'll review in a few moments.

Now, notice that with these versions, you take elements of type `T` and return a reduced value of type `T` as well.

However, if you want to return a reduced value of a different type, you have to use the three arguments version of `reduce()`:
```java
<U> U reduce(U identity,
             BiFunction<U,? super T, U> accumulator,
             BinaryOperator<U> combiner)
```

This is equivalent to using: 
```java
U result = identity;
for (T element : stream) {
    result = accumulator.apply(result, element) 
}
return result;
```

Consider for example that we want to get the sum of the length of all strings of a stream, so we take strings (type `T`), and we want an integer result (type `U`).

In that case, we use `reduce()` like this:
```java
int length =
    Stream.of("Parallel", "streams", "are", "great")
        .reduce(0,
                (accumInt, str) ->
                   accumInt + str.length(), //accumulator
                (accumInt1, accumInt2) ->
                   accumInt1 + accumInt2);//combiner
```

We can make it clearer by adding the argument types:
```java
int length =
  Stream.of("Parallel", "streams", "are", "great")
      .reduce(0,
            (Integer accumInt, String str) ->
               accumInt + str.length(), //accumulator
            (Integer accumInt1, Integer accumInt2) ->
               accumInt1 + accumInt2);//combiner
```

As the accumulator function adds a mapping (transformation) step to the accumulator function, this version of the `reduce()` can be written as a combination of `map()` and the other versions of the `reduce()` method (you may know this as the [map-reduce](https://en.wikipedia.org/wiki/MapReduce) pattern):
```java
int length =
    Stream.of("Parallel", "streams", "are", "great")
        .mapToInt(s -> s.length())
        .reduce(0,
               (sum, strLength) ->
                   sum + strLength);
```

Or simply:
```java
int length = Stream.of("Parallel", "streams", "are", "great")
                .mapToInt(s -> s.length())
                .sum();
```

In fact, the calculation operations that we learned about in the [first part](http://tutorials.pluralsight.com/java-and-j2ee/java-8-stream-api-part-1) are implemented as reduce operations under the hood:
```
average
count
max
min
sum
```

Also, notice that if we return a value of the same type, the combiner function is no longer necessary (it turns out that this function is the same as the accumulator function). So, in this case, it's better to use the two argument version.

It's recommended to use the three version `reduce()` method when:

- Working with parallel streams
- Having one function as a mapper and accumulator is more efficient than having separate mapping and reduction functions.


On the other hand, `collect()` has two versions:
```java
<R,A> R collect(Collector<? super T,A,R> collector)

<R> R collect(Supplier<R> supplier,
              BiConsumer<R,? super T> accumulator,
              BiConsumer<R,R> combiner)
```

The first version uses predefined collectors from the `Collectors` class while the second one allows you to create your own collectors. Primitive streams (like `IntStream`) only have this last version of `collect()`.

Remember that `collect()` performs a mutable reduction on the elements of a stream, which means that it uses a mutable object for accumulating, like a `Collection` or a `StringBuilder`. In contrast, `reduce()` combines two elements to produce a new one and represents an immutable reduction.

However, let's start with the version that takes three arguments, as it's similar to the `reduce()` version that also takes three arguments.

As you can see from its signature, first, it takes a `Supplier` that returns the object that will be used and returned as a container (accumulator).

The second parameter is an accumulator function, which takes the container and the element to be added to it.

The third parameter is the combiner function, which merges the intermediate results into the final one (useful when working with parallel streams).

This version of collect() is equivalent to:
```java
R result = supplier.get();
for (T element : stream) {
    accumulator.accept(result, element);
} 
return result;
```

For example, if we want to "reduce" or "collect" all the elements of a stream into a `List`, use the following algorithm:
```java
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(
           () -> new ArrayList<>(),// Creating the container
           (l, i) -> l.add(i), // Adding an element
           (l1, l2) -> l1.addAll(l2) // Combining elements
        );
```

We can make it clearer by adding the argument types:
```java
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(
           () -> new ArrayList<>(),
           (l, i) -> l.add(i),
           (l1, l2) -> l1.addAll(l2)
        );
```

We can also use method references:
```java
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(
           ArrayList::new,
           ArrayList::add,
           ArrayList::addAll
        );
```

# Collectors

The previous version of `collect()` is useful to learn how collectors work, but in practice, it's better to use the other version.

Some common collectors of the `Collectors` class are:

- `toList` Accumulates elements into a `List`.
- `toSet` Accumulates elements into a `Set`.
- `toCollection` Accumulates elements into a `Collection` implementation.
- `toMap` Accumulates elements into a `Map`.
- `joining` Concatenates elements into a `String`.
- `groupingBy` Groups elements of type `T` in lists according to a classification function, into a map with keys of type `K`.
- `partitioningBy` Partitions elements of type `T` in lists according to a predicate, into a map.

Since calculation methods can be implemented as reductions, the `Collectors` class also provides them as collectors:

- `averagingInt/averagingLong/averagingDouble` Methods return the average of the input elements.
- `counting` Counts the elements of input elements.
- `maxBy` Returns the maximum element according to a given `Comparator`.
- `minBy` Returns the minimum element according to a given `Comparator`.
- `summingInt/summingLong/summingDouble` Returns the sum of the input elements.

This way, we can rewrite our previous example:
```java
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(
           ArrayList::new,
           ArrayList::add,
           ArrayList::addAll
    );
```

As:
```java
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(Collectors.toList()); // [1, 2, 3, 4, 5]
```

Since all these methods are `static`, we can use `static` imports.
```java
import static java.util.stream.Collectors.*;
...
List<Integer> list =
    Stream.of(1, 2, 3, 4, 5)
        .collect(toList()); // [1, 2, 3, 4, 5]
```

If we are working with streams of strings, we can join all the elements into one `String` with:
```java
String s = Stream.of("a", "simple", "string")
               .collect(joining()); // "asimplestring"
```

We can also pass a separator:
```java
String s = Stream.of("a", "simple", "string")
              .collect(joining(" ")); // " a simple string"
```

And a prefix and a suffix:
```java
String s = Stream.of("a", "simple", "string")
             .collect(
                 joining(" ", "This is ", ".")
             ); // "This is a simple string."
```

The calculation methods are easy to use. Except for `counting()`, they either take a `Function` to produce a value to apply the operation or (in the case of `maxBy` and `minBy`) they take a `Comparator` to produce the result:
```java
double avg = Stream.of(1, 2, 3)
                .collect(averagingInt(i -> i * 2)); // 4.0

long count = Stream.of(1, 2, 3)
                .collect(counting()); // 3

Stream.of(1, 2, 3)
    .collect(maxBy(Comparator.naturalOrder()))
    .ifPresent(System.out::println); // 3

Integer sum = Stream.of(1, 2, 3)
    .collect(summingInt(i -> i)); // 6
```

The `Collectors` class also provides two functions to group the elements of a stream into a list, in a kind of an SQL `GROUP BY` style.

The first method is `groupingBy()` and it has three versions. This is the first one:
```java
groupingBy(Function<? super T,? extends K> classifier)
```

It takes a `Function` that classifies elements of type `T`, groups them into a list and returns the result in a `Map` where the keys (of type `K`) are the `Function` returned values.

For example, if we want to group a stream of numbers by the range they belong (tens, twenties, etc.), we can do it with something like this:
```java
Map<Integer, List<Integer>> map =
   Stream.of(2, 34, 54, 23, 33, 20, 59, 11, 19, 37)
      .collect( groupingBy (i -> i/10 * 10 ) );
```

The moment you compare this code with the iterative method (with a `for` loop), you realize the power of streams and `collect()`. Just look at how many lines of code are used in the traditional implementation.

```java
List<Integer> stream =
    Arrays.asList(2,34,54,23,33,20,59,11,19,37);
Map<Integer, List<Integer>> map = new HashMap<>();

for(Integer i : stream) {
    int key = i/10 * 10;
    List<Integer> list = map.get(key);

    if(list == null) {
        list = new ArrayList<>();
        map.put(key, list);
    }
    list.add(i);
}
```

In the end, both strategies return the same map.
```
{0=[2], 50=[54,59], 20=[23,20], 10=[11,19], 30=[34,33,37]}
```

## Downstream Collectors

As you may have noticed, the second version takes a *downstream collector* as an additional argument:
```java
groupingBy(Function<? super T,? extends K> classifier,
           Collector<? super T,A,D> downstream)
```

A downstream collector is a collector that is applied to the results of another collector.

We can use any collector here, for instance, to count the elements in each group of the previous example:
```
Map<Integer, Long> map =
    Stream.of(2, 34, 54, 23, 33, 20, 59, 11, 19, 37)
        .collect(
            groupingBy(i -> i/10 * 10,
                       counting()
            )
        );
```

Notice how the type of the values of the `Map` change to reflect the type returned by the downstream collector, `counting()`.

This will return the following map:
```java
{0=1, 50=2, 20=2, 10=2, 30=3}
```

We can even use another `groupingBy()` to classify the elements in a second level. For instance, instead of counting, we can further classify the elements as even or odd:
```java
Map<Integer, Map<String, List<Integer>>> map =
   Stream.of(2,34,54,23,33,20,59,11,19,37)
       .collect(groupingBy(i -> i/10 * 10,
                  groupingBy(i ->
                               i%2 == 0 ? "EVEN" : "ODD")
                )
       );
```

This will return the following map (with a little formatting):
```java
{
    0 = {EVEN=[2]},
   50 = {EVEN=[54], ODD=[59]},
   20 = {EVEN=[20], ODD=[23]},
   10 = {ODD=[11, 19]},
   30 = {EVEN=[34], ODD=[33, 37]}
}
```

The key of the high-level map is an `Integer` because the first `groupingBy()` returns a one.

The type of the values of the high-level map changed (again) to reflect the type returned by the downstream collector, `groupingBy()`.

In this case, a `String` is returned; this will be the type of the keys of the second-level map, and since we are working with an `Integer` Stream, the values have a type of `List<Integer>`.

Seeing the output of these examples, you may be wondering, is there a way to order the results?

Well, `TreeMap` is the only implementation of `Map` that is ordered. Fortunately, the third version of `groupingBy()` has a `Supplier` argument that lets us choose the type of the resulting `Map`:
```java
groupingBy(Function<? super T,? extends K> classifier,
           Supplier<M> mapFactory,
           Collector<? super T,A,D> downstream)
```

This way, if we pass an instance of TreeMap:
```java
Map<Integer, Map<String, List<Integer>>> map =
    Stream.of(2,34,54,23,33,20,59,11,19,37)
       .collect( groupingBy(i -> i/10 * 10,
                 TreeMap::new,
                 groupingBy(i -> i%2 == 0 ? "EVEN" : "ODD")
               )
       );
```

This will return the following map:
```java
{
    0 = {EVEN=[2]},
   10 = {ODD=[11, 19]},
   20 = {EVEN=[20], ODD=[23]},
   30 = {EVEN=[34], ODD=[33, 37]},
   50 = {EVEN=[54], ODD=[59]}
}
```
### partitioningBy()
The second method for grouping is `partitioningBy()`.

The difference with `groupingBy()` is that `partitioningBy()` will return a `Map` with a `Boolean` as the key type, which means there are only two groups, one for `true` and one for `false`.

There are two versions of this method. The first one is:
```java
partitioningBy(Predicate<? super T> predicate)
```

It partitions the elements according to a `Predicate` and organizes them into a `Map<Boolean, List<T>>`.

For example, if we want to partition a stream of numbers by the ones that are less than `50` and the ones that don't, we can do it this way:
```java
Map<Boolean, List<Integer>> map =
    Stream.of(45, 9, 65, 77, 12, 89, 31)
        .collect(partitioningBy(i -> i < 50));
```

This will return the following map:
```java
{false=[65, 77, 89], true=[45, 9, 12, 31, 12]}
```

As you can see, because of the `Predicate`, the map will always have two elements.

And like `groupingBy()`, this method has a second version that takes a downstream collector.

For example, if we want to remove duplicates, we just have to collect the elements into a `Set` like this:
```java
Map<Boolean, Set<Integer>> map =
    Stream.of(45, 9, 65, 77, 12, 89, 31, 12)
        .collect(
            partitioningBy(i -> i < 50,
                           toSet()
            )
        );
```

This will produce the following `Map`:
```java
{false=[65, 89, 77], true=[9, 12, 45, 31]}
```

However, unlike `groupingBy()`, there's no version that allows us to change the type of the `Map` returned. However, we only need two keys for our groups.
```java
Set<Integer> lessThan50 = map.get(true);
Set<Integer> moreThan50 = map.get(false);
```

# Parallel Streams

Until now, all the examples have used sequential streams, where each element are processed one by one.

> Parallel streams split the stream into multiple parts. Each part is processed by a different thread at the same time (in parallel).

Under the hood, parallel streams use the [Fork/Join Framework](http://ocpj8.javastudyguide.com/ch28.html).

This means that, by default, the number of threads available to process parallel streams equals the number of available cores in your machine's processor (CPU).

### Parallel stream operations

To create a parallel stream just use the `parallel()` method:
```java
Stream<String> parallelStream =
                   Stream.of("a","b","c").parallel();
```

To create a parallel stream from a `Collection` use the `parallelStream()` method:
```java
List<String> list = Arrays.asList("a","b","c");
Stream<String> parStream = list.parallelStream();
```

You can turn a parallel stream into a sequential one with the `sequential()` method:
```
stream
   .parallel()
      .filter(..)
         .sequential()
            .forEach(...);
```

Check if a stream is parallel with `isParallel()`:
```java
stream.parallel().isParallel(); // true
```

And turn an ordered stream into a unordered one (or ensure that the stream is unordered) with `unordered()`;
```java
stream
   .parallel()
      .unordered()
         .collect(...);
```

### Under the hood

But how do parallel streams work? Let's start with the simplest example:
```java
Stream.of("a","b","c","d","e")
    .forEach(System.out::print);
```

Printing a list of elements with a sequential stream will output the expected result:
```
abcde
```

However, when using a parallel stream:
```java
Stream.of("a","b","c","d","e")
    .parallel()
    .forEach(System.out::print);
```

The output can be different for each execution:
```
cbade // One execution
cebad // Another execution
cbdea // Yet another execution
```

Going back to the definition of parallel streams, this output starts making sense. The differences in output can be attributed to thread processing; it is possible that a different core is involved with a particular command each time the code is executed. 

Thus parallel streams are more appropriate for operations where the order of processing doesn't matter and the operations don't need to keep a state (stateless and independent operatoins).

An example to see this difference is the use of `findFirst()` versus `findAny()`.

In the [first part](http://tutorials.pluralsight.com/java-and-j2ee/java-8-stream-api-part-1), we mentioned that `findFirst()` method returns the first element of a stream. But since we're using parallel streams, this method has to "know" which element is the first one:
```java
long start = System.nanoTime();
String first = Stream.of("a","b","c","d","e")
    .parallel().findFirst().get();
long duration = (System.nanoTime() - start) / 1000000;
System.out.println(
    first + " found in " + duration + " milliseconds");
```

The output:
```java
a found in 2.436155 milliseconds
```

Because of that, if the order doesn't matter, it's better to use `findAny()` with parallel streams:
```java
long start = System.nanoTime();
String any = Stream.of("a","b","c","d","e")
    .parallel().findAny().get();
long duration = (System.nanoTime() - start) / 1000000;
System.out.println(
    any + " found in " + duration + " milliseconds");
```

The output:
```java
c found in 0.063169 milliseconds
```

Since a parallel stream is processed by multiple cores, its reasonable to believe that it will be processed faster than a sequential stream. But as you can see with `findFirst()`, this is not always the case.

For example:
```java
Stream<T> distinct()
Stream<T> sorted()
Stream<T> sorted(Comparator<? super T> comparator)
Stream<T> limit(long maxSize)
Stream<T> skip(long n)
```

The stateful operations above incorporate state from previously processed elements and usually need to go through the entire stream to produce a result. Thus they work better with sequential streams since they end up looking through the stream anyway.

But don't believe that by first executing the stateful operations in a sequential format and then turning the stream into a parallel one, the performance will be better in all cases. It would be worse to assume that the entire operation may run in parallel, like the following example:
```java
double start = System.nanoTime();
Stream.of("b","d","a","c","e")
    .sorted()
    .filter(s -> {
        System.out.println("Filter:" + s);
        return !"d".equals(s);
    })
    .parallel()
    .map(s -> {
        System.out.println("Map:" + s);
        return s += s;
    })
    .forEach(System.out::println);
double duration = (System.nanoTime() - start) / 1_000_000;
System.out.println(duration + " milliseconds");
```

One might think that the stream is sorted and filtered sequentially, but the output shows something else:
```java
Filter:c
Map:c
cc
Filter:a
Map:a
aa
Filter:b
Map:b
bb
Filter:d
Filter:e
Map:e
ee
79.470779 milliseconds
```

Compare this with the output of the sequential version (just comment out `.parallel()`):
```java
Filter:a
Map:a
aa
Filter:b
Map:b
bb
Filter:c
Map:c
cc
Filter:d
Filter:e
Map:e
ee
1.554562 milliseconds
```

Clearly, the sequential version performed better; it took 78 milliseconds less.

But if we have an independent or stateless operation, and order doesn't matter, such as with counting the number of odd numbers in a large range, the parallel version will perform better:
```java
double start = System.nanoTime();
long c = IntStream.rangeClosed(0, 1_000_000_000)
            .parallel()
            .filter(i -> i % 2 == 0)
            .count();
double duration = (System.nanoTime() - start) / 1_000_000;
System.out.println("Got " + c + " in " + duration + " milliseconds");
```

The parallel version output:
```java
Got 500000001 in 738.678448 milliseconds
```

The sequential version output:
```java
Got 500000001 in 1275.271882 milliseconds
```
In summary, parallel streams don't always perform better than sequential streams when it comes to stateful operations, but they usually perform better when ordering is not an issue and operations are independent and stateless.

This, the fact that parallel streams process results independently, and the idea that the order cannot be guaranteed are the most important things you need to know.

### Tips for deciding between sequential and parallel streams

In practice, how do you know when to use sequential or parallel streams for better performance?

Here are some rules:

- For a small set of data, sequential streams are almost always the best choice due to the overhead of the parallelism. Using parallel streams is simply unnecessary.
- Typically avoid using parallel streams with stateful (like `sorted()`) and order-based (like `findFirst()`) operations. Sequential streams do just fine (if not better) in these cases.
- Use parallel streams with operations that are computationally expensive (considering all the operation in the pipeline). 
- When in doubt, check the performance with an appropriate benchmark. To demonstrate, I used an execution time comparison, but this is just one benchmark. You may need your program to compile faster or use less memory. 

# Reducing Parallel Streams

In concurrent environments, assignments are bad.

This is because if you mutate the state of variables (especially if they are shared by more than one thread), you may run into invalid states.

Consider this example, which implements the factorial of `10` in a very particular way:
```java
class Total {
    public long total = 1;
    public void multiply(long n) { total *= n; }
}
...
Total t = new Total();
LongStream.rangeClosed(1, 10)
    .forEach(t::multiply);
System.out.println(t.total);
```

Here, we are using a variable to gather the result of the factorial. The output of executing this snippet of code is:
```java
3628800
```

However, when we turn the stream into a parallel one:
```java
LongStream.rangeClosed(1, 10)
    .parallel()
    .forEach(t::multiply);
```

Sometimes we get the correct result and other times we don't.

The problem is caused by the multiple threads accessing the variable total concurrently. Yes, we can synchronize the access to this variable, but that defeats the purpose of parallelism.

Here's where `reduce()` comes in handy.

Remember that `reduce()` combines the elements of a stream into a single one.

With parallel streams, this method creates intermediate values and then combines them, avoiding the *ordering* problem while still allowing streams to be processed in parallel by eliminating the shared state and keeping it inside the reduction process.

The only requirement is that the applied reducing operation must be associative.

This means that the operation op must follow this equality:
```
(a op b) op c == a op (b op c)
```

Or:
```
a op b op c op d == (a op b) op (c op d)
```

So we can evaluate (a op b) and (c op d) in parallel.

We can implement our example using `parallel()` and `reduce()` in this way:
```java
long tot = LongStream.rangeClosed(1, 10)
                .parallel()
                .reduce(1, (a,b) -> a*b);
System.out.println(tot);
```

When we execute this snippet of code, it produces the correct result every time (`3628800`). Reduce guaranteed that the threads would not access the same stream entries simultaneously and throw off the results.

Plus, if we time the execution of the first snippet and this last one, we can see a drastic improvement in performance.

We can safely use `collect()` with parallel streams if we follow the same requirements of associativity and identity. (For example, combining any partially accumulated result with an empty result container must produce an equivalent result.)

Or, if we are grouping with the `Collectors` class and ordering is not important, we can use the method `groupingByConcurrent()`, the concurrent version of `groupingBy()`.

If you understand when to use parallel streams and the issues associated with concurrent execution, you should be ready to use parallel streams in practice!

# Conclusion

We touched the most important parts of the Stream interface. I hope you find streams useful. Please post all your comments and feedback in the discussion section below. Thanks for reading.