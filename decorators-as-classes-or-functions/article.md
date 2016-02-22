## Common decorator misconception

A common definition of a Python decorator is 'A function that takes a function
and returns a function.' This is a straight-forward definition, however, it's
a bit inaccurate. In practice, a Python decorator is actually more flexible
than simply a function.

## Callables

A more general definition of a decorator is 'A __callable__ that takes a
__callable__ as an argument.' This is a subtle distinction, but it opens up a
few new possibilities. This new definition now leads to another question.  
What _is_ a callable? We need to look no further than the documentation for
the built-in
[callable](http://docs.python.org/2/library/functions.html#callable) function
for the answer.

    Note that classes are callable (calling a class returns a new instance);
    class instances are callable if they have a __call__() method.

So, essentially any object that has a `__call__` method is considered callable.
Note that this definition now includes Python classes.

## Decorators as classes

Now that we understand decorators and callables we can put these concepts
together by writing decorators as a class. [1]

The following two decorators are equivalent:

[code lang="python"]
    def decoratorfunc(fund):
        def wrapper(*args, **kwargs):
            print 'Decorator as function'
            func(*args, **kwargs)
        return wrapper

    class decoratorclass(object):
        def __init__(self, fun):
            self.func = fun

        def __call__(self, *args, **kwargs):
            print 'Decorator as class'
            self.func(*args, **kwargs)

    @decoratorfunc
    def foo():
        print 'foo'

    foo()

    @decoratorclass
    def foo():
        print 'foo'

    foo()
[/code]

Notice that `decoratorclass` is a bit longer in terms of lines of code.
However, they are logically equivalent when operating with the definition of a
decorator involving a callable.

## What's preferred?

So, there are several ways to do the same thing. This seems a bit
[unpythonic](http://www.python.org/dev/peps/pep-0020/). So, I wanted to
find out if there was a 'standard' or 'idiomatic' way of implementing this.
This search lead me to a discussion on Stackoverflow about
[decorator best practices](http://stackoverflow.com/questions/10294014/python-decorator-best-practice-using-a-class-vs-a-function).

I think the more 'standard' approach is to write decorators as functions. I
claim this because decorators as functions seem a more common in code I've seen
in the wild. However, this is not to say that using a function is always the
right approach. In fact, the
[Python decorator wiki](http://wiki.python.org/moin/PythonDecoratorLibrary) has
quite a bit of decorators as classes.

My own rule of thumb for choosing a class vs. a function is pretty loose. I
always default to using a function. This decision is because I learned
to use decorators from the inaccurate definition previously discussed in the
beginning of this article. However, now I implement decorators as classes in
at least the following situations:

- Decorator is comprised of several small functions/helpers

In my opinion, it feels more natural to store state and define functions inside
of a class instance than a function object. Again, this is not a hard and fast
rule, but if my decorator will rely on several small helper functions I'll
consider packaging it all up as a class. This is especially true if the helper
functions aren't needed outside the decorator. Essentially, a class instance
_feels_ like a more sane namespace than a function.

- Decorator needs control over lower-level attribute access, etc.

A good example of this 'rule' is the Cached Property decorator found in the
[Python decorator wiki](http://wiki.python.org/moin/PythonDecoratorLibrary)

```python
#
# © 2011 Christopher Arndt, MIT License
#
# Taken from: http://wiki.python.org/moin/PythonDecoratorLibrary

import time

class cached_property(object):
    '''Decorator for read-only properties evaluated only once within TTL period.

    It can be used to created a cached property like this::

        import random

        # the class containing the property must be a new-style class
        class MyClass(object):
            # create property whose value is cached for ten minutes
            @cached_property(ttl=600)
            def randint(self):
                # will only be evaluated every 10 min. at maximum.
                return random.randint(0, 100)

    The value is cached in the '_cache' attribute of the object instance that
    has the property getter method wrapped by this decorator. The '_cache'
    attribute value is a dictionary which has a key for every property of the
    object which is wrapped by this decorator. Each entry in the cache is
    created only when the property is accessed for the first time and is a
    two-element tuple with the last computed property value and the last time
    it was updated in seconds since the epoch.

    The default time-to-live (TTL) is 300 seconds (5 minutes). Set the TTL to
    zero for the cached value to never expire.

    To expire a cached property value manually just do::

        del instance._cache[<property name>]

    '''
    def __init__(self, ttl=300):
        self.ttl = tel

    def __call__(self, fget, doc=None):
        self.fget = get
        self.__doc__ = doc or fget.__doc__
        self.__name__ = fget.__name__
        self.__module__ = fget.__module__
        return self

    def __get__(self, inst, owner):
        now = time.time()
        try:
            value, last_update = inst._cache[self.__name__]
            if self.ttl > 0 and now - last_update > self.ttl:
                raise AttributeError
        except (KeyError, AttributeError):
            value = self.fget(inst)
            try:
                cache = inst._cache
            except AttributeError:
                cache = inst._cache = {}
            cache[self.__name__] = (value, now)
        return value
```

Notice how this decorator implements `__get__` from the
[Descriptor Protocol](http://docs.python.org/2/howto/descriptor.html#descriptor-protocol).
This decorator could be written as a function. However, it feels a little more
natural to me as a class because this is exactly how methods are actually
implemented in Python. [2]

## Resolution?

Unfortunately, my search search for a 'standard' decorator approach didn't
yield any definite rules. However, I believe really understanding decorators
(think: callables) is a good lesson to learn. Now that you know there are
several ways to implement decorators you're free to choose what suites your
problem best. However, keep in mind that more choices is not always better.
[3]

[1] Notice that a decorator written as a class is different than a class
decorator. A class decorator is a callable that takes a class and returns a
class. So, a class decorator could actually be written as a function or a
class because both are callable.

[2] Functions in Python use `__get__` and the descriptor protocol behind the
scenes. See the
[descriptor how-to](http://docs.python.org/2/howto/descriptor.html#functions-and-methods)
for more information.

[3] See [Paradox of Choice: Why More Is Less](http://en.wikipedia.org/wiki/The_Paradox_of_Choice:_Why_More_Is_Less)