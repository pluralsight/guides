[code lang="python"]
>>> x = range(50)
>>> x < 20
[/code]

What's the value of `x < 20`?

I'll kill the suspense for you, it is `False`.

I know, it didn't make sense to me either.  I don't know what comparing
integers to lists means semantically, but I would like to know why the answer
is `False`.  Python must have some reason for this being `False`, and I figured
it would be interesting to look into.

I thought the answer could potentially be relevant for my upcoming talk.  After
all, the implementation of the `<` operator is handled by the
[less-than dunder method](http://docs.python.org/reference/datamodel.html#object.__lt__)
for the [list](http://docs.python.org/library/types.html#types.ListType) type.

###Assumptions

1. My first thought was the `<` operator was comparing with `len()` for the
list, which would make `x < 51` `True`.  Result: **wrong**.

2. OK, then I figured it was using the list's
[id](http://docs.python.org/library/functions.html#id).  This would **sort of**
make sense.  Result: **wrong** again.

[code lang="python"]
>>> id(x)
170461900
>>> x < 170461901
False
[/code]

###To the source!

At this point the sheer mystery drove me to download the
[Python 2.7](http://python.org/download/releases/2.7.3/) source.
I could have scoured the Internet for a quick solution, but I haven't spent
much time digging in the
[Python 2.7](http://python.org/download/releases/2.7.3/) source so now seemed
like a good time to start.

I started my search by looking up the implementation for '<' in the
[list object](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/listobject.c#l991).
Now the fun of tracing
[C](http://en.wikipedia.org/wiki/C_(programming_language)) function pointers
and macros begins...

I'll save you the time and just provide a summary of what I found. The
comparison operation ends in the
[default_3way_compare](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/object.c#l757)
function.  Finally, the following snippet of code decides that our list is
always greater than (not less than) all integers:

[code lang="c"]
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
[/code]

Surprised?  Confused?  I was both.  

###TL;DR

The end result is that comparison is based on the actual name of the types!
So, in our case **List** is greater alphabetically than **Integer**!

Not exactly the rationale I was hoping for, but at least we have an answer.
However, now that we know what is happening see if you can figure out the
answer to a few more puzzles:

[code lang="python"]
>>> [] < ()
>>> [] < {}
[/code]

**Spoiler alert**

A **List** is less than a **Tuple**, remember 'L' comes before 'T' in the
alphabet, and of course by the same logic a **List** is NOT less than a
**Dict**.

###Easter Egg

`None` is **really**
[small in comparison](http://hg.python.org/releasing/2.7.3/file/7bb96963d067/Objects/object.c#l773):

[code lang="c"]
/* None is smaller than anything */
if (v == Py_None)
    return -1;
if (w == Py_None)
    return 1;
[/code]

###Python 3

This is a bit weird, but wait there's more!  What does
[Python 3](http://www.python.org/getit/releases/3.0/) do in this case?

[code lang="python"]
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
[/code]

Nice!  This little quirk has been fixed.  Now when you upgrade your awesome
application to [Python 3](http://www.python.org/getit/releases/3.0/) comparing
completely different types will complain loudly as it should. Then, you will
be free to forget everything you learned in this post. :)

###Suggested Reading

If your interested I later found a great
[stackoverflow post](http://stackoverflow.com/questions/7167657/python-list-greater-than-number)
confirming all of the above research.  You can also look into the Python
[documentation](http://docs.python.org/library/stdtypes.html#comparisons) to
find the same answer.

It's also worth nothing this is an implementation choice by
[CPython](http://en.wikipedia.org/wiki/CPython).  Your results with other
implementations of Python like [pypy](http://pypy.org/) could be different.
