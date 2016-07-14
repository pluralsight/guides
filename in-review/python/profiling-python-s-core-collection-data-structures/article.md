This guide discusses the basic fundamentals related to [Python](http://python.org) collection 'type' data structures and the trade offs associated with them.

The trade offs typically boil down to [space-time tradeoff](http://en.wikipedia.org/wiki/Space%E2%80%93time_tradeoff).  The idea is simple; You can reduce computation time if you increase the memory usage or vice-versa.  This tariff is common in all programming languages, but I'll narrow the scope of this discussion to just core [Python](http://python.org) data structures.

So, what are the core collection types in [Python](http://python.org), and what are they good at?

## 1. Lists

[Lists](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists) are typically the first type of collection developers encounter.  Their strengths are:

- Storing heterogeneous objects easily
- Low memory footprint

## 2. Sets

[Sets](http://docs.python.org/2/tutorial/datastructures.html#sets) are very similar to [lists](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists) except for a few fundamental differences:

1. Duplicate entries are now allowed
2. Entries are unordered

## 3. Dicts

[Dicts](http://docs.python.org/2/tutorial/datastructures.html#dictionaries) are the highest-level and most complicated of the core collections.  A [dict](http://docs.python.org/2/tutorial/datastructures.html#dictionaries) allows for users to map a key value (must be unique) to another item, typically referred to as the 'value.'

[Dicts](http://docs.python.org/2/tutorial/datastructures.html#dictionaries) are by unordered by default, but there are [ways around that](http://docs.python.org/2/library/collections.html#collections.OrderedDict).

### Trade-offs

Now we have a high-level understanding of what each collection can do for us, so what are the the memory/time trade-offs of each?

The differences between lists and sets introduce a few trade-offs:

1. Set insertion is slower because each insertion must check for a duplicate.
2. The size of [sets](http://docs.python.org/2/tutorial/datastructures.html#sets) is larger, mostly due to metadata needed to detect duplicates quickly.
3. Sets will decrease potential memory usage if you're storing duplicate data.

### 1. Memory size *

```python
    >>> import sys
    >>> sys.getsizeof({})
    136
    >>> sys.getsizeof([])
    32
    >>> sys.getsizeof(set())
    112
```

Notice the [dict](http://docs.python.org/2/tutorial/datastructures.html#dictionaries) is considerably larger than the [list](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists).  We can refer to this as the memory overhead of dicts vs lists.  This is only a few bytes, which doesn't seem like a lot.  However, this can become a big issue if your application maintains thousands or millions of dicts.

Also notice that the [set](http://docs.python.org/2/tutorial/datastructures.html#sets) is also significantly larger than the [list](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists).  This, like the dict, is due to the extra internal tracking needed to verify uniqueness of elements, etc.

There isn't much else to the memory size trade-off.  Just keep in mind a [list](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists) is a good collection type to start of with for memory purposes.  However, the decision to use a list depends heavily on the type of computation your algorithm requires.

### 2. Computation time

This is where things get a bit fuzzy and start to depend heavily on the specific requirements of your application.  You'll need to decide what operations you should optimize for, i.e the operations that occur most frequently.

Optimizing algorithms is another [science](http://en.wikipedia.org/wiki/Mathematical_optimization) all by itself, so let's  narrow the algorithm discussion and just use a single example, determining if an entry is in our collection or not.

```python
    >>> import timeit
    >>> # Timing lookup for a list
    >>> t = timeit.Timer('100000 in x', 
                    setup='x = [ii for ii in xrange(100000)]')
    >>> sum(t.repeat(3, 1)) / 3.0
    0.0018588701883951824
    >>> # Timing lookup for a set
    >>> t = timeit.Timer('100000 in x', 
                         setup='x = {ii for ii in xrange(100000)}')
    >>> sum(t.repeat(3, 1)) / 3.0
    3.337860107421875e-06
    >>> # Timing lookup for a dict
    >>> t = timeit.Timer('100000 in x', 
                         setup='x = {ii:ii for ii in xrange(100000)}')
    >>> sum(t.repeat(3, 1)) / 3.0
    2.384185791015625e-06
```

Notice how much slower the list lookup is.  So, a [list](http://docs.python.org/2/tutorial/datastructures.html#more-on-lists) is a poor choice for any application where you will be doing lots of lookup operations on a relatively large collection.

Please note that this particular metric might not be useful for your application.  For example, if your application is inserting elements very common?  If so, you'll want to choose a type collection that is very fast at insertion and finding an element might be less important.

Unfortunately, the decision is usually not as simple as finding what is fastest.  Remember the [space-time tradeoff](http://en.wikipedia.org/wiki/Space%E2%80%93time_tradeoff)?

This principle applies directly here.  What kind of space/memory trade-off would we be making to get this increased lookup speed?

```python
    >>> x = [ii for ii in xrange(100000)]
    >>> sys.getsizeof(x) / 1024
    402
    >>> x = {ii for ii in xrange(100000)}
    >>> sys.getsizeof(x) / 1024
    2048
    >>> x = {ii:ii for ii in xrange(100000)}
    >>> sys.getsizeof(x) / 1024
    3072
```

As you can see, we gave up a relatively large amount of memory to get the improved lookup speeds of the set and dict.

A set is the best of both worlds in this situation.  It is smaller in size than a dict while having a much faster lookup time than a list.  However, a set is also larger in size than a list and slower in lookup time than a dict.  So, a set is a good choice here if we are looking for a decent lookup time and a lower memory footprint.

* All data for memory usage taken from 32-bit Python 2.7.3
