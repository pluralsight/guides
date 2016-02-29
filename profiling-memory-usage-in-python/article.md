One of the ways [Python](http://python.org) makes development fast and easier
than languages like [C](http://en.wikipedia.org/wiki/C_(programming_language))
and [C++](http://en.wikipedia.org/wiki/C%2B%2B) is memory management.  In
Python it's simple, the language handles memory management for you.  However,
this doesn't mean memory should be forgotten.  Good developers will want
to track the memory usage of their application and look to lower memory usage.
This post will explain common tools for doing this.

### Profiling size of individual objects

The lowest layer of memory profiling involves looking at a single object in
memory.  You can do this by opening up a shell and doing something like the
following:

```python
    >>> import sys
    >>> sys.getsizeof({})
    136
    >>> sys.getsizeof([])
    32
    >>> sys.getsizeof(set())
    112
```

The above snippet illustrates the overhead associated with a
[list](http://docs.python.org/2/tutorial/introduction.html#lists) object. A
list is 32 bytes (on a 32-bit machine running Python 2.7.3).  This style of
profiling is useful when determining what type of [data
type](http://docs.python.org/2/tutorial/datastructures.html) to use.

### Profiling a single function or method

The easiest way to profile a single method or function is the open source
[memory-profiler](http://pypi.python.org/pypi/memory_profiler) package.  It's
similar to [line_profiler](http://pythonhosted.org/line_profiler/) which I've
[written about before](http://codrspace.com/durden/quick-profiling-in-python/).

You can use it by putting the `@profile` decorator around any function or
method and running `python -m memory_profiler myscript`.  You'll see
line-by-line memory usage once your script exits.

This is extremely useful if you're wanting to profile a section of
memory-intensive code, but it won't help much if you have no idea where the
biggest memory usage is.  In that case, a higher-level approach of profiling is
needed first.

### Profiling an entire application

There are a number of ways to profile an entire Python application.  You can
use the standard [unix](http://en.wikipedia.org/wiki/Unix) tools
[top](http://unixhelp.ed.ac.uk/CGI/man-cgi?top+1) and
[ps](http://unixhelp.ed.ac.uk/CGI/man-cgi?ps).  A more Python specific way is
[guppy](http://pypi.python.org/pypi/guppy/).

#### Guppy
<br>

To use [guppy](http://pypi.python.org/pypi/guppy/) you drop something like the
following in your code:

```python
    from guppy import hpy
    h = hpy()
    print h.heap()
```

This will print you a nice table of usage grouped by object type.  Here's an
example of an [PyQt4](http://www.riverbankcomputing.com/software/pyqt/intro)
application I've been working on:

    Partition of a set of 235760 objects. Total size = 19909080 bytes.
    Index  Count   %     Size   % Cumulative  % Kind (class / dict of class)
        0  97264  41  8370996  42   8370996  42 str
        1  47430  20  1916788  10  10287784  52 tuple
        2    937   0  1106440   6  11394224  57 dict of PyQt4.QtCore.pyqtWrapperType
        3    646   0  1033648   5  12427872  62 dict of module
        4  11683   5   841176   4  13269048  67 types.CodeType
        5  11684   5   654304   3  13923352  70 function
        6   1200   1   583872   3  14507224  73 dict of type
        7    782   0   566768   3  15073992  76 dict (no owner)
        8   1201   1   536512   3  15610504  78 type
        9   1019   0   499124   3  16109628  81 unicode

This type of profiling can be difficult if you have a large application using a
relatively small number of object types.

#### mprof
<br>

Finally, I recently discovered
[memory-profiler](http://pypi.python.org/pypi/memory_profiler) comes with a
script called mprof, which can show you memory usage over the lifetime of your
application. This can be useful if you want to see if your memory is getting
cleaned up and released periodically.

Using mprof is easy, just run `mprof run script script_args` in your shell
of choice.  mprof will automatically create a graph of your script's memory
usage over time, which you can view by running `mprof plot`. Be aware that
plotting requires [matplotlib](http://matplotlib.org).

I'm sure there are other approaches to profiling memory usage in Python.  So
let me know your recommendations in the comments.
