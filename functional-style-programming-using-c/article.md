> LEARNING PROGRAMMING AND LEARNING PROGRAMMING LANGUAGE ARE
NOT THE SAME. MOST OF OUR TEXT BOOKS TEACH US PROGRAMMING LANGUAGE. 
IN FACT, A PROGRAMMING LANGUAGE IS JUST A TOOL TO IMPLEMENT A PROGRAM THAT CAN BE EXECUTED ON THE COMPUTER. WE NEED
A GOOD BOOK THAT TEACHES US HOW TO SOLVE PROBLEMS USING A PROGRAMMING LANGUAGE.


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

Simple Programs
============================================================================================================

Some basic functions will be shown converted to programs in this section. First, we will see
how to write programs for unary functions. Later, we will learn the same
of binary and n-ary function.

**Programs for Unary Arithmetic Function**
------------------------------------------

### Identity Function

\[fig:func\]

function `id(x) = x` can be converted into -

```c
    int id(int x){
     return x;
    }
```

### Successor Function

   <span>**Operator**</span> | <span>**Name**</span>
  ---------------------------|-----------------------
              `+` | Addition
              `-` | Subtraction
              `*` | Multiplication
              `/` | Division
              `%` | Reminder

function `suc(x) = x + 1` can be converted into -

```c
    int suc(int x){
        return x + 1;
    }
```

### Twice Function

function in arithmetic can be defined as – `twice(x) = 2x`

It can be converted into a programming function as -
```c
    int twice(int x){
     return 2 * x;
    }
```

### Square Function

function in arithmetic can be defined as – `square(x) = x^2`

It can be converted into a programming function as -
```c
    int square(int x){
     return x * x;
    }
```

Here we present a pictorial representation of the above function
<span>*square*</span> where the parameter `x` is mutiplied with itself
to produce and return the result.

### Polynomial Function

function in arithmetic `poly_1(x) = x^2 + 1`

can be converted into a programming function as -
```c
    int poly1(int x){
     return x * x + 1;
    }
```

function in arithmetic can be defined as – `poly2(x) = x^4 + x^2 + 1`

It can be converted into a programming function as -
```c
    int poly2(int x){
     return (x * x * x * x) + (x * x) + 1;
    }
```

can be simplified by the following function –
```
\sum_{i=1}^ni = n(n+1)/2
```

It can be converted into a programming function as -
```c
    int sum(int n){
     return n * (n + 1) / 2;
    }
```

#### Exercise 

Convert the arithmetic functions into
programming function – 

1. `f1(x) = x + 2$ $f2(x) = x^3$ $f3(x) = 2x^2 + 3`

2. `f4(x) = 3x^3 + 4x^2 + 7$ $_1(m) = m(m+1)(2m+1)/6`

3. `sumOfSquare_2(m) = m^3/3 + m^2/2 + m/6$ $_1(m) = (m(m+1)/2)^2`

4. `sumOfSquare_2(m) = m^4/4+m^3/2+m^2/4`

5. `(y) = y - 1` 

6. `(z) = z / 2`

**Programs for Binary Function**
--------------------------------

function, such as ‘add’ in arithmetic can be defined as –
`add(x, y) = x + y`

It can be converted into a programming function as -
```c
    int add(int x, int y){
     return x + y;
    }
```

function in arithmetic – `addSquare(x, y) = x^2 + 2xy + y^2`

It can be converted into a programming function as -
```c
    int addSquare(int x, int y){
     return (x * x) + (2 * x * y) + (y * y);
    }
```

### Comparison

Now we will solve a problem of determining the larger number between two
numbers. In this case, we will construct a function that takes two
parameters and it will return `1` if the first parameter is larger than
the second parameter and `0` otherwise. For that we will introduce a new
operator `>`.

`a > b` is `1` or (<span>*`True`*</span> in meaning) if `a` is larger than
`b`. Otherwise it is `0` (<span>*`False`*</span> in meaning). So, our
function is – `isLarger(x, y) = x > y`

The corresponding program is –

   <span>**Operator**</span> | <span>**Name**</span>
  ---------------------------|--------------------------
              `>` | Greater than
             `>=` | Greater than or equal to
              `<` | Smaller than
             `<=` | Smaller than or equal to
             `==` | Equals to
             `!=` | Not equals to

```c
    int isLarger(int x, int y){
     return x > y;
    }
```

#### Exercise 

Convert the arithmetic functions
into programming function – 

1. `sub(x,y) = x - y`

2. `addOfSquare(x,y) = (x \times x) + (y \times y)`

3. `subOfSquare(x,y) = (x \times x) - (y \times y)`

4. `subOfSquare_1(x,y) = (x+y)(x-y)` 

5. `g(a,b) = a + ab + b`

Write functions in arithmetic and programming to – multiply two numbers.
calculate average of two numbers. determine if the two given numbers are
equal. determine if the two given numbers are not equal.

**Programs for n-Ary Function**
-------------------------------

### 3-ary Function

A ternary function takes three parameter. For example –
`avgOfThree(x, y, z) = (x + y + z) / 3`

It can be converted into a programming function as -
```c
    int avgOfThree(int x, int y, int z){
     return (x + y + z) / 3;
    }
```

### N-ary Function

In a n-ary function, number of parameters is n. In this example, square of distance
is an useful function to compare two distances.

The squareOfDistane function
`squareOfDistance(x_1, y_1, x_2, y_2) = (x_1-x_2)(x_1-x_2) + (y_1-y_2)(y_1-y_2)`

can be converted into a programming function as -
```c
    int squareOfDistance(int x1, int y1, 
                         int x2, int y2){
     return (x1-x2)*(x1-x2) + (y1-y2)*(y1-y2);
    }
```

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


