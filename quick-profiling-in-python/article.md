This is my 'high-level' approach to profiling and speeding up code in
[Python](http://python.org). This is by no means an exhaustive approach and
will most likely only point out quick and easy places to speed up your code.

The approach is pretty standard for profiling any code: start at the
top-level and methodically work your way down the codebase to find the
bottlenecks.

This approach is designed to find the most obvious performance bottlenecks
quickly and allow you to get back to coding. Fortunately it's is easy enough
to run on your code periodically and find some pain points sooner rather than
later. [1]

## Step One: Profile entire application

The first step is to run the script/application with the
[built-in profiler](http://docs.python.org/2/library/profile.html#module-cProfile)
and make sure to stress the slower or more interesting code sections:

`python -m cProfile -o stats <my_awesome_code>.py`

Next, exit the application and use this little script to see where most of the
running time was spent:

[gist 4272487]

This script will output all the function calls your application performed and
sort them by cumulative time.

## Step Two: Reduce calls to expensive code

This profile output represents an idea of where the majority of the processing
time was spent. This may or not be useful at this point, but it can give yield
a few clues.

1. Take a look at the `percall` column
    - Is any particular function taking a big chunk of time in a single call?
2. Take a look at the `ncalls` column
    - Can you reduce the number of times you call this function?

## Step Three: Focus on a single function

It's possible that steps one and two did not yield any useful performance
increases. So, now is the time to drop down a level and narrow down a single
function to focus on.

Take another look at the profiling output from step one. Find the function
with the largest `percall` column. Now, profile each line in this function to
see if there is something slow that can be easily re-factored.

For this step a few additional profiling tools are needed:

1. [Line Profiler](http://pypi.python.org/pypi/line_profiler)
2. [Kernprof.py](http://packages.python.org/line_profiler/kernprof.py)

You can find **some** documentation on these
[here](http://packages.python.org/line_profiler/). In general, I've found
these are a little awkward to use and documentation is a bit lacking. Luckily,
this simple approach to profiling doesn't need too much documentation.

The first and probably most awkward step is to place the `@profile` decorator
around the function you're interested in profiling [2]. Don't worry, there is
nothing to import for `@profile` because it's **magic** [3].

Next, run the `kernprof.py` script with your script/application as the
argument:

`kernprof.py -v -l <script> <your_script_args>`.

Now perform the operations that will cause the profiled function to run.

Finally, we can use the `line_profiler` module to look at the results. The
above invocation of the `kernprof.py` script created a profiler data file,
typically called `<your_profiled_script>.lprof`.

Feed this data file to `line_profiler` module, and it will print the timing of
the function broken down by line:

`python -m line_profiler <data_file>`

At this point, you will probably see a few lines stand out as far as the
`Time` and `% Time` columns go. In my experience, the slowness at this level
tends to be something like building a list with a lot of elements, constantly
looking for existence of an item in a big list, and other operations that deal
with large [iterables](http://docs.python.org/2/glossary.html#term-iterable).

## Step Four: Back to basics

Now the easy part is finished. It's time to think about the algorithm and
data structures. The process of improving your slow performing code is a bit
outside of this post since that process is usually very specific to the code
itself.

Remember all that
[algorithm complexity](http://en.wikipedia.org/wiki/Analysis_of_algorithms) and
[Big O](http://en.wikipedia.org/wiki/Big_O_notation) homework from college?
Bust our your books, refresh your memory, and let the real journey begin.

## Example Optimization

This is a pretty classic case, but you would be surprised how often it shows
up.

For example, consider a snippet of code that spends a lot of time looking for
existence of an object in a large list within a
[tight loop](http://stackoverflow.com/questions/2212973/what-is-a-tight-loop).
This could be a great place to use a dictionary instead.

The dictionary will greatly improve your look up time but will waste more
memory. This is the classic trade off and only you can decide if the increased
memory usage is worth it in the context of your application.


## Still Slow?

As I mentioned in the beginning, this approach might leave you needing more
speed. Luckily, there are still several options.

1. [Cython](http://www.cython.org/)
    - Cython is a common approach to speeding up Python code. It provides a
      way to add data type information to your existing code and other
      mechanisms to generally push your code closer to the
      [C](http://en.wikipedia.org/wiki/C_(programming_language)) layer.
2. [PyPy](http://pypy.org/)
    - PyPy is an alternative implementation of the [Python language
      spec](http://docs.python.org/2/reference/). PyPy is, in a nutshell,
      Python implemented in Python. A real explanation of PyPy is beyond this
      post and my expertise. However, if you're interested in languages and some
      serious [Computer Science](http://en.wikipedia.org/wiki/Computer_science)
      definitely look into it. It will test your thought process, which is
      always a good thing.
3. [Numba](https://github.com/numba/numba)
    - Numba is a concept similar to [Cython](http://www.cython.org/). It uses
      the ever popular [LLVM](http://llvm.org/) compiler tool chain to achieve
      super fast results. Another benefit of Numba is it's written by the
      great folks at [Continuum Analytics](http://continuum.io/). Thus, Numba
      has some of the founders of the scientific Python community and
      [numpy](http://docs.scipy.org/doc/) behind it. This project is still
      relatively young, but I expect big things from it in the future.

## Caveats

Some bottlenecks like storage I/O, network I/O, etc. are not going to easily
show up with this type of profiling. This approach is a quick way to profile
and fixing [CPU bound](http://en.wikipedia.org/wiki/CPU_bound) tasks.

## Conclusion

Profiling and optimization is a very complicated topic. So, my simple approach
is barely scratching the surface. This is a topic you will definitely want to
learn more about if you are interested in becoming a better programmer.
Luckily, there are some really
[great talks](http://pyvideo.org/search?models=videos.video&q=profiling) on
this subject to help you learn more from real experts.

<small>
[1] It would be useful to automate some of this process into your
[Continuous Integration](http://en.wikipedia.org/wiki/Continuous_integration)
server to find performance regressions, but that is a different post
altogether.
</small>

<small>
[2] I've seen problems trying to use `@profile` on more than one function at a
time.
</small>

<small>
[3] I wish this was done in a different way. The magic of inserting this into
the `__builtins__` really bothers me philosophically.
</small>

