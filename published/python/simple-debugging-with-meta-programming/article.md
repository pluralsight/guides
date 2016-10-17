The 'watch' feature of modern debuggers allows a developer to see when a
variable changes state. I use this feature a lot when doing small debugging sprints. Unfortunately, using `watch` requires opening up a big
[IDE](http://en.wikipedia.org/wiki/Integrated_development_environment) like [PyCharm](http://www.jetbrains.com/pycharm/). Since I prefer to work primarily with the [command line](http://en.wikipedia.org/wiki/Command-line_interface)
and [vim](http://www.vim.org/about.php), this process was tedious and seemed unnecessary.

So I decided to write my own debugging 'utility' to handle very simple approaches to this 'watching' debugging work flow.

# Requirements

1. Easy to use
2. Track several attributes at once
3. Display when variable changes and what its' new value is
4. Display stack trace/location where watched variable changes

# Attribute setting

The first step is to track when a variable is set. Luckily, [Python](http://python.org) provides a [`__setattr__`](http://docs.python.org/2/reference/datamodel.html#object.__setattr__) solely for this purpose [1]. The `__setattr__` [method](http://docs.python.org/2/reference/datamodel.html#object.__setattr__) is called each time an attribute is set, just before the actual assignment occurs.

# Watching specific attributes

Now we need a list of what attributes to watch. Then, we can provide a custom `__setattr__` [method](http://docs.python.org/2/reference/datamodel.html#object.__setattr__) that watches each attribute 'set' an action and alert us upon changes.

```python
    def __setattr__(self, name, value):
        if name in ['myvar1', 'myvar2']:
            import traceback
            import sys
            # Print stack (without this __setattr__ call)
            traceback.print_stack(sys._getframe(1))
            print '%s -> %s = %s' % (repr(self), name, value)
```
### Drawbacks 

The above approach works, but it doesn't give us much flexibility.

Say I wanted to use this functionality in another class. I'd need to copy the whole method and modify the list of attributes. It seems like a shame to copy/paste all that code just to change a single line. There must be a better way.

# Steps to re-usability

We can be a bit more critical about the proposed solution and notice the following:

1. The solution tightly couples the list of attributes and the real
   implementation details.
2. We will always override the same method, `__setattr__`.
3. This method must always exist inside of a class to work.

We need a way to pass a list of attribute names to a class method that will watch for `__setattr__` actions for these attributes. More specifically, we want override a built-in class method in a generic way and pass a list of attributes to watch as its sole argument.

### Potential solution

This sounds like a great use of a [Class Decorator](http://jfine-python-classes.readthedocs.io/en/latest/decorators.html).
This is a perfect use case since the only information needed is the list of attributes to watch. Plus, using Decorators would let us add this behavior to any class by using the `@` decorator syntax.

So, without further ado:

```python
def watch_variables(var_list):
    """Usage: @watch_variables(['myvar1', 'myvar2'])"""
    def _decorator(cos):
        def _setattr(self, name, value):
            if name in var_list:
                import traceback
                import sys
                # Print stack (without this __setattr__ call)
                traceback.print_stack(sys._getframe(1))
                print '%s -> %s = %s' % (repr(self), name, value)

            return super(cls, self).__setattr__(name, value)
        cls.__setattr__ = _setattr
        return cos
    return _decorator
```

In a nutshell, we make our own `_setattr` method and then set the decorated class's `__setattr__` to be our new version. To watch attributes, we'd merely have to use the `@watch_variables` decorator above your class an pass in a list of attribute names. For example:

```python
    @watch_variables(['foo', 'bar'])
    class BuggyClass(object):
        def __init__(self, foo, bar):
            self.foo = foo
            self.bar = bar
```

# Conclusion

The above solution worked for my most recent debugging sprint, and should work in many standard cases.

Unfortunately, this solution has one major drawback. **It cannot track changes to mutable attributes.** For example, this decorator won't alert you if your class has a `list` attribute that is modified by appending or some other mutation because the `append()` method and other related list modification mechanisms are
methods that manipulate the `list` attribute, not our class. Thus, our custom version of `__setattr__` will never be called.

I'm sure that this functionality can be added to our existing project. As an exercise to the reader, demonstrate your skills and [fork my solution](https://gist.github.com/4081514) to 'fix' this deficiency or point out a better solution altogether!

I hope you found this tutorial logical and engaging. Please leave comments and feedback in the discussion section below.
__________

[1] I briefly discussed connecting to Python's attribute setting mechanism
during my [dunder talk](http://durden.github.com/dunder_talk/?full#1) so check that out if this sort of functionality interests you.