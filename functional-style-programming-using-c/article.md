Study Instruction {#study-instruction}
=================

How to Read This Book
---------------------

- Several times a single function will be written using subscript. If
    the name of the function has an intuitive meaning, we should assume
    that the functions with same names and different subscripts are the
    same in meaning but different in definition.

- The line number (at the left of a code) is only for reference. Never
    write it in actual coding.

- In most mathemetical textbook, a function is named $f$, $g$, etc.
    (e.g. $f(x) = x^2$). Here most times we will name a function
    according to its meaning (e.g. $square(x) = x^2$).

- Since it is impossible to write subscript in coding (programming),
    it is converted into normal text especially when it is a part
    of name.

- Colors in the code indicates different types of texts used
    in programming.

How to Write Program
--------------------

- Use <span>*Code::Blocks IDE*</span>, a free tool where you can write
    C program, compile it and execute it without hassle. You can use
    other IDE too.

- Give meaningful name to functions and parameters.

Introduction _Good to read it_
========================================================================================================

From the elementary school, we learned what is a mathematical function and how to write it. For
example - 
```math
f(x)=x^2
``` 
is a simple square function of arithmetic.

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

(n1) <span>$in_1$</span>; (n2) \[below=of n1,yshift=7mm\] ; (n3)
\[below=of n2,yshift=7mm\] <span>$in_2$</span>; (p) \[minimum
height=3cm,right=of n2, text width=4cm,text centered\] <span> <span>Let
$x$ be $in_1 / in_2$\
Let $y$ be $x \times in_2$\
Let $out$ be $in_1 - y$</span></span>; ; (m) \[right=of p\]
<span>$out$</span>; ; ; ; (n1) – (p); (n3) – (p); (p) – (m);

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

Composition <span><span><span style="font-variant:small-caps;">— Add the programs —</span></span></span>
========================================================================================================

**Square of Distance**
----------------------

(x1) <span>$x_1$</span>; (y1) \[right=of x1,xx\] <span>$y_1$</span>;
(x2) \[right=of y1,xx\] <span>$x_2$</span>; (y2) \[right=of x2,xx\]
<span>$y_2$</span>; (sub1) \[below=of x1,yy,xshift=7mm\]
<span>$-$</span>; (sub2) \[below=of x2,yy,xshift=7mm\] <span>$-$</span>;
(sq1) \[below=of sub1,yy\] <span>$square$</span>; (sq2) \[below=of
sub2,yy\] <span>$square$</span>; (plus) \[below=of sq1,yy,xshift=14mm\]
<span>$+$</span>; (eq) \[below=of plus,yy\] <span>$=$</span>; (poly)
\[below=of eq,yy\] <span>$squareOfDistance_1(x)$</span>; (x1) – (sub1);
(x2) – (sub1); (y1) – (sub2); (y2) – (sub2); (sub1) – (sq1); (sub2) –
(sq2); (sq1) – (plus); (sq2) – (plus); (plus) – (eq); (poly) – (eq);

A function can be composed using other functions. It often simplifies
the expression. For example, the function, $squareOfDistance$ can be
simplified if we use the function $square$. It is shown here –
$$square(x) = x \times x$$
$$squareOfDistance_1(x_1,y_1,x_2,y_2)=square(x_1-x_2)+square(y_1-y_2)$$

Let us write it in programming language.

```c
    int square(int x){
            return x * x;
    }
    int squareOfDistance1(int x1, int y1, 
                          int x2, int y2){
     return square(x1-x2) + square(y1-y2);
    }
```

Here we present a pictorial representation of the above function
<span>*squareOfDistance1*</span>.

\(b) at (0,0) ; (xa) at (1.45,.70) <span>$x$</span>; at (-1.5,.75)
<span>*square*</span>; (dr) at (0,0) <span>$*$</span>; (xa.south) |-
(dr); (xa.west) -| (dr); (dr.south) – (0,-1);

\(b) at (5,0) ; (xb) at (6.45,.70) <span>$x$</span>; at (3.5,.75)
<span>*square*</span>; (dr) at (5,0) <span>$*$</span>; (xb.south) |-
(dr); (xb.west) -| (dr); (dr.south) – (5,-1);

\(b) at (2.5,3) ; (y2) at (4.95,3.70) <span>$y_2$</span>; (x2) at
(3.9,3.70) <span>$x_2$</span>; (y1) at (2.85,3.70) <span>$y_1$</span>;
(x1) at (1.8,3.70) <span>$x_1$</span>; (plus) at (2.5,2.75)
<span>$+$</span>; (minus1) \[left=of plus\] <span>$-$</span>; (minus2)
\[right=of plus\] <span>$-$</span>; (x1.west) -| (minus1); (x2.south) –
++(0,-.2) – ++(-2.5,0) |- (minus1); (y1.south) – ++(0,-.1) – ++(.75,0)
|- (minus2); (y2.south) |- (minus2); (minus1) – ++(0,-.75); (minus2) –
++(0,-.75); at (0,4.2) <span>*squareOfDistance1*</span>;

(1.45,.95) – ++(0,.4) – ++(-.67,0) – ++(0,.67); (6.45,.95) – ++(0,.4) –
++(-2.23,0) – ++(0,.67);

(0,-1) – node\[xshift=7mm\]<span>return</span> ++(0,-.5) –
node\[yshift=-2mm\]<span>$x*x$</span> ++(-2.5,0) – ++(0,3.1) – ++(4.2,0)
|- (plus); (5,-1) – node\[xshift=7mm\]<span>return</span> ++(0,-.5) –
node\[yshift=-2mm\]<span>$x*x$</span> ++(-2.5,0) – ++(0,3) – (plus);
(plus.east) – ++(.3,0) – ++(0,-1.1) – ++(3,0) – ++(0,3); (x1.north) –
++(0,.7); (y1.north) – ++(0,.7); (x2.north) – ++(0,.7); (y2.north) –
++(0,.7);

Here we can see that the function <span>*squareOfDistance1*</span> is
using two copies of the function <span>*square*</span>. It is very
important to understand.

**Some Previous functions in New Form**
---------------------------------------

In this section, we will show some functions presented in previous
chapter in new form using composition.

### The Function $Poly2$

The old function was defined as – $$Poly2(x) = x^4 + x^2 + 1$$

Now we will re-define it as $Poly2_1$ using the function $square$.
$$square(x) = x \times x$$
$$Poly2_1(x) = square(square(x)) + square(x) + 1$$

In programming, we can write it as –

    int square(int x){
            return x * x;
    }

    int poly21(int x){
            return square(square(x)) + square(x) + 1;
    }

### The Function sumOfQube

The old function was defined as – $$sumOfQube_1(x) = (x(x+1)/2)^2$$

Now we will re-define it as $sumOfQube$ using the function $square$ and
$sum$. $$square(x) = x \times x$$ $$sum(x) = (square(x) + x) / 2$$
$$sumOfQube(x) = square(sum(x))$$

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

Prove that – $x^3 = x \times square(x)$
$x^8 = square(square(square(x)))$ Simplify them by composing other
functions and write programs accordingly – $qube_c(x) = x^3$
$toPower8(x) = x^8$ $toPower4(x) = x^4$

Write programs for – $parabola(a,b,c,x) = ax^2 + bx + c$
$square_p(x) = parabola(1,0,0,x)$ $qube_p(x) = parabola(x,0,0,x)$

Write program for $toPower4_p(x) = parabola(parabola(1,0,0,x),0,0,x)$
and prove that $toPower4_p(x) = x^4$.

Condition <span><span><span style="font-variant:small-caps;">— Let’s our program decide —</span></span></span>
==============================================================================================================

From this chapter, we will not write same function twice.

**General Idea**
----------------

in a function, we may get different result based on different conditions
on the parameters. For example, the function <span>*absolute*</span>
negates its parameter if it is negative so that it can be turned into
positive (negative of negative is positive) to produce the result.
Otherwise (when parameter is already positive), it returns back the
result exactly same as the parameter. Thus, the function can be written
as – $$abs(x) = \begin{dcases*}
        x & when $x \geq 0$\\
        -x & otherwise
        \end{dcases*}$$

compute based on conditions of parameters.

The general idea of a conditional function is as follows –

$$f(x) = \begin{dcases*}
        e_1 & when $1^{st}$ condition satisfies\\
        e_2 & when $2^{nd}$ condition satisfies\\
        e_3 & when $3^{rd}$ condition satisfies\\
        \dots & \dots \\
        e_n & otherwise
        \end{dcases*}$$

The corresponding code is –

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

Now we will present some functions those use conditions.

**if …else …**
--------------

### Absolute

$$abs(x) = \begin{dcases*}
        x & when $x \geq 0$\\
        -x & otherwise
        \end{dcases*}$$

    int abs(int x){
            if( x >= 0 )
                    return x;
            else
                    return -x;
    }

### Filter

$$filter_{LT}(x,y) = \begin{dcases*}
        x & when $x \geq y$\\
        0 & otherwise
        \end{dcases*}$$

    int filterLT(int x, int y){
            if( x >= y )
                    return x;
            else
                    return 0;
    }

if …else …+ Composition
-----------------------

By composing functions, we can construct many familiar functions.
Although they are not always more efficient, for obvious reason, we will
do it.

Here the function <span>*abs*</span> is rewritten as –
$$switcher(a,b,c,d) = \begin{dcases*}
        c & when $a \geq b$\\
        d & otherwise
        \end{dcases*}$$ $$abs_1(x) = switcher(x, 0, x, -x)$$

    int switcher(int a,int b,int c,int d){
            if(a >= b)
                    return c;
            else
                    return d;
    }
    int abs1(int x){
            switcher(x, 0, x, -x);
    }

### Tution Fee Calculator

Lets think a problem –

Calculate your total tuition fee due based on the condition that if your
due of previous semester tuition fee is more that 1000 currency, your
total fee to pay this semester will be double of previous due plus the
fee of current semester. We can write a function to describe it
mathematically.

$$\hbox{\em tftp}(p, c)=\begin{dcases*}
        2\times p + c & when $p > 1000$\\
        p + c & otherwise
        \end{dcases*}$$

If we use the function $switcher$, we can rewrite it as –

$$\hbox{\em tftp}_1(p, c) = 
switcher(p, 1001, 2 \times p, p) + c$$ or, $$\hbox{\em tftp}_2(p, c) = 
switcher(p, 1001, p, 0) + p + c$$

In programming, the functions are –

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

Well, the parameters <span>p</span> and <span>c</span> does not mean
anything. So, often it misleads us if we use meaningless names. It is
also important to be able to read them easily. We can take different
approaches to avoid confusions and increase readibility. They are shown
as the function <span>*tftp2*</span> is rewritten –

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

or, in the best style –

    int tftp2(int previousDue, int currentFee){
            return switcher(previousDue, 1000,
                            previousDue, 0)  
                            + previousDue 
                            + currentFee;
    }

or, in a good style –

    int tftp2(int preDue, int curFee){
            return switcher(preDue, 1001, preDue, 0) 
                   + preDue + curFee;
    }

or, we can write a word to keep it small and yet understandable.

    int tftp2(int due, int fee){
            return switcher(due,1001,due,0)+due+fee;
    }

**if …else if …else …**
-----------------------

### Grade Letter (in numeric)

In this example, we will solve a problem of determining the grade letter
based on the total mark in an examination. The grade distribution is
given at the right side.

         Range Grade letter In numeric
  -------------------- -------------- ------------
         90-100 A 1
         80-89 B 2
         70-79 C 3
         60-69 D 4
         40-59 E 5
   0-39, $<0$, $>100$ F 6

The function that solves the problem is –
$$getGrade(mark) = \begin{dcases*}
        1 & when $mark \geq 90$ and $mark \leq 100$\\
        2 & else when $mark \geq 80$\\
        3 & else when $mark \geq 70$\\
        4 & else when $mark \geq 60$\\
        5 & else when $mark \geq 40$\\
        6 & otherwise
        \end{dcases*}$$ Hence, the code for the above function is –

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

In the above example, the function gives us a numeric value that in our
mind represents a grade. But it is often necessary to give user a direct
result instead of a numerical representation. If the function can
generate result with letter grades directly, it will be definitely more
understandable as shown here – $$getGradeLetter(mark) = \begin{dcases*}
        \hbox{'A'} & when $mark \geq 90$ and $mark \leq 100$\\
        \hbox{'B'} & else when $mark \geq 80$\\
        \hbox{'C'} & else when $mark \geq 70$\\
        \hbox{'D'} & else when $mark \geq 60$\\
        \hbox{'E'} & else when $mark \geq 40$\\
        \hbox{'F'} & otherwise
        \end{dcases*}$$ Since the letter grades are actually letters or
characters, we need to change the data type of the function to ‘char’.
‘char’ type corresponds to the characters. In other words, ‘char’ is the
set of all characters or visible symbols a computer can show.

  <span>**Type**</span> <span>**Nature of Data**</span>
  ----------------------- ---------------------------------
  int integer
  long large integer
  long long very large integer
  float real number
  double large real number
  char character

The code for the above function is –

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

\[node distance = 2cm, auto\] (init) <span>mark</span>; (c1)
<span>$90 \leq mark \leq 100$</span>; (c2) <span>$mark \geq 80$</span>;
(c3) <span>$mark \geq 70$</span>; (c4) <span>$mark \geq 60$</span>; (c5)
<span>$mark \geq 40$</span>; (o1) <span>’A’</span>; (o2)
<span>’B’</span>; (o3) <span>’C’</span>; (o4) <span>’D’</span>; (o5)
<span>’E’</span>; (o6) <span>’F’</span>; (init) – (c1); (c1) – node
<span>yes</span>(o1); (c2) – node <span>yes</span>(o2); (c3) – node
<span>yes</span>(o3); (c4) – node <span>yes</span>(o4); (c5) – node
<span>yes</span>(o5); (c5) – node <span>no</span>(o6); (c1) – node
<span>no</span> (c2); (c2) – node <span>no</span> (c3); (c3) – node
<span>no</span> (c4); (c4) – node <span>no</span> (c5);

**switch …case …**
------------------

### Item Price {#itemprice}

In this example, we will solve a problem of getting prices of different
items in an restaurant. We will determine an item by its item no. The
prices for different item numbers are given at the right.

      Item Item No. Price
  ------------- ---------- --------
   Fish Curry 1 60.00
    Fish Fry 2 80.00
   Beef Curry 3 100.50
    Beef Vuna 4 130.75
     Chicken 5 85.00
      Rice 6 15.25
      Water 7 20.00
   Cold Drinks 8 15.00

The function that solves the problem is –
$${\hbox{\em getPrice}}({\hbox{\em item}}) = \begin{dcases*}
        60.00 & when {\em item} $= 1$\\
        80.00 & when {\em item} $= 2$\\
        100.50 & when {\em item} $= 3$\\
        130.75 & when {\em item} $= 4$\\
        85.00 & when {\em item} $= 5$\\
        15.25 & when {\em item} $= 6$\\
        20.00 & when {\em item} $= 7$\\
        15.00 & when {\em item} $= 8$\\
        0 & otherwise 
        \end{dcases*}$$ Hence, the code for the above function is –

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

  <span>**Type**</span> <span>**Range**</span>
  ----------------------- ---------------------------
  int/long $-2^{31}$ to $2^{31} - 1$
  long long $-2^{63}$ to $2^{63} - 1$
  float 1.2E-38 to 3.4E+38
  double 2.3E-308 to 1.7E+308
  long double 3.4E-4932 to 1.1E+4932
  char -128 to 127

Note: $2^{31}=2,147,483,648$ and

$2^{63}=9,223,372,036,854,775,808$

For a realistic approach, we can add a simple function with it that
calculates total price of an item for a quantity -
$${\hbox{\em calcTotalPrice}}({\hbox{\em item}}, {\hbox{\em quantity}}) = {\hbox{\em quantity}} \times {\hbox{\em getPrice}}({\hbox{\em item}})$$

The code:

    float calcTotalPrice(int item, int quantity){
            return quantity * getPrice( item );
    }

Recursions <span><span><span style="font-variant:small-caps;">— Easy tricks to repeat —</span></span></span>
============================================================================================================

Recursion is a technique that enhances capabilities of programs very
much. For example, if we want to write an exponential function
$exp(x,k)=x^k$, we understand that $x$ have to be multiplied to itself
$k$ times. We can understand $x^k$ operation from our childhood
mathematics. But unfortunately C programming language does not have
operator for exponential operation. We can also understand $k$ times and
we can repeat this multiplication by counting $1$ to $k$.

But how to write the function using multiplication only? We can write
$exp(x,k)=x \times x \times \dots \times x$ ($k$ times). If we knew $k$
is $2$, the expression becomes $exp(x,2) = x \times x$, if it is $3$,
the expression is $exp(x, 3) = x \times x \times x$ and so on. But when
$k$ is unknown, then it is difficult to write in very clear way. Most
importantly, C programming does not support any expression like
‘$x \times x \times \dots \times x$ ($k$ times)’.

**Intuition of Recursive definition**
-------------------------------------

![image](rectree4){height="4cm"} ![image](kidney-tree){height="3cm"}

We use recursion to express such functions or programs. Recursion is a
special kinds of composition where the definable function itself is
composed with something else to define itself. It may heard weird or
twisted at the first time. We will see some analogous definition in our
society to understand its power.

But just imagine a simple definition of <span><span>
*human*</span></span> (according to Quran and Bible) - ‘Adam and Eve
were <span><span> *human*</span></span> and any living mammal that borns
from another <span><span> *human*</span></span> is a <span><span>
*human*</span></span>’. This definition certainly covers all the humans.
Here <span><span> *human*</span></span> is defined from the definition
of another human and it is called recursion or induction. In recursive
(or inductive) defintion, there must be at least one base point for
example ‘Adam was <span><span> *human*</span></span> and Eve was
<span><span> *human*</span></span>’.

Let us take another example - definition of natural numbers. Suppose we
have all kinds of numbers on our knowledge including real numbers,
complex numbers, etc.. We need to define natural number. Here is the
plain definition - $0$ is a natural number and $n+1$ is a natural number
if $n$ is a natural number. We can use it to write a function to know
whether a number is natural –
$${\hbox{\em isNatural}}(x)=\begin{dcases*}
        {\hbox{\em true}}& when $x=0$\\
        {\hbox{\em false}}& when $x<0$\\
        {\hbox{\em isNatural}}(x-1) & otherwise
        \end{dcases*}$$

Similarly we can write another function to distinguish an animal as
human – $${\hbox{\em isHuman}}(x)=\begin{dcases*}
        {\hbox{\em true}}& when $x=$ {\em Adam}\\
        {\hbox{\em true}}& when $x=$ {\em Eve}\\
        {\hbox{\em false}}& when $x=$ {\em Amoeba}\\
        {\hbox{\em isHuman}}({\hbox{\em mother of }}x) & otherwise
        \end{dcases*}$$

It says, $x$ is a human if $x$ is <span>*Adam*</span> or
<span>*Eve*</span>. $x$ is not human is $x$ is <span>*Amiba*</span>.
Besides $x$ is also human if and only if mother of $x$ is also human.
Well, in the real process, to justify an animal as human, we need to
know the total family tree from <span>*Adam*</span> and
<span>*Eve*</span> which is very unrealistic. But the definition itself
is very intuitive.

In mathematics and computer science, this recursive style of definition
often expresses a fact in a very simple and intuitive way, which helps
to formalize an algorithm or program in very effective way.

<span> To create a recursive definition of a function, we need to forget
how it really works. We simply need to know if the definition is correct
mathematically. </span>

For an easy construction of the definition, here we would like to
present a tips. We have to remember that the function has one (or more)
driving parameter(s).

\[node distance=2mm\] (fun) at (.8,0)
<span><span>*exp*</span>$($</span>; (p1) at (1.5,0) <span>$n$,</span>;
(p2) at (2,0) <span>$x$</span>; (eq) at (3,0) <span> $)=\begin{dcases*}
          & \\
        & 
        \end{dcases*}$ </span>; (baseret) at (3.5,.5) <span>$1$</span>;
(base) at (7.5,.5) <span>when $x=0$</span>; (rec1) at (3.5,-.5)
<span>$x \times$</span>; (rec2) at (4.2,-.5)
<span><span>*exp*</span>$($</span>; (rec3) at (4.7,-.54)
<span>$n,$</span>; (rec4) at (5.5,-.5) <span>$x-1$</span>; at (6.0,-.5)
<span>$)$</span>; (otherwise) at (7.4,-.5) <span>otherwise</span>;
(funref) at (.8,-1.5) <span>function name</span>; (p1ref) at (1.5,2)
<span>ordinary parameter</span>; (p2ref) at (2,-2) <span>driving
parameter</span>; (baseref) \[above=of base,yshift=.5cm\] <span>base
case</span>; (baseretref) \[above=of baseret\] <span>result for base
case</span>; (rec1ref) \[below=of rec1,yshift=-2.0cm\] <span>computation
with something</span>; (rec2ref) \[below=of rec2,yshift=-1.5cm\]
<span>recursion</span>; (rec4ref) \[below=of rec4,yshift=-.2cm,text
width=2cm,text centered\] <span>driving parameter is reduced 1
step</span>; (othref) \[below=of otherwise,yshift=-1.5cm\] <span>other
cases</span>; (funref) – (fun); (p1ref) – (p1); (p2ref) – (p2);
(baseref) – (base); (baseretref) – (baseret); (rec1ref) – (rec1);
(rec2ref) – (rec2); (rec4ref) – (rec4); (othref) – (otherwise);

Base cases part

: \
    Need to determine the value of the function for the base case(s) of
    the driving parameter(s). For example, ${\hbox{\em exp}}(n,x)=1$
    when $x=0$.

Recursion part

: - The function itself will be appeared with smaller (typically
        1 step) value or size of the driving parameter(s). For example,
        ${\hbox{\em exp}}(x-1)$. This is called induction
        hypothesis (HI).

    - The induction hypothesis often needs to be computed with
        something else to be equal to the original definition. For
        example, ${\hbox{\em exp}}(x)=x \times {\hbox{\em exp}}(x-1)$.

    - Other parameters remain the same in recursion call inside
        the body.

If we follow the above steps, it will help us to construct recursive
defintion. We will explore some examples soon. In some examples, we will
see how the function is constructed from the above tips. We will also
see whether definition is correct.

**Exponential**
---------------

We will define $(x,k)$. Here $x$ is the base and $k$ is the power. So,
$k$ is the driving parameter. Now we know the base case when $k=0$ -
$(x,0)=1$. Next, we will have to construct the recursion part. First, we
know that the expression $(x,k-1)$ exists in this part because the 1
step smaller value of $k$ is $k-1$. $x$ will remain same. Now it is
necessary to multiply it to $x$ to be equal to $(x,k)$. Therefore, the
full expression becomes $(x,k) = x \times $$(x,k-1)$. Now together the
function is looks like –

$${\hbox{\em exp}}(x,k)=\begin{dcases*}
        1 & when $k=0$\\
        x \times {\hbox{\em exp}}(x,k-1) & otherwise
        \end{dcases*}$$

Obviously, the code for the above function is –

    int exp(int x, int k){
     if( k == 0 )
                    return 1;
            else
                    return x * exp( x, k-1 );
    }

<span>**Proof of correctness**</span>

Let us see whether the above function, $exp(x,k)$ is really $x^k$. We
can prove it by using induction on $k$.

Case 1. $k=0$.

  ----------------------------- ------ -------------------
  $exp(x,0) = 1 $ …(1) from the function
  $x^0=1$ …(2) naturally
  $\therefore exp(x,0) = x^0$ from (1) and (2)
  ----------------------------- ------ -------------------

Case 2. $k \neq 0$.

We want to prove that $exp(x,k)=x^k$. So, naturally $exp(x,k-1)$
supposed to be $x^{k-1}$. We will take this as the induction hypothesis.

  ------------------------------------ ------ -------------------------
  $exp(x,k-1) = x^{k-1}$ …(1) by induction hypothesis
  so, $x \times x^{k-1} = x^{k-1+1}$ we know from arithmatic
  so, $x \times x^{k-1} = x^k$ $k-1+1=k$
  so, $x \times exp(x,k-1) = x^k$ from (1)
  $\therefore exp(x,k) = x^k$ from the function
  ------------------------------------ ------ -------------------------

**Factorial**
-------------

Factorial is another arithmetic function that is informally,
$$x! = x \times (x-1) \times \ldots \times 1$$. We can define it
recursively. First, it has the only parameter, $x$. We know that $x$ can
be at least $0$ and ${\hbox{\em factorial}}(0)=1$. This is our base
part. Now in the recursion part it must contain
${\hbox{\em factorial}}(x-1)$. Finally, we can multiply it with $x$ to
make it equal to ${\hbox{\em factorial}}(x)$. See the proof later for a
better understanding.

Now our function looks like –
$${\hbox{\em factorial}}(x)=\begin{dcases*}
        1 & when $x=0$\\
        x \times {\hbox{\em factorial}}(x-1) & otherwise
        \end{dcases*}$$

Obviously, the code for the above function is –

    int factorial(int x){
     if( x == 0 )
                    return 1;
            else
                    return x * factorial( x - 1 );
    }

<span>**Proof of correctness**</span>

Let us see whether the above function, ${\hbox{\em factorial}}(x)$ is
really $x!$. We can prove it by using induction on $x$.

Case 1. $x=0$.

  ${\hbox{\em factorial}}(0) = 1 $ by definition, 0! = 1
  ---------------------------------- -- -----------------------

Case 2. $x \neq 0$.

We want to prove that ${\hbox{\em factorial}}(x)=x!$. So, naturally
${\hbox{\em factorial}}(x-1)$ supposed to be $(x-1)!$. We will take this
as the induction hypothesis.

  ---------------------------------------------------------------------------- ------ ------------------------------
  ${\hbox{\em factorial}}(x-1) = (x-1)!$ …(1) by induction hypothesis
  $(x-1)! = (x-1) \times (x-2) \times \ldots \times 1$ expanded form
  so, $x \times (x-1)! = x \times (x-1) \times (x-2) \times \ldots \times 1$ multiplying both side by $x$
  so, $x \times {\hbox{\em factorial}}(x-1) = x!$ from (1)
  $\therefore {\hbox{\em factorial}}(x) = x!$ from the function
  ---------------------------------------------------------------------------- ------ ------------------------------

**Fibonacci**
-------------

Now our function looks like –
$${\hbox{\em fibonacci}}(x)=\begin{dcases*}
        0 & when $x=0$\\
        1 & when $x=1$\\
        {\hbox{\em fibonacci}}(x-1) + {\hbox{\em fibonacci}}(x-2) & otherwise
        \end{dcases*}$$

Obviously, the code for the above function is –

    int fibonacci(int x){
     if( x == 0 )
                    return 0;
            else if( x == 1 )
                    return 1;
            else
                    return fibonacci( x - 1 ) + 
                           fibonacci( x - 2 );
    }

**Addition**
------------

$${\hbox{\em add}}(x,y)=\begin{dcases*}
        x & when $y=0$\\
        suc({\hbox{\em add}}(x,y-1)) & otherwise
        \end{dcases*}$$

Obviously, the code for the above function is –

    int suc(int x){
            return x + 1;
    }

    int add(int x, int y){
     if( y == 0 )
                    return x;
            else
                    return suc( factorial( x - 1 ) );
    }

Write a recursive function to represent multiplication of two numbers by
using only addition. Write a recursive function to represent integer
division of two numbers by using only subtraction.

**Series**
----------

$$\sum_{i=1}^ni^4 = 1^4 + 2^4 + \ldots + n^4$$

can be rewritten as – $${\hbox{\em sumOfQuad}}(n)=\begin{dcases*}
        1 & when $n=1$\\
        {\hbox{\em exp}}(n,4) + {\hbox{\em sumOfQuad}}(n-1) & otherwise
        \end{dcases*}$$

    int sumOfQuad(int n){
     if( n == 0 )
                    return 0;
            else
                    return exp( n , 4 ) + 
                           sumOfQuad( n - 1 );
    }

### A Similar Series

A similar, in fact, more generalized series is –
$$\sum_{i=1}^ni^k = 1^k + 2^k + \ldots + n^k$$ It can be rewritten as –
$${\hbox{\em sumOfPow}}(n,k)=\begin{dcases*}
        1 & when $n=1$\\
        {\hbox{\em exp}}(n,k) + {\hbox{\em sumOfPow}}(n-1,k) & otherwise
        \end{dcases*}$$ It is converted into the program –

    int sumOfPow(int n, int k){
     if( n == 0 )
                    return 0;
            else
                    return exp( n, k ) + 
                           sumOfPow( n - 1, k );
    }

### A Product Series

A general product series can be rewritten as –
$$\prod_{i=1}^ni^k = 1^k \times 2^k \times \ldots \times n^k$$

See how we can get a product series from the previous series by only
changing the base case result and by replacing $+$ by $\times$ in the
recursion part. Obviously, we should change the name of the function.

$${\hbox{\em prodOfPow}}(n,k)=\begin{dcases*}
        1 & when $n=1$\\
        {\hbox{\em exp}}(n,k) \times {\hbox{\em prodOfPow}}(n-1,k) & otherwise
        \end{dcases*}$$

Here is the code –

    int prodOfPow(int n, int k){
     if( n == 0 )
                    return 1;
            else
                    return exp( n, k ) + 
                           prodOfPow( n - 1, k );
    }

### Natural Square Root

In this example, we will calculate the natural square root of a number
$n$. A natural square root is a natural number that is integer and also
positive. We will examine a very easy function to do that. In this
function, $0$ will first be tested if it is the square root of $n$. If
the square root of $n$ is not a natural number, the smallest larger
value ($\sqrt{n}+1$) will be accepted as an approximation. For each test
fails, the next natural number will be tested similarly.

The logic behind the condition <span>square(k) >= n</span> is that
$\sqrt{n}=k$ if and only if $k^2=n$ and $\sqrt n+1=k+1$ and then
$(k+1)^2 > n$. Since the value of <span>k</span> is increasing from
<span>0</span> one by one, either $\sqrt n $ of $\sqrt n+1$ will be
accepted as the natural square root of $n$.

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

Can you explain how the code below determines a number as a prime?

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

**Real Life Example**
---------------------

### Robot Move

<span>**Problem:**</span> Suppose we have a two dimensional grid of $N$
columns and $M$ rows. We name a cell of it in terms of its vertical and
horizontal index ($x$,$y$). We have a robot that can move either to one
cell left or one cell below. It cannot cross the border of the grid
indeed. Let’s place the robot at any cell in the grid and our current
challenge is to know the number of ways it can reach to the bottom-left
most corner, cell ($0$,$0$).

<span>**Solution:**</span> Lets imagine a grid of $N=9$ columns and
$M=5$ rows where the <span>robot</span> is placed at the cell ($6$,$3$).
The <span>goal</span> of its move is to reach the cell ($0$,$0$). It can
move only <span>downward</span> or <span>left</span>.

If the robot is placed at the cell ($0$,$0$), it needs $0$ move to reach
the goal and hence trivially there is only one way to reach the goal.
From all the cells in the left-most column (($0$,$1$), ($0$,$2$),
$\ldots$), the robot can move only downward. So, from each of these
positions (where $x$ is $0$), the robot has only one way to reach the
goal. Similarly, from all the cells at bottom-most row (where $y$ is
$0$), it has only one way to reach the goal. From any other cell, the
path to reach the goal depends on its left and below adjacent cells. So,
the number of possible ways to reach the goal can be determined by
adding that of those two cells. So, our function is as follows:

$${\hbox{\em ways}}(x,y)=\begin{dcases*}
        1 & when $x=0$ or $y=0$\\
        ways(x-1,y)+ways(x,y-1) & otherwise\\
        \end{dcases*}$$

Now we can write our function in C very easily:

   <span>**Operator**</span> <span>**Name**</span>
  --------------------------- -----------------------
              $!$ Not
             $||$ Or (Logical)
            $\&\&$ And (Logical)

<span>lll</span> ! True & = & False\
! False & = & True\
\
True || A & = & True\
False || A & = & A\
\
True && A & = & A\
False && A & = & False

    int ways(int x, int y){
     if( x == 0 || y == 0 )
                    return 1;
            else
                    return ways( x-1, y ) + ways( x, y-1 );
    }

Program Execution <span><span><span style="font-variant:small-caps;">— Run your program —</span></span></span>
==============================================================================================================

**main**
--------

Whatever function we write to compute something, we must need a special
function <span><span> **main**</span></span> to really execute it on
computer. Because computer knows that the <span><span>
**main**</span></span> function is the starting point of any program.

Here is the simple version of <span><span> **main**</span></span>
function –

    int main(){

    }

We can call any other function from the function <span><span>
**main**</span></span>. For example -

    int f(){

    }
    int main(){
      f();
    }

**Required Software**
---------------------

We have simply two options –

We can download an Integrated Development Environment (IDE) such as
code::blocks, Dev C++, etc. They can provide us a nice place where we
can write the program, compile it and with just one click, run the
program.

The second way is to download and install gcc. We can write programs
using any text editing program like, notepad, notepad++, emacs, etc. In
console, terminal or command prompt, we can change our directory where
we saved our source code. Next, if our source file name is source.c, we
can provide command <span><span> **gcc source.c**</span></span> and it
will generate a file <span><span> **a.exe**</span></span>. Now give the
command <span><span> **gcc a.exe**</span></span> to run the program.

Input/Output <span><span><span style="font-variant:small-caps;">— Using keyboard and monitor —</span></span></span>
===================================================================================================================

**Input**
---------

Basic input means receiving keystrokes from Keyboard. Keystrokes can be
received as a single character or as a number.

A typical example of receiving input is –

    #include<stdio.h>
    int main(){
     int a;
            scanf("%d", &a);
    }

Here,

- <span>stdio.h</span> is a library (header) file that contains
    necessary definition of the function `scanf`.

- `a` is an integer type variable.

- `scanf` is the function that helps to receive the input from
    standard input device (by default, it is key baord).

- `%d` is used so that our input is treated as an integer.

- `&a` means the address of `a`.

The function `scanf` receives the input as integer and keeps it at the
memory address which is regarded as `a`.

##### Input As Other Type

`char` type –

    #include<stdio.h>
    int main(){
     char letter;
            scanf("%c", &letter);
    }

Here –

- `%c` is used so that our input is treated as an character. It is
    called the format specifier.

- `&letter` means the address of `letter` that is a `char` type
    of variable.

The function `scanf` receives the input as a character and keeps it at
the memory address which is regarded as `letter`.

Here format specifier of more input types are presented.

   **Data Type** **Format Specifier** **Description**
  --------------- ---------------------- ----------------------------------------
        int “%d” Decimal Integer
        int “%i” Integer
        int “%x” Hexadecimal Integer
        int “%o” Octal Integer
        int “%u” only Positive Number
       long “%ld” Long Decimal Integer
     long long “%lld” Very Long Decimal Integer
       char “%c” Characters
       float “%f” Single Precision Floating Point Number
      double “%lf” Double Precision Floating Point Number
     char\[\] “%s” String of Characters
        \* “%p” Pointer Address

##### More Input Together

We can take more than one input in a same statement.

    #include<stdio.h>
    int main(){
            long id;
     char type;
            scanf("%l %c", &id, &type);
    }

Here two inputs are received in two different format.

- `%c` is used so that our input is treated as an character.

- `&letter` means the address of `letter` that is a `char` type
    of variable.

The function `scanf` receives the input as a character and keeps it at
the memory address which is regarded as `letter`.

**Output**
----------

Basic output means showing texts to the monitor through the console or
terminal or command prompt.

It is important to know that

A string is a sequence of characters and is enclosed by `"`. It is used
to represent an arbitrary text.

A typical example of printing output is –
$${\hbox{\em printTest}}()={\hbox{\em printf}}(\hbox{"It is my first output to monitor."})$$

    #include<stdio.h>
    int printTest(){
            return 
            printf("It is my first output to monitor.");
    }
    int main(){
            printTest();
    }

Here,

- <span>stdio.h</span> is a library (header) file that contains
    necessary definition of the function `printf`.

- `printf` is the function that helps to print our desired output to
    the standard output device (by default, it is monitor or terminal).

- `` is used to enclose what we intend to show on the monitor.

### Output of the Variables

Output using `printf` also includes values of the variables. An example
of formatted output is presented here –

    #include<stdio.h>
    int inputInt(){
            int x;
            scanf("%d", &x);
            return x;
    }
    int printSuc(int z){
            return printf("%d + 1 is %d", z, z + 1);
    }
    int main(){
            printSuc( inputInt() );
    }

The above program can be written in traditional way as –

    #include<stdio.h>
    int main(){
            int x;
            scanf("%d", &x);
            printf("%d + 1 is %d",x, x + 1);
    }

But, we think that the former one is more elegant because it can help us
to write complex program in simple way. The function actually wraps a
complex statement and gives us a simple statement. It also gives us a
good pattern or structure that can be used to write large programs in an
easy way. Here we present a better example.

    #include<stdio.h>
    int iInt(){
        int x;
        scanf("%d", &x);
        return x;
    }
    int square(int x){
        return x * x;
    }
    int calcDistance(int x1, int y1, int x2, int y2){
        return square(x1-x2) + square(y1-y2);
    }
    int getDistance(){
        return calcDistance(iInt(),iInt(),
                            iInt(),iInt());
    }
    int getSmaller(int x, int y){
        if ( x < y )
            return x;
        else
            return y;
    }
    int main(){
        printf("Smaller Distance is %d", 
            getSmaller( getDistance(), getDistance() ); 
    }

We can also write the same program in traditional way –

    #include<stdio.h>
    int square(int x){
            return x * x;
    }
    int calcDistance(int x1, int y1, int x2, int y2){
            return square(x1-x2) + square(y1-y2);
    }
    int getSmaller(int x, int y){
            if ( x < y )
                    return x;
            else
                    return y;
    }
    int main(){
            int x1;
            int y1;
            int x2;
            int y2;
            int x3;
            int y3;
            int x4;
            int y4;
            scanf("%d", &x1);
            scanf("%d", &y1);
            scanf("%d", &x2);
            scanf("%d", &y2);
            scanf("%d", &x3);
            scanf("%d", &y3);
            scanf("%d", &x4);
            scanf("%d", &y4);
            int dist1 = calcDistance(x1, y1, x2, y2);
            int dist2 = calcDistance(x3, y3, x4, y4);
            int smaller = getSmaller(dist1, dist2);
            printf("Smaller Distance is %d", smaller);
    }

Array <span><span><span style="font-variant:small-caps;">— Dealing with list of data —</span></span></span>
===========================================================================================================

Array is used to hold a collection of similar data. For example, a list
of numbers, a list of ID numbers, a list of city post codes, etc.

To use an array, we need to declare it with its legal maximum capacity.
Each item in an array can be said an element. Each element has an index.
The index starts with 0. Therefore, index of last possible element is 1
less than its maximum capacity.

Size of an array is number of elements it holds at a particular time in
execution. Sometimes we need an associated variable with an array that
holds its size.

Here we would like to present some operations with array.

Initialize an array –

    int arr[10];

**Operation on Single Element**
-------------------------------

Now we will learn basic operations on a single element of an array.

### Read

Read an element of the array

    x = arr[0];

### Write

Assign an element a value

    arr[0] = x;

### Input from Keyboard

Assign an element a value that an user provides.

    scanf("%d", &x[0]);

### Output to Monitor

Print an element to show user.

    printf("%d", x[0]);

**Operation on Whole Array**
----------------------------

### Input

Assign all the elements user input using the same recursion model used
before. It shows that the operation has the similar strategy and
philosophy as before –

input to array\[index to end\] = input to array\[index\] and input to
array\[index+1 to end\]

    #include<stdio.h>
    int inArray(int* array, int index, int end){
            if(index > end)
                    return 0;
            else
                    return scanf("%d", &array[index]) + 
                           inArray( array, index+1, end );    
    }
    int main(){
            int array[10];
            inArray( array, 0, 9 );
    }

### Output

Here we will show values of all the elements of an array.

input to array\[index to end\] = input to array\[index\] and input to
array\[index+1 to end\]

    #include<stdio.h>
    int inArray(int* array, int index, int end){
            if(index > end)
                    return 0;
            else
                    return scanf("%d", &array[index]) + 
                    inArray( array, index+1, end );    
    }
    int outArray(int* array, int index, int end){
            if(index > end)
                    return 0;
            else
                    return printf("%d ", array[index]) + 
                           outArray( array, index+1, end );    
    }
    int main(){
            int array[10];
            inArray( array, 0, 9 );
            outArray( array, 0, 9 );
    }

### Constant Assignment

Assign all the elements of the array to a constant value –

    #include<stdio.h>
    void cArray(int* array, int index, int end, int c){
            if(index <= end){
                    array[index] = c;    
                    cArray( array, index+1, end, c );
            }
    }
    void outArray(int* array, int index, int end){
            if(index <= end){
                    printf("%d ", array[index]); 
                    outArray( array, index+1, end );    
            }
    }
    int main(){
            int array[10];
            cArray( array, 0, 9, 1 );
            outArray( array, 0, 9 );
    }

### Progressive Assignment

Assign all the elements of an array progressively.

    #include<stdio.h>
    int square(int x){
            return x * x;
    }
    void cArray(int* array, int index, int end, int c){
            if(index <= end){
                    array[index] = square(index);    
                    cArray( array, index+1, end, c );
            }
    }
    void outArray(int* array, int index, int end){
            if(index <= end){
                    printf("%d ", array[index]); 
                    outArray( array, index+1, end );    
            }
    }
    int main(){
            int array[10];
            cArray( array, 0, 9, 1 );
            outArray( array, 0, 9 );
    }

### Sum of Array

Here we will sum values of all the elements of an array.

sum of array\[index to end\] = array\[index\] + sum of array\[index+1 to
end\]

Sometimes, we can use mathematical notations as –
$${\hbox{\em sum}}(a[i],\dots,a[n]) = \begin{dcases*}
        0 & when $i > n$\\
        a[i] + {\hbox{\em sum}}(a[i+1], \dots, a[n]) & otherwise
        \end{dcases*}$$

    #include<stdio.h>
    int iInt(int* array, int index){
        return scanf("%d", &array[index]);
    }
    int inArray(int* array, int index, int end){
        if(index > end)
            return 0;
        else
            return iInt( array, index ) + 
                   inArray( array, index+1, end );    
    }
    int sumArray(int* array, int index, int end){
        if(index > end)
            return 0;
        else
            return array[index] + 
                   sumArray( array, index+1, end );
    }
    int main(){
        int array[10];
        inArray( array, 0, 9 );
        printf("Sum is %d\n", sumArray(array, 0, 9));
        return 0;
    }

### Maximum of Array

Here we will find the maximum value of all the elements of an array.

‘max of array\[index to end\]’ = larger among ‘array\[index\]’ and ‘max
of array\[index+1 to end\]’

Sometimes, we can use mathematical notations as –
$${\hbox{\em larger}}(x, y) = \begin{dcases*}
        x & when $x > y$\\
        y & otherwise
        \end{dcases*}$$
$${\hbox{\em max}}(a[i],\dots,a[n]) = \begin{dcases*}
        a[i] & when $i = n$\\
        larger( a[i], {\hbox{\em max}}(a[i+1], \dots, a[n]) ) & otherwise
        \end{dcases*}$$

    #include<stdio.h>
    int iInt(int* array, int index){
        return scanf("%d", &array[index]);
    }
    int inArray(int* array, int index, int end){
        if(index > end)
            return 0;
        else
            return iInt( array, index ) + 
                   inArray( array, index+1, end );    
    }
    int larger(int x, int y){
        if( x > y )
            return x;
        else
            return y;
    }
    int max(int* a, int i, int e){
        if(i == e)
            return a[i];
        else
            return larger( a[i], max( a, i+1, e );
    }
    int main(){
        int array[10];
        inArray( array, 0, 9 );
        printf("Max is %d\n", max(array, 0, 9));
        return 0;
    }

Loop <span><span><span style="font-variant:small-caps;">— Another theory of repetition —</span></span></span>
=============================================================================================================

**While Loop**
--------------

A simple recursion where the recursion occurs at the right most place,
can be replaced by <span>*while*</span> construct.

### Model of While

Here we would like to present the model of <span>*While*</span> loop.

    initialization
    while( condition ){
            statements
            progression
    }  

\[whileloop\] Before description, we would like to give an example.

    i = 0;
    while( i < 10 ){
            printf("%d\n", i);
            i = i + 1;
    }  

Now we would like to explain it.

initialization

: is one or more assignment statements. It should assign a variable to
    a value so that the loop body get a chance to be executed. For
    example, `i = 0;` ensures that `i < 10` is true.

condition

: is a boolean expression. The value of a boolean expression is either
    <span>*True*</span> or <span>*False*</span>. It controls the
    repetition of the loop. If it is true, repetition continues and if
    it is false, repetition stops. For example, the loop will be
    continued exactly while `i < 10` is true.

statements

: is set of statements that we want to repeat.

progression

: is a statement which changes value of at least one variable used in
    the condition. There is no obvious pattern of it, however, most
    times it is increment, decrement or user given input. It solely
    depends on the condition. So, be carefull to construct
    the progression. For example, `i = i + 1;` keeps changing the value
    of `i` so that eventually it becomes `10` and the loop stops.

(start) <span>Start</span>; (init) \[below=of start,text width=3cm\]
<span>Initialization</span>; (decision) \[below=of init,text width=3cm\]
<span>Condition</span>; (statement) \[below=of decision,text width=3cm\]
<span>Statements</span>; (progression) \[left=of statement,text
width=3cm\] <span>Progression</span>; (end) \[right=of statement\]
<span>End</span>; (start) – (init); (init) – (decision); (decision) –
node\[xshift=5mm\] <span>True</span> (statement); (statement) –
(progression); (progression) |- (decision); (decision) -|
node\[yshift=3mm,xshift=-1cm\] <span>False</span> (end);

Loop is very important in C, because it is possible to iterate very
large number of times using different loop patterns. Recursion occupies
some memory (in special place called stack) for each of its steps. So,
for a large number of steps, the amount of occupatied memory is
multiplied and it can overload the limit of the available memory.

Unlike C, many other languages (e.g. Haskell, Erlang, etc.) those are
designed especially for recursion, have special technique to overcome
this problem.

If we want to compare it to a recursive function in construction, it is
roughly –

    function( parameters )
            if( condition == 0 )
                    return c;
            else
                    statements + function( progression );
    }

    ...
            function( initialization ); 
    ... 

### Print 1 to n

Here we will write a program equivalent to the example at Page .

    #include<stdio.h>
    int main(){
        int i = 1;
        int n = 10;
        while( i <= n ){
            printf("%d\n", i);
            i ++;
        }
    }

### Print Even Numbers from 2 to n

Here we will write a program equivalent to the example at Page .

   **Shorthand** **Full form**
  --------------- ---------------
     `a += b` `a = a + b`
     `a -= b` `a = a - b`
     `a *= b` `a = a * b`
     `a /= b` `a = a / b`
     `a %= b` `a = a % b`

    #include<stdio.h>
    int main(){
        int i;
        i = 2;
        int n;
        scanf("%d", &n);
        while( i <= n ){
            printf("%d\n", i);
            i += 2;
        }
    }

### Bill of a Customer with While

Let us see how can we rewrite the example of <span>*Bill of a
Customer*</span> at Page without recursion.

    #include<stdio.h>
    int main(){
        int item;
        int quantity;
        float price;
        float total = 0.0;
        int option;
        printf("Enter 1 to add, 0 to finish\n");
        scanf("%d", &option);
        while(option != 0){
            printf("Enter Item Code\n");
            scanf("%d", &item);
            printf("Enter Quantity\n");
            scanf("%d", &quantity);
            price = calcTotalPrice(item, quantity);
            total = total + price;
            printf("Enter 1 to add, 0 to finish\n");
            scanf("%d", &option);
        }
        printf("Total bill is BDT %.2f\n", price);
    } 

If we compare the above code with the one in Page , we will find that
both are equivalent when executing. However, the code is significantly
different. For some people, using `while` is easier because the code is
straightforward. But, in practice, the earlier will give us some benefit
-

- The mathematical modeling of `while` is very complex and often lead
    us to write incorrect code. But recursion (in example of Page )
    gives us easier mathematical modeling as we have seen throughout
    the text.

- The code of the above example is clumsy. It will be more clumsy when
    our feature will be more. In the example in Page , the code
    is cleaner.

**For Loop**
------------

<span>*For*</span> loop is mostly used when we know the number of steps
a code segment should be iterated. It is just an alternative pattern of
while loop. It has pattern so that programmer do not forget to put any
of initializatio, condition, progression and statements.

It is as follows –

    for(initialization; condition; progression){
            statements;
    }

The example give at Page can be rewritten using <span>*for*</span> loop.

    #include<stdio.h>
    int main(){
        int i;
        for(i = 0; i < 10; i = i + 1){ 
            printf("%d\n", i);
        }
    }  

Except the structure, there is no difference between
<span>*While*</span> and <span>*For*</span> loop structures.

### With Array

However, <span>*For*</span> loop is very usefull to process arrays.

Here is an example of finding minimum value of an array,

    #include<stdio.h>
    int main(){
        int i;
        int min;
        int array[10];
        for(i = 0; i < 10; i ++){ 
            scanf("%d", &array[i]);
        }
        min = array[0];
        for(i = 1; i < 10; i ++){ 
            if( array[i] < min )
                min = array[i];
        }
        printf("Min is %d\n", min);
        return 0;
    }  

**Do …While Loop**
------------------

<span>*Do …While*</span> loop is another variant of <span>*While*</span>
loop. It is used when we know that the body of the loop will be executed
at least once without any condition. After the first step, repetition
depends on the condition.

Its structure is as follows –

    initialization;
    do{
            statements;
            progression;
    }while( condition );

The example give at Page can be rewritten using <span>*for*</span> loop.

    i = 0;
    do{
            printf("%d\n", i);
            i ++;
    }while( i < 10 );

(start) <span>Start</span>; (init) \[below=of start,text width=3cm\]
<span>Initialization</span>; (statement) \[below=of init,text
width=3cm\] <span>Statements</span>; (progression) \[below=of
statement,text width=3cm\] <span>Progression</span>; (decision)
\[right=of progression,text width=3cm\] <span>Condition</span>; (end)
\[below=of decision\] <span>End</span>; (start) – (init); (init) –
(statement); (statement) – (progression); (progression) – (decision);
(decision) |- node\[xshift=5mm\] <span>True</span> (statement);
(decision) – node\[xshift=6mm\] <span>False</span> (end);

Function as Parameter <span><span><span style="font-variant:small-caps;">— A New Horizon —</span></span></span>
===============================================================================================================

So far we have learned how a parameter is used in a function. We have
also seen many examples of how to call a function and pass our value to
the parameters. These parameters typically are data parameter because
they can store data only.

C also supports parameter for function. In that case, the parameter is
treated as a function instead of a data. We also need to pass a function
as the argument for the parameter.

Signature of a function is the ordered list of types of the parameters.
So, the signature of the function <span>void f(int x)</span> is
<span>*int*</span>, <span>void f(int x, int y)</span> is
<span>*int,int*</span>, <span>void f(int a, double b)</span> is
<span>*int, double*</span>, etc. Signature is used to understand the
type of a function.

Parameters must be declared with the signatures of the functions in this
case.

We will see some example here.

**Simple Example**
------------------

    #include<stdio.h>
    void f(){
        printf("This is f()\n");
    }
    void g(){
        printf("This is g()\n");
    }
    void p(void (*x)()){
        printf("Parameter x() is being called.\n");
        x();
    }
    int main(){
        p(f);
        p(g);
    }

Here <span>x</span> is a function parameter. It is function with empty
signature.\

**Unary Function Parameter**
----------------------------

    #include<stdio.h>
    void dec(int a){
        printf("In Decimal: %d\n", a);
    }
    void hex(int b){
        printf("In Hexadecimal: %x\n", b);
    }
    void oct(int c){
        printf("In Hexadecimal: %o\n", c);
    }
    void conv(void (*d)(int)){
        d(15);
    }
    int main(){
        conv(dec);
        conv(hex);
        conv(oct);
    }

Here the function parameter <span>d</span> has the signature
<span>*int*</span>. So, the name of the functions those have exactly one
int parameter can be the argument for this function.\

**Another Unary Fucntion Parameter with a Return Type**
-------------------------------------------------------

As easily guessed, the return type can be changed to something other
than void. We will have to pass name of similar functions as the
argument too.

    #include<stdio.h>
    double inc(double a){
        return a + 1;
    }
    double dec(double b){
        return b - 1;
    }
    double inv(double c){
        return 1 / c;
    }
    void f(double (*d)(double)){
        double x = d(10.0);
        printf("Result is %.2lf\n", x);
    }
    int main(){
        f(inc);
        f(dec);
        f(inv);
    }

Here <span>a</span>, <span>`b`</span> and <span>c</span> are data
parameter but <span>d</span> is the function parameter.\

**Function in Memory**
----------------------

It can be noted that there is a reason behind the syntax of ’’ with the
parameter name ’<span>x</span>’ (<span>(\*x)</span>). In memory,
functions are list of bytes that corresponds to a list of commands. So,
functions are really nothing but a different kinds of data.

Have a look in the following program.

    #include<stdio.h>
    void a(){
        printf("a() is called\n");
    }
    unsigned int f(void (*x)()){
        printf("%x\n", (unsigned int)x);
        return (unsigned int)x;
    }
    void g(void (*x)()){
        x();
    }
    int main(){
        unsigned int address = f(a);
        g((void (*)())address);
    }

The function <span>f</span> outputs and returns the integer value
(unsigned to avoid negative value, since addresses are never negative)
of the address where the bytes of the function is stored. We can use raw
address to convert it into the function as shown at the following line.\

### Note for Critics

In a 32 bit machine, we expect consecutive 4 bytes as a single
instruction. So, while a program is running, in C, it is possible to
change the behavior of a particular function (i.e. the program itself)
if we correctly know the binary representation of the instructions.
Because, it is possible to convert between pointers of different types
and it is not difficult to overwrite a data in a memory using syntax
like <span>x = 0;</span>. Most of the time, we do not need such a
program. On the other hand, an incorrect instruction can be very
dangerous. For that reason, it has not been encouraged in this book to
learn pointer in general. Instead, we will learn how to use pointer
correctly in certain situation.

We will use function as parameters in the next chapters exclusively.

Nested Iteration <span><span><span style="font-variant:small-caps;">— Repetition of repetition —</span></span></span>
=====================================================================================================================

**Composition of Recursive Functions**
--------------------------------------

### Prime Numbers in a Range

We often need to repeat a job which in turn requires to repeat another
job. For example, if we need to print all prime numbers from $2$ to $n$,
we may need to test each number if it is a prime number. Again, to check
if a number is a prime number, we need to check if it is divisible by
any number from $2$ to $n-1$. Basically we can do it simply by composing
two recursive functions.

    #include<stdio.h>
    int checkPrime(int x, int n){
      if(x == n)
        return 1;
      else if(x % n == 0)
        return 0;
      else
        checkPrime(x, n+1);
    }
    void printPrimes_I_to_N(int i, int n){
      if(i < n){
        if(checkPrime(i, 2) == 1)
          printf("%d\n", i);
        
        printPrimes_I_to_N(i+1, n);
      }
    }
    int main(){
      printPrimes_I_to_N(2,100);
    }

##### Efficiency

It is important to write program simple. There are two ways to achieve
it. One is, writing smaller size of code. It helps to understand the
code easily. It also helps to check for errors or bugs. Writing smaller
code may sometimes mean code with fewer lines, functions with fewer
parameters, etc. But it is not of course too much short code that
sacrifices simplicity. Anyway, it does not mean to make our variable
names very small that is difficult to understand.

Another way to write simpler code can be accomplish by writing a code
that can solve a problem with less time and memory. This is a bit harder
and we will focus it later.

Although the above functions are straightforward, With some small
tricks, there are several ways to write such a program. For example, we
can reduce one parameter from the function
<span>*printPrimes\_i\_to\_n*</span>. Here it is –

    #include<stdio.h>
    int checkPrime(int x, int n){
      if(x == n)
        return 1;
      else if(x % n == 0)
        return 0;
      else
        checkPrime(x, n+1);
    }
    void printPrimes_i_to_n(int n){
      if(n > 1){
        printPrimes_i_to_n(n - 1);
        if(checkPrime(n, 2) == 1)
          printf("%d\n", n);
      }
    }
    int main(){
      printPrimes_i_to_n(100);
    }

Think by yourself how the above two programs are equivalent. If you can
understand it, in future, you will be able to write lots of functions in
simpler way.

### (n$\times$m) Matrix

(n$\times$m) $0$-matrix is a matrix that has n number of rows and m
number of columns and at each cell, the values are $0$.

    #include<stdio.h>
    void printRow(int col){
        if(col > 0){
            printf("%d\t",0);
            printRow(col - 1);
        }
    }
    void printMatrix(int row, int col){
        if(row > 0){
            printRow(col);
            printf("\n");
            printMatrix(row - 1, col);
        }
    }
    int main(){
        printMatrix(4, 5);
    }

Now you can tweak the above program to get very interesting output. Such
as –

``` {startFrom="4"}
        printf("%d\t", 1);
```

or,

``` {startFrom="4"}
        printf("%d\t", col);
```

Although the above matrixes (or matrices) are casual, we can tweak it
like before to get real matrixes. Here it is with evidence (just run the
program).

    #include<stdio.h>
    void printRow(int row, int col){
        if(col > 0){
            printRow(row, col - 1);
            printf("(%d,%d)\t", row,col);
        }
    }
    void printMatrix(int row, int col){
        if(row > 0){
            printMatrix(row - 1, col);
            printRow(row, col);
            printf("\n");
        }
    }
    int main(){
        printMatrix(4, 5);
    } 

Now change just one line or a little to get many interesting results
such as –

``` {startFrom="4"}
        printf("%d\t", row*col);
```

or,

    #include<stdio.h>
    void printRow(int row, int col, int cols){
        if(col > 0){
            printRow(row, col - 1, cols);
            printf("%d\t", (row-1)*cols + (col-1));
        }
    }
    void printMatrix(int row, int col){
        if(row > 0){
            printMatrix(row - 1, col);
            printRow(row, col, col);
            printf("\n");
        }
    }
    int main(){
        printMatrix(10, 10);
    }

**Two Dimensional Array**
-------------------------

Now we will analyze a more complex case. Suppose, ULAB has $5$
departments. Each department has $12$ running batches (we will exclude
students who passed 12 semesters for simplicity). We have different
kinds of records of these $12 \times 5$ batches such as total number of
students, highest CGPA own by a student, average CGPA, etc. Now we may
need to know –

- total number of students of each department

- total number of students of each batch in all departments

- total number of students of the university

- highest CGPA own by a student of each department

- highest CGPA own by a student of each batch

- highest CGPA in the university

- average CGPA of each department

- average CGPA of each batch

- average CGPA of the university

- etc.

Our first task to solve such problems is to store data, process it and
then output it.

Now we will first learn that a two dimensional array can be the first
choice to hold such data. Suppose, our table of total students is as
follows –

  ------------------ --------- --------- --------- --------- ---------
                      BBA (0) MSJ (1) ENG (2) CSE (3) ETE (4)
     Spring 2011 (0) 119 80 20 30 25
     Summer 2011 (1) 119 80 20 30 25
       Fall 2011 (2) 119 80 20 30 25
     Spring 2012 (3) 119 80 20 30 25
     Summer 2012 (4) 119 80 20 30 25
       Fall 2012 (5) 119 80 20 30 25
     Spring 2013 (6) 119 80 20 30 25
     Summer 2013 (7) 119 80 20 30 25
       Fall 2013 (8) 119 80 20 30 25
     Spring 2014 (9) 119 80 20 30 25
    Summer 2014 (10) 119 80 20 30 25
      Fall 2014 (11) 119 80 20 30 25
  ------------------ --------- --------- --------- --------- ---------

It is possible to manually input all the data. In this example, we will
create a 2D array of size $12 \times 5$, store the data by input from
keyboard and print the data in a tabular view.

We have clearly two choices to take input in a 2D array. One is in the
same pattern as described before.

    #include<stdio.h>
    #define row 12
    #define col 5
    #include<stdio.h>
    #define row 12
    #define col 5
    void colIn(int r, int c, int students[row][col]){
        if(c < col){
            scanf("%d",&students[r][c]);
            colIn(r, c+1, students);
        }
    }
    void rowIn(int r, int c, int students[row][col]){
        if(r < row){
            colIn(r, c, students);
            rowIn(r+1, c, students);
        }
    }

    int main(){
        int students[row][col];
        rowIn(0, 0, students);
    }

We can take input in a 2D array using nested for loop too. It is
basically more traditional style.

    #include<stdio.h>
    #define row 12
    #define col 5

    int main(){
        int students[row][col];
        int i, j;
        for(i=0; i<row; i++)
            for(j=0; j<col; j++)
                scanf("%d", &students[i][j]);
    }

<span>\#define row 12</span> creates a macro named <span>row</span>. It
will replace all the occurances of <span>row</span> by <span>12</span>
in the rest of the code. We use the macro in C so that we can use the
code as a template. Because, it allows to change the size of the 2D
array easily without much modification.

Now it is time to print the matrix.

    #include<stdio.h>
    #define row 12
    #define col 5
    void colIn(int r, int c, int students[row][col]){
        if(c < col){
            scanf("%d",&students[r][c]);
            colIn(r, c+1, students);
        }
    }
    void rowIn(int r, int c, int students[row][col]){
        if(r < row){
            colIn(r, c, students);
            rowIn(r+1, c, students);
        }
    }
    void colOut(int r, int c, int students[row][col]){
        if(c < col){
            printf("%d ",students[r][c]);
            colOut(r, c+1, students);
        }
    }
    void rowOut(int r, int c, int students[row][col]){
        if(r < row){
            colOut(r, c, students);
            printf("\n");
            rowOut(r+1, c, students);
        }
    }
    int main(){
        int students[row][col];
        rowIn(0, 0, students);
        rowOut(0, 0, students);
    }

### Function as Parameter

The above code may look clear enough for you. If it is, you can just
follow it. However, notice that the function <span>colIn</span> and
<span>colOut</span> is almost same except the $3^{rd}$ line. So, they
are of same pattern. Again, <span>rowIn</span> and <span>rowOut</span>
also have same pattern.

The above code becomes little clumsy because of the repetition of
similar functions. Can we make it short so that a single function for
repetition can be composed with different smaller function to perform
different task? Now we will try the genuine functional approach.

<span> The following code may look extremely difficult for many people.
This is because, C does not have a better way to pass a function in
another function. We will certainly explain it. The great advantage is
that it will reduce a lots of code in a large program and make it
simple. </span>

Let us try it –

    #include<stdio.h>
    #define row 12
    #define col 5
    void input(int r, int c, int students[row][col]){
        scanf("%d", &students[r][c]);
    }
    void nothing(int r, int c, int students[row][col]){
    }
    void output(int r, int c, int students[row][col]){
        printf("%d ", students[r][c]);
    }
    void linebreak(int r, int c, int students[row][col]){
        printf("\n");
    }
    /*
        * The following two functions will be necessary only once
        * in your entire program. And you do not need to change it.
    */
    void colsRep( int r, int c, int matrix[row][col],
                  void (*f)(int, int, int[row][col])){

        if(c < col){
            colsRep(r, c+1, matrix, f);
            (*f)(r, c, matrix);
        }
    }
    void rowsRep( int r, int c, int matrix[row][col],
                  void (*f)(int, int, int[row][col]),
                  void (*g)(int, int, int[row][col])){

        if(r < row){
            colsRep(r, c, matrix, f);
            (*g)(r, c, matrix);
            rowsRep(r+1, c, matrix, f, g);
        }
    }

    int main(){
        int students[row][col];
        rowsRep(0, 0, students, input, nothing);
        rowsRep(0, 0, students, output, linebreak);
    }

Again, if you feel the above code is difficult for you, an easy recipe
is –

    #include<stdio.h>
    #define row 12
    #define col 5

    int main(){
        int students[row][col];
        int i, j;
        for(i=0; i<row; i++)
            for(j=0; j<col; j++)
                scanf("%d", &students[i][j]);

        for(i=0; i<row; i++){
            for(j=0; j<col; j++)
                printf("%d ", students[i][j]);
            printf("\n");
        }
    }

### File Input

It is a good time to introduce how to use file in C program. Files are
necessary for several reasons such as parmanently storing data, read
from a stored data, avoid large amount of input from keyboard, etc. This
time we want to avoid input from keyboard because it wastes energy and
time both.

There are several ways to use a file in C. We will learn them according
to our needs.

Here we will create a file named “students.txt”. We will put the file in
the same directory (or folder) where the source code of the program is
stored. Now we need to manually store data, the matrix as follows –

<span>||lllll||</span> 119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\
119 & 80 & 20 & 30 & 25\

Now add a single lines to the function <span>main</span> and keep the
rest of the code unchanged.

``` {startFrom="42"}
int main(){
    int students[row][col];
    freopen("students.txt","r",stdin);
    rowsRep(0, 0, students, input, nothing);
    rowsRep(0, 0, students, output, linebreak);
}
```

Generally, the standard input is <span>stdin</span> that is a file
stream that represents the keyboard. By the line
<span>freopen(“students.txt”,“r”,stdin);</span>, standard input is
changed from <span>stdin</span> to the file named “students.txt”.

### Operation on a Matrix

Now we will present another example where our program will show total
tution fee earned from each batch. For simplicity, assume that each
student pays BDT 200,000 as his/her tution fee. Therefore, after taking
input, we will multiply the number of students at each batch with the
tution fee.

    #include<stdio.h>
    #define row 12
    #define col 5
    void output(int r, int c, int students[row][col]){
        printf("%d ", students[r][c]);
    }
    void input(int r, int c, int students[row][col]){
        scanf("%d", &students[r][c]);
    }
    void nothing(int r, int c, int students[row][col]){
    }
    void linebreak(int r, int c, int students[row][col]){
        printf("\n");
    }
    /* New Function */
    void totalTution(int r, int c, int students[row][col]){
        students[r][c] = students[r][c] * 2000000;
    }
    void colsRep( int r, int c, int students[row][col],
                    void (*f)(int, int, int[row][col])){
        if(c < col){
            colsRep(r, c+1, students, f);
            (*f)(r, c, students);
        }
    }
    void rowsRep( int r, int c, int students[row][col],
                    void (*f)(int, int, int[row][col]),
                    void (*g)(int, int, int[row][col])){
        if(r < row){
            colsRep(r, c, students, f);
            (*g)(r, c, students);
            rowsRep(r+1, c, students, f, g);
        }
    }
    int main(){
        int students[row][col];
        freopen("students.txt","r",stdin);
        rowsRep(0, 0, students, input, nothing);
        /* New Line */
        rowsRep(0, 0, students, totalTution, nothing);
        rowsRep(0, 0, students, output, linebreak);
    }

Common Programs <span><span><span style="font-variant:small-caps;">— Some Algorithms —</span></span></span>
===========================================================================================================

**Linear Search**
-----------------

Linear Search is important to find a data into an unsorted array.\

### Single Typed Linear Search

First, we will search an integer data into an integer array.

    #include<stdio.h>
    #define LEN 100000
    /* Used to fill and print systematic unsorted dummy data 
       for convenience */
    void fillCell(int *data, int index){
        data[index] = index * (index % 10);
    }
    void printCell(int *data, int index){
        printf("%d, ", data[index]);
    }
    void repeat(int *data, 
                int i,
                void (*f)(int*, int)){

        if(i < LEN){
            (*f)(data, i);
            repeat(data, i+1, f);
        }
    }
    /* The linear search algorithm implementation */
    int fsearch(int *d, int i, int key){
        if(i >= LEN)
            return -1;
        if(d[i] == key) //key is at i-th index
            return i;
        else //search in the rest of data
            fsearch(d, i+1, key);
    }
    /* Testing */
    int main(){
        int data[LEN];
        int r, key;
        int i = 0;
        repeat(data, i, fillCell);
        repeat(data, i, printCell);
        printf("\nEnter search item: \n");
        scanf("%d", &key);
        r = fsearch(data, i, key);
        if(r >= 0)
            printf("%d is found at %d index\n", key, r);
        else
            printf("%d is not found\n", key);
    }

In the above code, we have five functions including main. The function
<span>repeat</span> repeats execution of a given function <span>f</span>
total of <span>LEN</span> times on the array <span>data</span>. The
purpose of <span>fillCell</span> is to assign a dummy data to the given
<span>index</span>-th cell of the array <span>data</span>. The purpose
of <span>printCell</span> is to print the data at the
<span>index</span>-th cell of the array <span>data</span>.

From the function <span>main</span>, the function <span>repeat</span> is
called twice. First, it is called with supplying an empty array and the
function name <span>fillCell</span>. At each time <span>repeat</span> is
called, the function <span>fillCell</span> is called within it via the
function parameter <span>f</span>. The array <span>data</span> and the
index <span>i</span> is supplied to the calling function. Next, it is
called with suppying the filled array and the function
<span>printCell</span> so that all the data can be printed on terminal.

The function <span>fsearch</span> searches the <span>key</span> in the
array <span>data</span>. Search begins with the index <span>i</span>. It
returns the index where <span>key</span> is found and -1 if it is not
found.

We can write same code in our previous fashion. But in C, it is not
always true that similar pattern code is always easy to understand.
Consider the following program –

    #include<stdio.h>
    #define LEN 100
    int fillCell(int *data, int index, int b){
        data[index] = index * (index % 10);
        return -1;
    }
    int printCell(int *data, int index, int b){
        printf("%d, ", data[index]);
        return -1;
    }
    int findData(int *data, int index, int b){
        if(data[index] == b)
            return index;
        else
            return -1;
    }
    int repeat(int *data, int i,
                  int (*f)(int*, int, int), int b){
        int r;

        if(i < LEN){
            r = (*f)(data, i, b);
            if(r == -1)
                return repeat(data, i+1, f, b);
            else
                return r;
        }else{
            return -1;
        }
    }

    int main(){
        int data[LEN];
        int r, key;
        repeat(data, 0, fillCell, 0);
        repeat(data, 0, printCell, 0);
        printf("\nEnter search item: \n");
        scanf("%d", &key);
        r = repeat(data, 0, findData, key);
        if(r >= 0)
            printf("%d is found at %d index\n", key, r);
        else
            printf("%d is not found\n", key);
    }

These functions are of different in nature. So, it is not very wise to
use them this way. Naturally, <span>fillCell</span> and
<span>printCell</span> are similar in the sense that they just do a job
with the array and the index and it does not make sense to return any
value from these functions. But, the function <span>findData</span>
matches a cell data to a given data and returns a data indicating if it
does not find it (<span>-1</span>) or the index where it is found.

Anyway, if the length of the array is very big, and the program crashes
while running, it is better to use <span>for</span> loop. Anyway,
similar program using <span>for</span> loop is not be very difficult.

    #include<stdio.h>
    #define LEN 1000000
    int fillCell(int *data, int index, int b){
        data[index] = index * (index % 10) + (index % 3);
        return -1;
    }
    int printCell(int *data, int index, int b){
        printf("%d, ", data[index]);
        return -1;
    }
    int findData(int *data, int index, int b){
        if(data[index] == b)
            return index;
        else
            return -1;
    }
    int repeat(int *data, int b,
                  int (*f)(int*, int, int)){
        int i, r;
        for(i = 0; i < LEN; i ++){
            r = f(data, i, b);
            if(r >= 0)
                return r;
        }
    }
    int main(){
        int data[LEN];
        int r, key;
        repeat(data, 0, fillCell);
        repeat(data, 0, printCell);
        printf("\nEnter search item: \n");
        scanf("%d", &key);
        r = repeat(data, key, findData);
        if(r >= 0)
            printf("%d is found at %d index\n", key, r);
        else
            printf("%d is not found\n", key);
    }

Notice that the array size in this program is 10 times than the array
size in the previous programs.

### Universal Typed Linear Search

In last section, we worked with only integer type of data. If we want to
make a function that should work with all kinds of data types we need
some special tricks. The main concept is to split up all the data into
an array of bytes (<span>char</span> in C). We will keep information of
the size of the array needed for our target data type. And we will
process over all the bytes in the array collectively. Now we will see
how can we make a linear search function that works with every kinds of
data types.

    #include<stdio.h>
    int equals(void *x, void *y, int size){
        if(size <= 0)
            return 1;
        char a = ((char *)x)[size-1];
        char b = ((char *)y)[size-1];
        if(a != b)
            return 0;
        return equals(x, y, size-1);
    }
    int fsearch(void *d, int size, void *key, int keysize){
        if(size <= 0)
            return -1;
        /*
            Since the array is flattened, 
            we need to calculate
            exact number of bytes to skip 
            to begin search.
        */
        void *t = d + (keysize * (size - 1));
        if(equals(t, key, keysize) == 1)
            return size-1;
        else
            return fsearch(d, size-1, key, keysize);
    }
    int main(){
        int x[10] = {2,5,7,8,9,200,4,5,6,3};
        int y = 9;
        printf("%d is found at %d index\n", y, 
               fsearch(x, 10, &y, sizeof(y)));

        double a[5] = {5.78, 1.999E12, 3E-40, 
                       10000000.00009, 10};
        double b = 10000000.00009;
        printf("%lf is found at %d index\n", b, 
                fsearch(a, 5, &b, sizeof(b)));

        unsigned long long p[10] = {
          20000000000ULL,50000000000000000ULL,
          7000000001ULL,8000000000000ULL,
          9000000000000ULL,30000000000ULL,
          200,4,5,6};
        unsigned long long q = 9000000000000ULL;
        printf("%lld is found at %d index\n", q,
               fsearch(p, 10, &q, sizeof(q)));
    }

**Binary Search**
-----------------

Just tweak any of the above code to achieve a binary search.

    #include<stdio.h>
    #define LEN 50
    /* Used to fill and print sorted dummy data 
       for convenience */
    void fillCell(long long *data, int index){
        if( index == 0 || index == 1 || index == 2)
            data[index] = index;
        else
            data[index] = data[index - 1] +
                          data[index - 2] +
                          data[index - 3];
    }
    void printCell(long long *data, int index){
        printf("%lld, ", data[index]);
    }
    void repeat(long long *data, int i,
                  void (*f)(long long*, int)){
        if(i < LEN){
            (*f)(data, i);
            repeat(data, i+1, f);
        }
    }
    /* The binary search implementation */
    int bSearch(long long *d, int i, int l, long long key){
        int m;
        //i >= l only when nothing more to search
        if(i >= l)
            return -1;
        //Get the middle index
        m = (i + l) /2;
        if(d[m] == key) //the middle is the key
            return i;
        else if(d[m] < key) //search left part only
            return bSearch(d, i, m-1, key);
        else //search right part only
            return bSearch(d, m+1, l, key);
    }
    /* Testing */
    int main(){
        long long data[LEN];
        int r;
        long long key;
        repeat(data, 0, fillCell);
        repeat(data, 0, printCell);
        printf("\nEnter search item: \n");
        scanf("%lld", &key);
        r = bSearch(data, 0, LEN, key);
        if(r >= 0)
            printf("%lld is found at %d index\n", key, r);
        else
            printf("%lld is not found\n", key);
    }

**Selection Sort**
------------------

Here we present an implementation of selection sort using recursive
functions. In this program we use the function <span>fRepeat</span> that
is more versatile than previous repeat function in the sense that it is
compatible with more function.

This code contains different use of the function <span>fRepeat</span>,
that also includes the sort function.

In this program, we concentrate only what should be happened at each
cell, at last cell and what should be the relation between the each cell
and the result from rest of the cells.

    #include<stdio.h>
    #define LEN 51
    int fillCell(int *data, int index){
        data[index] = index * (index % 5) + (index % 3);
        return 0;
    }
    int printCell(int *data, int index){
        printf("%d, ", data[index]);
        return 0;
    }
    int nothing(int a, int b){
        return 0;
    }
    int smaller(int a, int b){
        if(a < b)
            return a;
        else
            return b;
    }
    int id(int *data, int i){
        return data[i];
    }
    int fRepeat( int *data, int i,
                    int (*each)(int*, int),
                    int (*agg)(int, int),
                    int (*last)(int*, int)){
        if(i >= LEN-1)
            return last(data, i);
        else
            return agg( fRepeat(data, i+1,
                                each, agg, last),
                        each(data, i));
    }
    void swap(int *data, int i, int j){
        int t = data[i];
        data[i] = data[j];
        data[j] = t;
    }
    int lSearch(int *d, int i, int key){
        if(i >= LEN)
            return -1;
        if(d[i] == key)
            return i;
        else
            lSearch(d, i+1, key);
    }
    int selSort(int *data, int i){
        int minimum;
        int minIndex;

        minimum = fRepeat(data, i, id, smaller, id);
        minIndex = lSearch(data, i, minimum);
        swap(data, i, minIndex);
        return 0;
    }
    int reverse(int *data, int i){
        if(i < (LEN - 1)/2)
            swap(data, i, LEN-i-1);
    }
    int main(){
        int data[LEN];
        int r, key;
        //fill array with unsorted data
        fRepeat(data, 0, fillCell, nothing, fillCell);
        //print the unsorted array
        fRepeat(data, 0, printCell, nothing, printCell);
        //sort the array
        fRepeat(data, 0, selSort, nothing, id);
        //print the sorted array
        printf("\n\n");
        fRepeat(data, 0, printCell, nothing, printCell);
        //reverse the array
        fRepeat(data, 0, reverse, nothing, reverse);
        //print the reversed array
        printf("\n\n");
        fRepeat(data, 0, printCell, nothing, printCell);
    }

**Quick Sort**
--------------

Here we present two version of quick sort implementations.

    #include<stdio.h>
    #define N 50
    void input(int *d, int i, int l){
     if(i < l){
      d[i] = (i % 10 ) * (i % 5) + i;
      input(d, i+1, l);
     }
    }

    void output(int *d, int i, int l){
     if(i < l){
      printf("%d ", d[i]);
      output(d, i+1, l);
     }else
      printf("\n");
    }

    int searchLarger(int *d, int p, int i, int l){
     if(d[i] > d[p] || i == l)
      return i;
     else
      return searchLarger(d, p, i+1, l);
    }

    int searchSmaller(int *d, int p, int i, int l){
     if(d[l] < d[p] || i > l)
      return l;
     else
      return searchSmaller(d, p, i, l-1);
    }

    void swap(int *d, int a, int b){
     int t = d[a];
     d[a] = d[b];
     d[b] = t;
    }

    int placement(int *d, int p, int i, int l){
     i = searchLarger(d, p, i, l);
     l = searchSmaller(d, p, i, l);
     if(i < l){
      swap(d, i, l);
      return placement(d, p, i, l);
     }
     return l;
    }

    void quicksort(int *d, int i, int l){
     int p;
     if(i < l){
      p = placement(d, i, i+1, l);
      swap(d, i, p);
      quicksort(d, i, p-1);
      quicksort(d, p+1, l);
     }
    }

    int main(){
     int data[N];
     input(data, 0, N);
     output(data, 0, N);
     quicksort(data, 0, N-1);
     output(data, 0, N);
    }

The function <span>input</span> and <span>output</span> are not really
part of the quick sort. They are intended to be used for testing our
implementation. One can easily modify especially the <span>input</span>
function to feed the array with systematic sorted or unsorted data.
Although it does not produce real random data, however, it is very
simple and it is able to produce unsorted data set that is enough for
our simulation. A more complex equation may be used to produce a better
data set.

Now we will prove that the functions <span>forward</span>,
<span>backward</span>, <span>placement</span> and <span>quicksort</span>
indeed sort an array.

### Proof of Correctness

We will prove that all the functions involved with the quicksort program
are correct. This proof is more descriptive than mathematical to favor
the young readers.

<span>**Swap:**</span> First, we will prove that the function
<span>swap</span> is correct. It is intended to exchange the value of
<span>d\[a\]</span> and <span>d\[b\]</span>. We will prove that if
<span>d\[a\]</span> is $x$ and <span>d\[b\]</span> is $y$ before the
function is executed, after its execution <span>d\[a\]</span> will be
$y$ and <span>d\[b\]</span> will be $x$.

- Suppose, <span>d\[a\]</span> is $x$ and <span>d\[b\]</span> is $y$.

- <span>int t = d\[a\];</span> gives us <span>t</span> is $x$.

- <span>d\[a\] = d\[b\];</span> gives us <span>d\[a\]</span> is $y$.

- At last, <span>d\[b\] = t;</span> gives us <span>d\[b\]</span> is
    $x$.

<span>**searchLarger:**</span> Now we will show that the function
<span>searchLarger</span> is correct. Its intended purpose is to find
the index of an array where the value of that index is larger than the
value of a given index. This search must be performed within a given
boundary. In case, there is such larger value, it will give the boundary
index.

- Suppose, <span>d</span> is the array or starting memory address of
    the collection of data, <span>p</span> is the given index,
    <span>i</span> is the index to be examine if its value is larger and
    finally, <span>l</span> is the given boundary.

- We will search it one by one in the array until we reach
    <span>l</span>.

- If <span>d\[i\]</span> is larger than <span>d\[p\]</span>,
    <span>i</span> is our intended output of the function. If
    <span>i</span> is equal to <span>l</span>, still it is our output.

- Otherwise, search the larger value in the next cell.

<span>**searchSmaller:**</span> In the same way as previous function,
<span>searchSmaller</span> is correct too.

<span>**placement:**</span> Next, we will show that the function
<span>placement</span> is correct. This function has the most crucial
role in quicksort algorithm. It has a simple purpose. It takes an
element from the segment of the array that is called pivot. It then
places the pivot such that all the smaller and larger value (compared to
the pivot) must resides at the left and the right of the pivot
respectively. In a typical case, the first element can be considered as
the pivot. An example can be given to explain the fact. In the segment
$[4,2,7,6,1]$, if <span>d</span> is $[2,7,6,1]$ and pivot value is $4$,
<span>placement</span> will convert it to an array $[2,1,4,6,7]$. Here
$2,1$ are smaller than $4$ and hence $4$ is placed after those two
elements. Similarly, $6,7$ are placed after $4$.

Among the function parameters, <span>d</span>, <span>p</span>,
<span>i</span> and <span>l</span> represent the total array, the pivot,
the starting index of the segment and its end index respectively.
<span>searchLarger</span> gives the first value that is larger than the
pivot ($i$) and <span>searchSmaller</span> gives the last value that is
smaller than the pivot. If $i$ does not cross $l$, the elements of those
indices are swapped. It then performs the same operations for the inner
segment bound by the new values of $i$ and $l$. If $i$ and $l$ are same
or cross each other, we assume that the segment is perfectly rearranged
and we return the last smaller (than pivot) value, $l$ to which the
pivot can be swapped.

<span>**quicksort:** </span> The function <span>quicksort </span> sorts
the array $d$ between the range from $i$ to $l$ ($d_i,\ldots, d_l$). It
attempts to sort the range only when there are more than one data in the
range. $i<l$ ensures it.

We know that the function <span>placement</span> gives us a place $p$
where $i<p<l$ and rearranged $d_{i+1},\ldots,d_l$ such that
$d_{i+1},\ldots,d_{p} \leq d_i$ and $d_i \leq d_{p+1},\ldots,d_l$ where
$d_{i}$ is considered as the pivot. Then it swaps $d_i$ with $d_p$. Now
we get $d$ such that $d_{i},\ldots,d_{p-1} \leq d_p$ and
$d_p \leq d_{p+1},\ldots,d_l$.

Next, we sort $d_{i},\ldots,d_{p-1}$ and $d_p \leq d_{p+1},\ldots,d_l$.

Hence we can write,

quicksort($d_i,\ldots,d_l$) =
(quicksort($d_i,\ldots,d_{p-1}$)$,d_p,$quicksort($d_{p+1},\ldots,d_l$))
By induction hypothesis, quicksort($d_i,\ldots,d_{p-1}$ gives us
$d_i \leq \ldots \leq d_l$ and quicksort($d_{p+1},\ldots,d_l$ gives us
$d_{p+1} \leq \ldots \leq d_l$. Hence, finally we get
$d_i \leq \dots \leq d_{p-1} \leq d_p \leq d_{p+1} \leq \ldots \leq d_l$.

  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --
                                                                                                                                                      **Operation & **Relation between the cells\ 
                                                                                                                       begin quicksort $d_i, \ldots, d_{l}$ & $d_i \not\leq \ldots \not\leq d_l$\ 
                                                                                                                              begin placement for <span>$d_i$</span> in & $d_{i+1}, \ldots, d_l$\ 
                                                                                                               after placement & $d_{i+1},\ldots,d_p \leq d_i$ and $d_i \leq d_{p+1},\ldots,d_l$\ 
    swap <span> $d_i$</span> and <span> $d_p$</span> & <span> ($d_i \not\leq \ldots \not\leq d_{p-1}$)</span> <span>$\leq d_p \leq$</span> <span>($d_{p+1} \not\leq \ldots \not\leq d_l$)</span>\ 
              quicksort <span> $d_i, \ldots, d_{p-1}$</span> & <span> ($d_i \leq \ldots \leq d_{p-1}$)</span> <span>$\leq d_p \leq$</span> <span>($d_{p+1} \not\leq \ldots \not\leq d_l$)</span>\ 
                       quicksort <span>$d_{p+1}, \ldots, d_l$</span> & <span> ($d_i \leq \ldots \leq d_{p-1}$)</span> <span>$\leq d_p \leq$</span> <span>($d_{p+1} \leq \ldots \leq d_l$)</span>\ 
                                                                                                                                                           finally, & $d_i \leq \ldots \leq d_l$\ 
                                                                                                                                                                                             **** 
  ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --

**List of Prime Numbers: Sieve of Eratosthenes**
------------------------------------------------

The sieve of Eratosthenes is one of the most efficient ways to find all
of the smaller primes. Use less number of <span>LEN</span>, if memory
problems or stack overflow occurs.

    #include<stdio.h>
    #define LEN 100000
    int fillOne(char *data, int index){
        data[index] = 1;
        return 0;
    }
    int printCell(char *data, int index){
        if(data[index]==1)
            printf("%d, ", index);
        return 0;
    }
    int nothing(int a, int b){
        return 0;
    }

    void setZero(char *data, int i, int interval){
        if(i < LEN){
            data[i] = 0;
            setZero(data, i+interval, interval);
        }
    }
    int eachPrime(char *data, int index){
        int i;
        if(data[index] == 1)
            setZero(data, index+index, index);
    }

    int fRepeat( char *data, int i,
                    int (*each)(char*, int),
                    int (*agg)(int, int),
                    int (*last)(char*, int)){
        if(i >= LEN-1)
            return last(data, i);
        else
            return agg( fRepeat(data, i+1,
                                each, agg, last),
                        each(data, i));
    }

    int main(){
        char data[LEN];
        int r, key;
        fRepeat(data, 0, fillOne, nothing, fillOne);
        data[0] = 0;
        data[1] = 0;
        fRepeat(data, 2, eachPrime, nothing, eachPrime);
        //Print the prime numbers
        fRepeat(data, 0, printCell, nothing, printCell);
    }

Note that this program is quicker than the program we presented earlier.
Because, it avoids any multiplicative (multiplication or modulus or
division) operation.

However, it is enough to repeat the function <span>eachPrime</span> only
$\sqrt{LEN}$ times. Therefore, we can bring small changes to our program
as follows –

    #include<stdio.h>
    #include<math.h>
    #define LEN 100000
    int fillOne(char *data, int index){
        data[index] = 1;
        return 0;
    }
    int printCell(char *data, int index){
        if(data[index]==1)
            printf("%d, ", index);
        return 0;
    }
    int nothing(int a, int b){
        return 0;
    }
    void setZero(char *data, int i, int interval){
        if(i < LEN){
            data[i] = 0;
            setZero(data, i+interval, interval);
        }
    }
    int eachPrime(char *data, int index){
        int i;
        if(data[index] == 1)
            setZero(data, index+index, index);
    }
    int fRepeat( char *data, int i, int l,
                    int (*each)(char*, int),
                    int (*agg)(int, int),
                    int (*last)(char*, int)){
        if(i >= l-1)
            return last(data, i);
        else
            return agg( fRepeat(data, i+1, l,
                                each, agg, last),
                        each(data, i));
    }

    int main(){
        char data[LEN];
        int r, key;
        //fill array with 1
        fRepeat(data, 0, LEN, fillOne, nothing, fillOne);
        data[0] = 0;
        data[1] = 0;
        fRepeat(data, 2, sqrt(LEN)+1, eachPrime, nothing, eachPrime);

        fRepeat(data, 0, LEN, printCell, nothing, printCell);
    }

Prove the correctness of all the functions for the algorithm
<span>****</span> Sieve of Eratosthenes.

Next Goal
=========

We intends to extend this book to include the following topics in near
future (next edition).

- Stack

- Queue

- Linked List

- Tree

- Graph

- Hash Table

- Binary Search tree

- Red Black tree

- Heap Sort (heap)

- Raddix Sort

- Bucket sort

- Rod Cutting (DP)

- Counting the number of parenthesizations

- Greedy algithm

- Amortized algorithm

- Multitheaded Algorithm

- Matrix Operations

- Number theoretic algorithms

- String matching algorithms

- Computational Geometry

- Hamiltonian Cycle

- Vertex Cover Problem


