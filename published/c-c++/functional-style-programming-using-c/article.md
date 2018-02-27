> Learning programming and learning programming language are not the same. most of our textbooks teach us programming languages. In truth, a programming language is just a tool that we use to implement an executable program. We need resources that teach us how to solve problems using a programming language.

Introduction
========================================================================================================

As early as elementary school, we learned what is a mathematical function and how to write it. For example, ``` f(x)=x^2``` is a simple quadratic function.

A function describes a mathematical statement. The above function maps the relationship between two sets of natural numbers as given at the right in the graph below. Often, it is impossible to show such a mapping graphically when we have a large number of discrete elements in either of the sets. Therefore, for convenience, we express a function by writing it as an inherent relation between the sets.

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6a1dd0fd-87a9-4864-a01f-4a916d534c15.jpg)

This is perhaps the simplest way to demonstrate relationships when we deal with sets of numbers (integer, natural number, real numbers, complex numbers, etc.)

We can also think of a function as a machine which has taken in inputs and produces an output by using a mechanism that maps the relationship from input to output. The sets to which the inputs and output belong are also important. A function often has a name that allows us to identify it. Typically a function is deterministic, it can produce only a single output from a unique set of inputs.

**Philosophical Background**
----------------------------

Using different mathematical strategies, we can solve problems under various domains.

One philosophical foundation of computer programming was to represent or express all kinds of mathematical problems by a single language and solve them using generalized mathematics. The result was two equivalent systems, <span><span> **the Turing machine**</span></span> and <span><span> **lambda calculus**</span></span>.

The concept of a Turing machine influenced imperative programming paradigm such as those of <span><span> *Assembly*</span></span>, <span><span> *C*</span></span>, <span><span> *C++*</span></span>, <span><span> *Java*</span></span>, <span><span> *Ruby*</span></span>, <span><span> *Python*</span></span>, and other programming languages. 

On the other hand, lambda calculus influenced the birth of several functional languages such as <span><span> *LISP*</span></span>, <span><span> *ML*</span></span>, <span><span> *Haskell*</span></span>, and <span><span> *Erlang*</span></span>.

Although we can solve many more problems by using those programming languages than by using high school math, it is often easy to learn programming by solving smaller problems first. In this book (or booklet), we will learn how to solve a problem of known category. In the beginning, we will see how a simple arithmetic function can be converted into a computer program. Later, we will learn to convert more complex functions. We will also learn how to compose a function using other functions. A function can also be built using itself through recursion. We will see how they can be converted using usual pattern matching.

Rather than learning alien text, we will try to use our common sense, simple IQ and pattern matching to convert a mathematical function into a computer program. Although we will write all the code in the language <span><span> *C*</span></span>, any other language can be used instead. If we use other languages, we may need to transform the given programs to that target language. So, if we do not know any other language, it is better to stick to <span><span> *C*</span></span>.

Anyway, instead of coding (writing in a computer to execute it physically) the converted code in this book, we will try to learn the programs by jotting it down independently. Initially, most problems given here are small and easy to write on a notebook (paper) by pen or pencil. In this way, the learner becomes familiar with the relationship between theoretical formulas and the implemented programs.

Simple Programs
============================================================================================================

We will convert some basic functions into programs in this section. First, we will see how to write programs for unary functions. Later, we will learn the same of binary and n-ary function.

**Programs for Unary Arithmetic Function**
------------------------------------------

### Identity Function

The identity function (or a function in which output mirrors the input) looks like this: 

![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/6a1dd0fd-87a9-4864-a01f-4a916d534c15.jpg)


Function `id(x) = x` can be converted into:

```c
    int id(int x){
     return x;
    }
```

### Successor Function

Successor functions use the `+` operator to add `1` to the input and return its successor. 

   <span>**Operator**</span> | <span>**Name**</span>
  ---------------------------|-----------------------
              `+` | Addition
              `-` | Subtraction
              `*` | Multiplication
              `/` | Division
              `%` | Reminder

Function `suc(x) = x + 1` can be converted to:

```c
    int suc(int x){
        return x + 1;
    }
```

### Twice Function

This is defined as `twice(x) = 2x` in arithmetic.

