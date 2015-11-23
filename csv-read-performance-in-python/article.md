There are a few ways to read data from a
[csv](http://en.wikipedia.org/wiki/Comma-separated_values) file into a numpy
array:

1. [numpy.loadtxt](http://docs.scipy.org/doc/numpy/reference/generated/numpy.loadtxt.html)
2. [pandas.read_csv](http://pandas.pydata.org/pandas-docs/stable/generated/pandas.io.parsers.read_csv.html) [1]

I recently had to read this type of data for a project I'm working on. It's
worth looking into the performance since there are a few choices. This post is
the result of that investigation.

## Sample data setup

I wrote the following small function to generate a sufficient amount of
'random' data for testing:

```python
def generate_test_data(column_names, row_count, filename):
    """
    Generate file of random test data of size (row_count, len(column_names))

    column_names - List of column name strings to use as header row
    row_count - Number of rows of data to generate
    filename - Name of file to write test data to
    """

    col_count = len(column_names)
    rand_arr = np.random.rand(row_count, col_count)
    header_line = ' '.join(column_names)
    np.savetxt(filename, rand_arr, delimiter=' ', fmt='%1.5f',
               header=header_line, comments='')
```

Hopefully this function is straight-forward so I won't discuss it further.

For this test I simplify used the above function to create a relatively small
file:

```python
    # For testing just create a column for each lower case letter in English
    # alphabet
    columns = [char for char in string.lowercase]
    row_count = 1000

    # Don't need the file open. In order to time things properly we should
    # allow each method to open the file, etc. itself.
    fd, filename = tempfile.mkstemp()
    os.close(fd)

    generate_test_data(filename, columns, row_count)
```

This creates a space-separated file of random float data that is about 208 KB,
comprised of 26 columns and 1000 rows.

## Test results

The following snippet is from an [IPython](http://ipython.org/) shell utilizing
the `%timeit` functionality [2]:

```python
    >>> import numpy as np
    >>> import pandas as pd
    >>> %timeit -n 100 pd.read_csv('test.out', delim_whitespace=True)
    100 loops, best of 3: 6.66 ms per loop
    >>> %timeit -n 100 f = open('test.out', 'r');f.readline();np.loadtxt(f, unpack=True)
    100 loops, best of 3: 28 ms per loop
```

## Pandas wins!

The result is that [Pandas](http://pandas.pydata.org/) is **much** faster, but
why? The short answer, as developer [Wes McKinney](http://wesmckinney.com)
posted in [response](http://wesmckinney.com/blog/?p=635#comment-823036360) to
my [question](http://wesmckinney.com/blog/?p=635#comment-823023854):

> Short answer: file tokenization and type inference is being handled at the
> lowest level possible in C/Cython. If you look at the impl of numpy.loadtxt
> you'll see a lot of Python.

So there you have it, straight from the author! Interestingly enough the
massive speed increases for `pandas.read_csv` are relatively recent, and
[Wes](http://wesmckinney.com) has written a few great articles detailing it in
full.

- [Speeding up pandas' file parsers with Cython](http://wesmckinney.com/blog/?p=278)
- [A new high performance, memory-efficient file parser engine for pandas](http://wesmckinney.com/blog/?p=543)
- [Update on upcoming pandas v0.10, new file parser, other performance wins](http://wesmckinney.com/blog/?p=635)

### Notes

[1] Using `pandas.read_csv` will return a
[DataFrame](http://pandas.pydata.org/pandas-docs/version/0.9.1/dsintro.html#dataframe)
object, which is essentially a 2 dimensional
[numpy array](http://docs.scipy.org/doc/numpy/reference/generated/numpy.array.html).
So, the performance improvements of `pandas.read_csv` could come at the price
of another dependency in your project for [Pandas](http://pandas.pydata.org/).
Also, you'll be getting back a 
[DataFrame](http://pandas.pydata.org/pandas-docs/version/0.9.1/dsintro.html#dataframe)
object instead of more stripped down
[numpy arrays](http://docs.scipy.org/doc/numpy/reference/generated/numpy.array.html).

[2] Note that I used the `unpack=True` argument to `numpy.loadtxt` because I
want to read all the data as column-based arrays, not the default row-based.
This was just a requirement for the application I was profiling for. This
isn't necessary in the [Pandas](http://pandas.pydata.org/) library because data
is read into a
[DataFrame](http://pandas.pydata.org/pandas-docs/version/0.9.1/dsintro.html#dataframe)
object, which allows slicing by columns.

