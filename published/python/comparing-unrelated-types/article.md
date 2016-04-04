```python
>>> x = range(50)
>>> x < 20
```

What's the value of `x < 20`?

I'll kill the suspense for you, it is `False`.

I know, it didn't make sense to me either.  I don't know what comparing
Integers to Lists means semantically, but I would like to know why Python translates this comparison to `False`. 

I thought the answer could be relevant for my upcoming talk.  After
all, the implementation of the `<` operator is handled by the
[less-than dunder method](http://docs.python.org/reference/datamodel.html#object.__lt__)
for the [list](http://docs.python.org/library/types.html#types.ListType) type.

##Assumptions

1. My first thought was that the `<` operator was comparing with `len()` for the
list, which would make `x < 51` `True`.  Result: **wrong**.

2. Maybe Python was using the list's
[id](http://docs.python.org/library/functions.html#id).  This would _sort of_
make sense, since each object has a unique Long id -- an ideal comparison tool.  Result: **wrong**.

```python
>>> id(x)
170461900
>>> x < 170461901
False
```

##To the source!

At this point, the mystery drove me to download the [Python 2.7](http://python.org/download/releases/2.7.3/) source.

I could have scoured the Internet for a quick solution, but I wanted to spend a little time digging through the 
[Python 2.7](http://python.org/download/releases/2.7.3/) source.

I started my search by looking up the implementation for '<' in the
[list object](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/listobject.c#l991).
That's when the fun of tracing
[C](http://en.wikipedia.org/wiki/C_(programming_language)) function pointers
and macros began...

I'll save you the time and just provide a summary of what I found. The
comparison operation ends in the
[default_3way_compare](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/object.c#l757)
function.  

The following snippet of code dictates the outcome of all list-integer comparisons by deciding that our list is always greater than (not less than) all integers:

```c
/* different type: compare type names; numbers are smaller */
if (PyNumber_Check(v))
    vname = "";
else
    vname = v->ob_type->tp_name;
if (PyNumber_Check(w))
    wname = "";
else
    wname = w->ob_type->tp_name;
c = strcmp(vname, wname);
if (c < 0)
    return -1;
if (c > 0)
    return 1;
```

Surprised?  Confused?  I was both.  

##Surprise!

After some more research, I found that the comparison is based on the actual name of the types!
So, in our case **List** is greater alphabetically than **Integer**! This pattern persists in most type comparisons.

Not exactly the rationale I was hoping for, but at least we have an answer.

However, now that we know what is happening see if you can figure out the
answer to a few more puzzles:

```python
>>> [] < ()
>>> [] < {}
```

**Spoiler alert**

A **List** is less than a **Tuple**, remember 'L' comes before 'T' in the
alphabet, and of course by the same logic a **List** is NOT less than a
**Dict**.

##Easter Egg

`None` is **really**
[small in comparison](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/object.c#l773):

```c
/* None is smaller than anything */
if (v == Py_None)
    return -1;
if (w == Py_None)
    return 1;
```

##Python 3

Until now, I was using Python 2.7. I wanted to know what 
[Python 3](http://www.python.org/getit/releases/3.0/) would do in this case.

```python
>>> x = range(50)
>>> x < 20
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: unorderable types: range() < int()
>>> [] < ()
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: unorderable types: list() < tuple()
>>> [] < {}
Traceback (most recent call last):
File "<stdin>", line 1, in <module>
TypeError: unorderable types: list() < dict()
```

Nice!  As you see above, unrelated type comparisons now lead to TypeErrors, a result that's more comprehensible than alphabetical order. So this little quirk is fixed. 

So when you upgrade your awesome
application to [Python 3](http://www.python.org/getit/releases/3.0/) comparing
completely different types will cause the interpreter to complain loudly, and it should. Then, you are free to forget everything you learned in this post. :)

##Suggested Reading

If you are interested, I found a great
[stackoverflow post](http://stackoverflow.com/questions/7167657/python-list-greater-than-number)
confirming all of the above research after I finished with my digging.  You can also look into the Python
[documentation](http://docs.python.org/library/stdtypes.html#comparisons) to
find the same answer. Nonetheless, taking the chance to look through Python source code proved to be an enlightening experience.

A side note: this implementation is an implementation choice by
[CPython](http://en.wikipedia.org/wiki/CPython).  Your results with other
implementations of Python like [pypy](http://pypy.org/) could be different. If you have any other interesting observations from using Python, feel free to describe them in the comments section!