In C, the same function takes the form below. 
```c
    int twice(int x){
     return 2 * x;
    }
```

### Square Function

This function is defined as `square(x) = x^2` arithmetically.

In C, it takes the form below. 
```c
    int square(int x){
     return x * x;
    }
```

Here we present a pictorial representation of the above function <span>*square*</span> where the parameter `x` is mutiplied with itself to produce and return the result.


![description](https://raw.githubusercontent.com/pluralsight/guides/master/images/026d259c-d095-4b36-a73c-dc96a1bdd926.gif)


### Polynomial Function

In arithmetic, the function is defined as: `poly_1(x) = x^2 + 1`

It can be converted into a programming function as shown below: 
```c
    int poly1(int x){
     return x * x + 1;
    }
```

In arithmetic, the function is defined as: `poly2(x) = x^4 + x^2 + 1`

It can be converted into a programming function as shown below: 
```c
    int poly2(int x){
     return (x * x * x * x) + (x * x) + 1;
    }
```

This can be simplified:
```
sum_{i=1}^ni = n(n+1)/2
```

It can be converted into a programming function as:
```c
    int sum(int n){
     return n * (n + 1) / 2;
    }
```

#### Exercise 

Convert the arithmetic functions into
programming function:

1. `f1(x) = x + 2$ $f2(x) = x^3$ $f3(x) = 2x^2 + 3`

2. `f4(x) = 3x^3 + 4x^2 + 7$ $_1(m) = m(m+1)(2m+1)/6`

3. `sumOfSquare_2(m) = m^3/3 + m^2/2 + m/6$ $_1(m) = (m(m+1)/2)^2`

4. `sumOfSquare_2(m) = m^4/4+m^3/2+m^2/4`

5. `(y) = y - 1` 

6. `(z) = z / 2`

**Programs for Binary Function**
--------------------------------

function, such as ‘add’ in arithmetic can be defined as:
`add(x, y) = x + y`

It can be converted into a programming function as:
```c
    int add(int x, int y){
     return x + y;
    }
```

function in arithmetic – `addSquare(x, y) = x^2 + 2xy + y^2`

It can be converted into a programming function as:
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

The corresponding program is:

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
into programming function:

1. `sub(x,y) = x - y`

2. `addOfSquare(x,y) = (x 	times x) + (y 	times y)`

3. `subOfSquare(x,y) = (x 	times x) - (y 	times y)`

4. `subOfSquare_1(x,y) = (x+y)(x-y)` 

5. `g(a,b) = a + ab + b`

Write functions in arithmetic and programming to – multiply two numbers.
calculate average of two numbers. determine if the two given numbers are
equal. determine if the two given numbers are not equal.

**Programs for n-Ary Function**
-------------------------------

### 3-ary Function

A ternary function takes three parameter. For example:
`avgOfThree(x, y, z) = (x + y + z) / 3`

It can be converted into a programming function as:
```c
    int avgOfThree(int x, int y, int z){
     return (x + y + z) / 3;
    }
```

### N-ary Function

In a n-ary function, number of parameters is n. In this example, square of distance
is an useful function to compare two distances.

The `squareOfDistance` function:
`squareOfDistance(x_1, y_1, x_2, y_2) = (x_1-x_2)(x_1-x_2) + (y_1-y_2)(y_1-y_2)`

Can be converted into a programming function as:
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
simplified if we use the function `square`. It is shown here:
``` 
square(x) = x times x

squareOfDistance_1(x_1,y_1,x_2,y_2) = square(x_1-x_2)+square(y_1-y_2)
``` 

Let us write it in a programming language:
```c
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

The old function was defined as `Poly2(x) = x^4 + x^2 + 1`.

Now we will re-define it as `Poly2_1` using the function `square`:
``` 
square(x) = x times x

Poly2_1(x) = square(square(x)) + square(x) + 1
``` 

In C, we can write it as:
```c
    int square(int x){
            return x * x;
    }

    int poly21(int x){
            return square(square(x)) + square(x) + 1;
    }
```
### The Function sumOfQube

The old function was defined as `sumOfQube_1(x) = (x(x+1)/2)^2`.

Now we will re-define it as `sumOfQube` using the function `square` and
`sum`:
```
square(x) = x times x

sum(x) = (square(x) + x) / 2

sumOfQube(x) = square(sum(x))
``` 

In C, we can write it as:

    int square(int x){
            return x * x;
    }

    int sum(int x){
            return (square(x) + x) / 2;
    }

    int sumOfSquare(int x){
            return square(sum(x));
    }

#### Exercise

1. Prove that:

  a. `x^3 = x times square(x)`

  b. `x^8 = square(square(square(x)))` 

2. Simplify them by composing other functions and write programs accordingly:

 a. `qube_c(x) = x^3`

 b. `toPower8(x) = x^8` 

 c. `toPower4(x) = x^4`

3. Write programs for:

 a. `parabola(a,b,c,x) = ax^2 + bx + c`

 b. `square_p(x) = parabola(1,0,0,x)`
 
 c. `qube_p(x) = parabola(x,0,0,x)`

4. Write program for `toPower4_p(x) = parabola(parabola(1,0,0,x),0,0,x)`
and prove that `toPower4_p(x) = x^4`.

Condition 
==============================================================================================================

From this chapter, we will not write same function twice.

**General Idea**
----------------

In a function, we may get different result based on different conditions
on the parameters. For example, the function <span>*absolute*</span>
negates its parameter if it is negative so that it can be turned into
positive (negative of negative is positive) to produce the result.
Otherwise (when parameter is already positive), it returns back the
result exactly same as the parameter. Thus, the function can be written
as:
``` 
abs(x) = begin{dcases*}
        x & when `x geq 0`
        -x & otherwise
        end{dcases*}
``` 

Compute based on conditions of parameters.

The general idea of a conditional function is as follows:

``` 
f(x) = begin{dcases*}
        e_1 & when `1^{st}` condition satisfies
        e_2 & when `2^{nd}` condition satisfies
        e_3 & when `3^{rd}` condition satisfies
        dots & dots 
        e_n & otherwise
        end{dcases*}
``` 

The corresponding code is:
```c
    int f(int x){
            if( 1st condition )
                    return e1;
            else if( 2nd condition )
                    return e2;
            else if( 3rd condition )
                    return e2;
            ...
                    ...
            else
                    return en;
    }
```

Now we will present some functions those use conditions.

**if …else …**
--------------

### Absolute

``` 
abs(x) = begin{dcases*}
        x & when `x geq 0`
        -x & otherwise
        end{dcases*}
``` 
It's code:
```c
    int abs(int x){
            if( x >= 0 )
                    return x;
            else
                    return -x;
    }
```

### Filter

``` 
filter_{LT}(x,y) = begin{dcases*}
        x & when `x geq y`
        0 & otherwise
        end{dcases*}
``` 
Its code:
```c
    int filterLT(int x, int y){
            if( x >= y )
                    return x;
            else
                    return 0;
    }
```

if …else …+ Composition
-----------------------

By composing functions, we can construct many familiar functions.
Although they are not always more efficient, for obvious reason, we will
do it.

Here the function <span>*abs*</span> is rewritten as:
``` 
switcher(a,b,c,d) = begin{dcases*}
        c & when `a geq b`
        d & otherwise
        end{dcases*}

abs_1(x) = switcher(x, 0, x, -x)
``` 

In C:
```c
    int switcher(int a,int b,int c,int d){
            if(a >= b)
                    return c;
            else
                    return d;
    }
    int abs1(int x){
            switcher(x, 0, x, -x);
    }
```

### Tuition Fee Calculator

Lets think a problem.

Calculate your total tuition fee due based on the condition that if your
due of previous semester tuition fee is more that 1000 currency, your
total fee to pay this semester will be double of previous due plus the
fee of current semester. We can write a function to describe it
mathematically.

``` 
tftp(p, c)=begin{dcases*}
        2 times p + c & when `p > 1000`
        p + c & otherwise
        end{dcases*}
``` 

If we use the function `switcher`, we can rewrite it as:

``` 
hbox{em tftp}_1(p, c) = 
    switcher(p, 1001, 2 times p, p) + c
``` 
 or, 

```
hbox{em tftp}_2(p, c) = 
    switcher(p, 1001, p, 0) + p + c
``` 

In programming, the functions are:

```c
    int switcher(int a,int b,int c,int d){
            if(a >= b)
                    return c;
            else
                    return d;
    }
    int tftp1(int p, int c){
            return switcher(p, 1001, 2 * p, p) + c;
    }
    int tftp2(int p, int c){
            return switcher(p, 1001, p, 0) + p + c;
    }
```

Well, the parameters <span>`p`</span> and <span>`c`</span> does not mean
anything. So, often it misleads us if we use meaningless names. It is
also important to be able to read them easily. We can take different
approaches to avoid confusions and increase readibility. They are shown
as the function <span>*`tftp2`*</span> is rewritten as:

```c
    /*
    tftp: tuition fee to pay
    */
    int tftp2(int p, int c){
            /*
            p: previous month's due
            c: current month's fee
            */
            return switcher(p, 1001, p, 0) + p + c;
    }
```
Or, in the best style:
```c
    int tftp2(int previousDue, int currentFee){
            return switcher(previousDue, 1000,
                            previousDue, 0)  
                            + previousDue 
                            + currentFee;
    }
```
Or, in a good style:
```c
    int tftp2(int preDue, int curFee){
            return switcher(preDue, 1001, preDue, 0) 
                   + preDue + curFee;
    }
```
Or, we can write a word to keep it small and yet understandable:
```c
    int tftp2(int due, int fee){
            return switcher(due,1001,due,0)+due+fee;
    }
```

**if …else if …else …**
-----------------------

### Grade Letter (in numeric)

In this example, we will solve a problem of determining the grade letter
based on the total mark in an examination. The grade distribution is
given at the right side.

         Range Grade | letter | In numeric
  -------------------|--------|-------------------
         90-100 | A |             1
         80-89 | B | 2
         70-79 | C | 3
         60-69 | D | 4
         40-59 | E | 5
   0-39, <0, >100 | F | 6

The function that solves the problem is:
``` 
getGrade(mark) = begin{dcases*}
        1 & when `mark geq 90` and `mark leq 100`
        2 & else when `mark geq 80`
        3 & else when `mark geq 70`
        4 & else when `mark geq 60`
        5 & else when `mark geq 40`
        6 & otherwise
        end{dcases*}
``` 
Hence, the code for the above function is:
```c
    int getGrade(int mark){
            if( mark >= 90 && mark <= 100)
                    return 1;
            else if( mark >= 80 )
                    return 2;
            else if( mark >= 70 )
                    return 3;
            else if( mark >= 60 )
                    return 4;
            else if( mark >= 40 )
                    return 5;
            else 
                    return 6;
    }
```

In the above example, the function gives us a numeric value that in our
mind represents a grade. But it is often necessary to give user a direct
result instead of a numerical representation. If the function can
generate result with letter grades directly, it will be definitely more
understandable as shown here: 
``` 
getGradeLetter(mark) = begin{dcases*}
        hbox{'A'} & when `mark geq 90` and `mark leq 100`
        hbox{'B'} & else when `mark geq 80`
        hbox{'C'} & else when `mark geq 70`
        hbox{'D'} & else when `mark geq 60`
        hbox{'E'} & else when `mark geq 40`
        hbox{'F'} & otherwise
        end{dcases*}
``` 
Since the letter grades are actually letters or
characters, we need to change the data type of the function to ‘char’.
‘char’ type corresponds to the characters. In other words, ‘char’ is the
set of all characters or visible symbols a computer can show.

  <span>**Type**</span> |     <span>**Nature of Data**</span>
  ----------------------|---------------------------------
  int | integer
  long | large integer
  long long | very large integer
  float | real number
  double | large real number
  char | character

The code for the above function is:
```c
    char getGradeLetter(int mark){
            if( mark >= 90 && mark <= 100)
                    return 'A';
            else if( mark >= 80 )
                    return 'B';
            else if( mark >= 70 )
                    return 'C';
            else if( mark >= 60 )
                    return 'D';
            else if( mark >= 40 )
                    return 'E';
            else 
                    return 'F';
    }
```


**switch …case …**
------------------

### Item Price 

In this example, we will solve a problem of getting prices of different
items in an restaurant. We will determine an item by its item no. The
prices for different item numbers are given at the right.

      Item Item | No. |Price
  --------------|-----|-------------
   Fish Curry | 1 | 60.00
    Fish Fry | 2 | 80.00
   Beef Curry | 3 | 100.50
    Beef Vuna | 4 | 130.75
     Chicken | 5 | 85.00
      Rice | 6 | 15.25
      Water | 7 | 20.00
   Cold Drinks | 8 | 15.00

The function that solves the problem is:
``` 
{hbox{em getPrice}}({hbox{em item}}) = begin{dcases*}
        60.00 & when {em item} `= 1`
        80.00 & when {em item} `= 2`
        100.50 & when {em item} `= 3`
        130.75 & when {em item} `= 4`
        85.00 & when {em item} `= 5`
        15.25 & when {em item} `= 6`
        20.00 & when {em item} `= 7`
        15.00 & when {em item} `= 8`
        0 & otherwise 
        end{dcases*}
``` 
Hence, the code for the above function is:
```c
    float getPrice(int item){
            switch( item ){
            case 1:
                    return 60.00;
            case 2:
                    return 80.00;
            case 3:
                    return 100.50;
            case 4:
                    return 130.25;
            case 5:
                    return 85.00;
            case 6:
                    return 15.25;
            case 7:
                    return 20.00;
            case 8:
                    return 15.00;
            default:
                    return 0;
            }
    }
```

  <span>**Type**</span> | <span>**Range**</span>
  ----------------------|---------------------------
  int/long |                  `-2^{31}` to `2^{31} - 1`
  long long | `-2^{63}` to `2^{63} - 1`
  float |     `1.2E-38` to `3.4E+38`
  double | `2.3E-308` to `1.7E+308`
  long double | `3.4E-4932` to `1.1E+4932`
  char |     `-128` to `127`

Note: `2^{31}=2,147,483,648` and `2^{63}=9,223,372,036,854,775,808`

For a realistic approach, we can add a simple function with it that
calculates total price of an item for a quantity:
``` 
calcTotalPrice(item, quantity) = quantity times getPrice(item)
``` 

The code:
```c
    float calcTotalPrice(int item, int quantity){
            return quantity * getPrice( item );
    }
```

Recursions 
============================================================================================================
### Easy tricks to repeat

Recursion is a technique that enhances capabilities of programs very
much. For example, if we want to write an exponential function
`exp(x,k)=x^k`, we understand that `x` have to be multiplied to itself
`k` times. We can understand `x^k` operation from our childhood
mathematics. But unfortunately C programming language does not have
operator for exponential operation. We can also understand `k` times and
we can repeat this multiplication by counting `1` to `k`.

But how to write the function using multiplication only? We can write
`exp(x,k)=x times x times dots times x` (`k` times). If we knew `k`
is `2`, the expression becomes `exp(x,2) = x 	times x`, if it is `3`,
the expression is `exp(x, 3) = x times x times x` and so on. But when
`k` is unknown, then it is difficult to write in very clear way. Most
importantly, C programming does not support any expression like
‘`x times x times dots times x` (`k` times)’.

**Intuition of Recursive definition**
-------------------------------------

We use recursion to express such functions or programs. Recursion is a
special kinds of composition where the definable function itself is
composed with something else to define itself. It may heard weird or
twisted at the first time. We will see some analogous definition in our
society to understand its power.

But just imagine a simple definition of <span><span>
*human*</span></span> (according to Quran and Bible):
>‘Adam and Eve
were <span><span> *human*</span></span> and any living mammal that borns
from another <span><span> *human*</span></span> is a <span><span>
*human*</span></span>’. 

This definition certainly covers all the humans.
Here <span><span> *human*</span></span> is defined from the definition
of another human and it is called recursion or induction. In recursive
(or inductive) defintion, there must be at least one base point for
example ‘Adam was <span><span> *human*</span></span> and Eve was
<span><span> *human*</span></span>’.

Let us take another example, the definition of natural numbers. Suppose we
have all kinds of numbers on our knowledge including real numbers,
complex numbers, etc.. We need to define natural number. Here is the
plain definition - `0` is a natural number and `n+1` is a natural number
if `n` is a natural number. We can use it to write a function to know
whether a number is natural
``` 
{hbox{em isNatural}}(x)=begin{dcases*}
        {hbox{em true}}& when `x=0`
        {hbox{em false}}& when `x<0`
        {hbox{em isNatural}}(x-1) & otherwise
        end{dcases*}
``` 

Similarly we can write another function to distinguish an animal as
human:
``` 
{hbox{em isHuman}}(x)=begin{dcases*}
        {hbox{em true}}& when `x=` {em Adam}
        {hbox{em true}}& when `x=` {em Eve}
        {hbox{em false}}& when `x=` {em Amoeba}
        {hbox{em isHuman}}({hbox{em mother of }}x) & otherwise
        end{dcases*}
``` 

It says, `x` is a human if `x` is <span>*Adam*</span> or
<span>*Eve*</span>. `x` is not human if `x` is <span>*Amiba*</span>.
Besides `x` is also human if and only if mother of `x` is also human.
Well, in the real process, to justify an animal as human, we need to
know the total family tree from <span>*Adam*</span> and
<span>*Eve*</span> which is very unrealistic. But the definition itself
is very intuitive.

In mathematics and computer science, this recursive style of definition
often expresses a fact in a very simple and intuitive way, which helps
to formalize an algorithm or program in very effective way.

To create a recursive definition of a function, we need to forget
how it really works. We simply need to know if the definition is correct
mathematically.

For an easy construction of the definition, here we would like to
present a tips. We have to remember that the function has one (or more)
driving parameter(s).

1. Base cases part  

    Need to determine the value of the function for the base case(s) of
    the driving parameter(s). For example, `exp(n,x)=1`
    when `x=0`.

2. Recursion part  

   The function itself will be appeared with smaller (typically
   1 step) value or size of the driving parameter(s). For example,
   `exp(x-1)`. This is called induction hypothesis (HI).

    - The induction hypothesis often needs to be computed with
        something else to be equal to the original definition. For
        example, `exp(x)=x 	times exp(x-1)`.

    - Other parameters remain the same in recursion call inside
        the body.

If we follow the above steps, it will help us to construct recursive
definition. We will explore some examples soon. In some examples, we will
see how the function is constructed from the above tips. We will also
see whether definition is correct.

**Exponential**
---------------

We will define `exp(x,k)`. Here `x` is the base and `k` is the power. So,
`k` is the driving parameter. Now we know the base case when `k=0` -
`exp(x,0) = 1`. Next, we will have to construct the recursion part. First, we
know that the expression `exp(x,k-1)` exists in this part because the 1
step smaller value of `k` is `k-1`. `x` will remain same. Now it is
necessary to multiply it to `x` to be equal to `exp(x,k)`. Therefore, the
full expression becomes `exp(x,k) = x times exp(x,k-1)`. Now together the
function is looks like:

``` 
exp(x,k) =
        1, when `k=0`
        x times exp(x,k-1), otherwise
``` 

Obviously, the code for the above function is:
```c
    int exp(int x, int k){
     if( k == 0 )
                    return 1;
            else
                    return x * exp( x, k-1 );
    }
```

<span>**Proof of correctness**</span>

Let us see whether the above function, `exp(x,k)` is really `x^k`. We
can prove it by using induction on `k`.

**Case 1.** `k=0`.

               . | .
  ---------------|--------------
  `exp(x,0) = 1 ` |…(1) from the function
  `x^0=1` |                         …(2) naturally
  `	herefore exp(x,0) = x^0` | from (1) and (2)


**Case 2.** `k eq 0`.

We want to prove that `exp(x,k)=x^k`. So, naturally `exp(x,k-1)`
supposed to be `x^{k-1}`. We will take this as the induction hypothesis.

                  . | .
  ------------------|------------------
  `exp(x,k-1) = x^{k-1}` | …(1) by induction hypothesis
  so, `x times x^{k-1} = x^{k-1+1}` | we know from arithmatic
  so, `x times x^{k-1} = x^k` | `k-1+1=k`
  so, `x times exp(x,k-1) = x^k` | from (1)
  `	herefore exp(x,k) = x^k` | from the function


**Factorial**
-------------

Factorial is another arithmetic function that is informally,
`x! = x times (x-1) times ... times 1` . We can define it
recursively. First, it has the only parameter, `x`. We know that `x` can
be at least `0` and `factorial(0)=1`. This is our base
part. Now in the recursion part it must contain
`factorial(x-1)`. Finally, we can multiply it with `x` to
make it equal to `factorial(x)`. See the proof later for a
better understanding.

Now our function looks like:
``` 
factorial(x)=
        1 , when `x=0`
        x times factorial(x-1), otherwise
``` 

Obviously, the code for the above function is:
```c
    int factorial(int x){
     if( x == 0 )
                    return 1;
            else
                    return x * factorial( x - 1 );
    }
```

<span>**Proof of correctness**</span>

Let us see whether the above function, `factorial(x)` is
really `x!`. We can prove it by using induction on `x`.

Case 1. `x=0`.

  `factorial(0) = 1 ` by definition, `0! = 1`
  ---------------------------------- -- -----------------------

Case 2. `x 
eq 0`.

We want to prove that `factorial(x)=x!`. So, naturally
`factorial(x-1)` supposed to be `(x-1)!`. We will take this
as the induction hypothesis.

             . | .
  -------------|----------------------------------------------------------------
  `factorial(x-1) = (x-1)!` | …(1) by induction hypothesis
  `(x-1)! = (x-1) 	times (x-2) 	times ... 	times 1` | expanded form
  so, `x 	times (x-1)! = x 	times (x-1) 	times (x-2) 	times ldots 	imes 1` | multiplying both side by `x`
  so, `x 	times factorial(x-1) = x!` | from (1)
  `	herefore factorial(x) = x!` | from the function


**Fibonacci**
-------------

Now our function looks like:
``` 
{hbox{em fibonacci}}(x)=begin{dcases*}
        0 & when `x=0`
        1 & when `x=1`
        {hbox{em fibonacci}}(x-1) + {hbox{em fibonacci}}(x-2) & otherwise
        end{dcases*}
``` 

Obviously, the code for the above function is:
```c
    int fibonacci(int x){
     if( x == 0 )
                    return 0;
            else if( x == 1 )
                    return 1;
            else
                    return fibonacci( x - 1 ) + 
                           fibonacci( x - 2 );
    }
```

**Addition**
------------

``` 
{hbox{em add}}(x,y)=begin{dcases*}
        x & when `y=0`
        suc({hbox{em add}}(x,y-1)) & otherwise
        end{dcases*}
``` 

Obviously, the code for the above function is:
```c
    int suc(int x){
            return x + 1;
    }

    int add(int x, int y){
     if( y == 0 )
                    return x;
            else
                    return suc( factorial( x - 1 ) );
    }
```

Write a recursive function to represent multiplication of two numbers by
using only addition. Write a recursive function to represent integer
division of two numbers by using only subtraction.

**Series**
----------

```
sum_{i=1}^ni^4 = 1^4 + 2^4 + ldots + n^4
``` 
Can be rewritten as:
``` 
{hbox{em sumOfQuad}}(n)=begin{dcases*}
        1 & when `n=1`
        {hbox{em exp}}(n,4) + {hbox{em sumOfQuad}}(n-1) & otherwise
        end{dcases*}
```
In C:
 
```c
    int sumOfQuad(int n){
     if( n == 0 )
                    return 0;
            else
                    return exp( n , 4 ) + 
                           sumOfQuad( n - 1 );
    }
```

### A Similar Series

A similar, in fact, more generalized series is:
```
sum_{i=1}^ni^k = 1^k + 2^k + ldots + n^k
``` 

It can be rewritten as:

``` 
{hbox{em sumOfPow}}(n,k)=begin{dcases*}
        1 & when `n=1`\n        {hbox{em exp}}(n,k) + {hbox{em sumOfPow}}(n-1,k) & otherwise
        end{dcases*}
``` 

It is converted into the program:

```c
    int sumOfPow(int n, int k){
     if( n == 0 )
                    return 0;
            else
                    return exp( n, k ) + 
                           sumOfPow( n - 1, k );
    }
```

### A Product Series

A general product series can be rewritten as:
```
prod_{i=1}^ni^k = 1^k times 2^k times ldots times n^k
``` 

See how we can get a product series from the previous series by only
changing the base case result and by replacing `+` by `times` in the
recursion part. Obviously, we should change the name of the function.

``` 
{hbox{em prodOfPow}}(n,k)=begin{dcases*}
        1 & when `n=1`
        {hbox{em exp}}(n,k) times {hbox{em prodOfPow}}(n-1,k) & otherwise
        end{dcases*}
``` 

Here is the code:
```c
    int prodOfPow(int n, int k){
     if( n == 0 )
                    return 1;
            else
                    return exp( n, k ) * 
                           prodOfPow( n - 1, k );
    }
```

### Natural Square Root

In this example, we will calculate the natural square root of a number
`n`. A natural square root is a natural number that is an integer and also
positive. We will examine a very easy function to do that. In this
function, `0` will first be tested if it is the square root of `n`. If
the square root of `n` is not a natural number, the smallest larger
value (`sqrt{n}+1`) will be accepted as an approximation. For each test
fails, the next natural number will be tested similarly.

The logic behind the condition `square(k) >= n` is that
`sqrt{n} = k` if and only if `k^2 = n` and `sqrt n+1 = k+1` and then
`(k+1)^2 > n`. Since the value of `k` is increasing from
`0` one by one, either `sqrt n ` of `sqrt n+1` will be
accepted as the natural square root of `n`.

```c
    int square(int x){
            return x*x;
    }
    int squareRootNatTest(int n, int k){
            if( square(k) >= n )
                    return k;
            else
                    return squareRootNatTest(n, k+1);
    }
    int squareRootNat(int n){
     return squareRootNatTest(n, 0);
    }
```

Can you explain how the code below determines a number as a prime?
```c
    int doesDivides(int x, int y){
        return (x % y) == 0;
    }
    int primeTest(int i, int n){
        if( i == n )
            return 1;
        else if( doesDivides(n, i) )
            return 0;
        else
            return primeTest(i+1, n);
    }
    int isPrime(int n){
        return primeTest(2, n);
    }
```

**Real Life Example**
---------------------

### Robot Move

<span>**Problem:**</span> Suppose we have a two-dimensional grid of `N`
columns and `M` rows. We name a cell of it in terms of its vertical and
horizontal index (`x`,`y`). We have a robot that can move either to one
cell left or one cell below. It cannot cross the border of the grid
indeed. Let’s place the robot at any cell in the grid and our current
challenge is to know the number of ways it can reach to the bottom-left
most corner, cell (`0`,`0`).

<span>**Solution:**</span> Lets imagine a grid of `N=9` columns and
`M=5` rows where the <span>robot</span> is placed at the cell (`6`,`3`).
The <span>goal</span> of its move is to reach the cell (`0`,`0`). It can
move only <span>downward</span> or <span>left</span>.

If the robot is placed in the cell (`0`,`0`), it needs `0` move to reach
the goal and hence trivially there is only one way to reach the goal.
From all the cells in the left-most column ((`0`,`1`), (`0`,`2`),
`ldots`), the robot can move only downward. So, from each of these
positions (where `x` is `0`), the robot has only one way to reach the
goal. Similarly, from all the cells at bottom-most row (where `y` is
`0`), it has only one way to reach the goal. From any other cell, the
path to reach the goal depends on its left and below adjacent cells. So,
the number of possible ways to reach the goal can be determined by
adding that of those two cells. So, our function is as follows:

``` 
{hbox{em ways}}(x,y)=begin{dcases*}
        1 & when `x=0` or `y=0`
        ways(x-1,y)+ways(x,y-1) & otherwise
        end{dcases*}
``` 

Now we can write our function in C very easily:

   <span>**Operator**</span> | <span>**Name**</span>
  ---------------------------|-----------------------
              `!` | Not
             `||` | Or (Logical)
            `&&` | And (Logical)


```c
    int ways(int x, int y){
     if( x == 0 || y == 0 )
                    return 1;
            else
                    return ways( x-1, y ) + ways( x, y-1 );
    }
```
