Introduction
========================================================================================================
#### Good to read it

From the elementary school, we learned what is a mathematical function and how to write it. For
example - ``` f(x)=x^2``` is a simple square function of arithmetic.

A function describes a mathematical statement. The above function is
basically a mapping between two sets of natural number as given at the
right in figure \[fig:func1\]. Often it is impossible to show a mapping
graphically when we have large or infinite number of discrete elements
in one of the sets. Therefore, for convenience we express a function by
writing the inherent relation between the sets.

\[fig:func1\]

This is perhaps the simplest way when we deal with sets of numerics
(integer, natural number, real numbers, complex numbers, etc.).

We can also think of a function as a system/machine which has some
inputs, an output and a mechanism that produces the output solely from
the inputs. The sets to which the inputs and output are belongs to are
also important. A function often (not always) has a name so that we can
identify it uniquely. Typically a function is deterministic, means that
it can produce only a single output from an unique set of inputs.

**Philosophical Background**
----------------------------

Using different mathematics we tends to solve problems of various
domains.

One philosophical foundation of computer programming was to represent or
express all kinds of mathematical problems by a single language and
solve them using a generalized mathematics. The result was two
equivalent systems - <span><span> **Turing machine**</span></span> and
<span><span> **lambda calculus**</span></span>.

Turing machine influenced imperative programming paradigm such as that
of <span><span> *Assembly*</span></span>, <span><span>
*C*</span></span>, <span><span> *C++*</span></span>, <span><span>
*Java*</span></span>, <span><span> *Ruby*</span></span>, <span><span>
*Python*</span></span>, etc. languages. On the other hand, lambda
calculus influenced birth of several functional languages such as
<span><span> *LISP*</span></span>, <span><span> *ML*</span></span>,
<span><span> *Haskell*</span></span>, <span><span>
*Erlang*</span></span>, etc.

Although we can solve many more problems using those programming
languages than that of mathematics learnt in our high school, it is
often easy to learn programming by solving smaller problems first. In
this book (or booklet), we will learn how to solve a problem of known
category. At the beginning, we will see how a simple arithmetic function
can be converted into a computer program. Later, we will learn to
convert more complex functions. We will also learn how to compose a
function using other functions. A function can also be built using
itself which is called recursive functions. We will see how they can be
converted using usual pattern matching.

Rather than learning alien text, we will try to use our common sense,
simple IQ and pattern matching to convert a mathematical function into a
computer program. Although we will write all the code in the language
<span><span> *C*</span></span>, any other language can be use instead.
If we use other language, we may need to transform the given pragrams to
that target language. So, if we do not know any other language, it is
better to stick to <span><span> *C*</span></span>.

Anyway, instead of coding (writing in computer to execute it physically)
the converted code in this book, we will try to learn the programs by
writing on notebook. Initially, most problems given here are small and
easy to write on notebook (paper) by pen or pencil. In this way, the
learner could be familiarize with the relation of theoretical formula
and the implemented program.

Composition 
========================================================================================================

**Square of Distance**
----------------------

A function can be composed using other functions. It often simplifies
the expression. For example, the function, `squareOfDistance` can be
simplified if we use the function `square`. It is shown here –
``` square(x) = x \times x``` 
``` squareOfDistance_1(x_1,y_1,x_2,y_2)=square(x_1-x_2)+square(y_1-y_2)``` 
```c
Let us write it in programming language.

    int square(int x){
            return x * x;
    }
    int squareOfDistance1(int x1, int y1, 
                          int x2, int y2){
     return square(x1-x2) + square(y1-y2);
    }
```


Here we can see that the function <span>*squareOfDistance1*</span> is
using two copies of the function <span>*square*</span>. It is very
important to understand.

**Some Previous functions in New Form**
---------------------------------------

In this section, we will show some functions presented in previous
chapter in new form using composition.

### The Function `Poly2`

The old function was defined as – ``` Poly2(x) = x^4 + x^2 + 1``` 

Now we will re-define it as `Poly2_1` using the function `square`.
``` square(x) = x \times x``` 
``` Poly2_1(x) = square(square(x)) + square(x) + 1``` 

In programming, we can write it as –
```c
    int square(int x){
            return x * x;
    }

    int poly21(int x){
            return square(square(x)) + square(x) + 1;
    }
```
### The Function sumOfQube

The old function was defined as – ``` sumOfQube_1(x) = (x(x+1)/2)^2``` 

Now we will re-define it as `sumOfQube` using the function `square` and
`sum`. ``` square(x) = x \times x``` ``` sum(x) = (square(x) + x) / 2``` 
``` sumOfQube(x) = square(sum(x))``` 

In programming, we can write it as –

    int square(int x){
            return x * x;
    }

    int sum(int x){
            return (square(x) + x) / 2;
    }

    int sumOfSquare(int x){
            return square(sum(x));
    }

Prove that – `x^3 = x \times square(x)`
`x^8 = square(square(square(x)))` Simplify them by composing other
functions and write programs accordingly – `qube_c(x) = x^3`
`toPower8(x) = x^8` `toPower4(x) = x^4`

Write programs for – `parabola(a,b,c,x) = ax^2 + bx + c`
`square_p(x) = parabola(1,0,0,x)` `qube_p(x) = parabola(x,0,0,x)`

Write program for `toPower4_p(x) = parabola(parabola(1,0,0,x),0,0,x)`
and prove that `toPower4_p(x) = x^4`.


