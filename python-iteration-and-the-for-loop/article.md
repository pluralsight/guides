I have spent some time really digging into
[generators](http://docs.python.org/2/tutorial/classes.html#generators) and by
extension
[iterables](http://docs.python.org/2/library/stdtypes.html#iterator-types).
The official docs are pretty good, but I found that
[David Beazley](http://dabeaz.com/)'s
[generator explanation](http://www.dabeaz.com/generators/Generators.pdf) fills
a lot of the gaps.

To truly understand
[generators](http://docs.python.org/2/tutorial/classes.html#generators) you
must start with the concept of protocols. So, I decided this might be a good
time for a little article about iteration and look inside the python
[for loop](http://docs.python.org/2/tutorial/controlflow.html#for-statements).

## Protocols

One of the nice features of [Python](http://python.org) is that under the hood
objects can be designed and implemented by so-called protocols. These are not
very well documented, but several exist:

- [Descriptor](http://www.rafekettler.com/magicmethods.html#descriptor)
- [Iteration](http://docs.python.org/2/library/stdtypes.html#iterator-types)
- [Pickling](http://www.rafekettler.com/magicmethods.html#pickling)
- [Sequence](http://www.rafekettler.com/magicmethods.html#sequence)

These protocols are implemented by providing your own
[special methods](http://docs.python.org/2/reference/datamodel.html#special-method-names),
sometimes called [dunders](http://durden.github.com/dunder_talk/?full#1).

So, to make your object iterable it must respond to the
[iteration protocol](http://docs.python.org/2/library/stdtypes.html#iterator-types), namely the `__iter__()` and `next()` methods. [1]

## Iteration and generators

Have you ever wondered how the `for` loop actually worked in
[Python](http://python.org)? Surprise, it's all based on the same
[iteration protocol](http://docs.python.org/2/library/stdtypes.html#iterator-types).

[David Beazley](http://dabeaz.com/)'s
[generator explanation](http://www.dabeaz.com/generators/Generators.pdf) does a
great job of explaining how this protocol fuels the implementation of the `for`
loop:

```python
    _iter = iter(obj) # Get iterator object
    while 1
        try:
            x = _iter.next() # Get next item
        except StopIteration: # No more items
            break
        # statements
```

Make sure to read through the entire
[generator presentation](http://www.dabeaz.com/generators/Generators.pdf).
There are all sorts of other nice details you might have forgotten or never
seen before.

<small>
    [1] `next()` doesn't have a `__next__()` in Python 2.7, but this was
    [fixed](http://docs.pythonsprints.com/python3_porting/py-porting.html#creating-iterators)
    in [Python 3](http://docs.python.org/3.3/).
</small>