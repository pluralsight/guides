_This tutorial is part of a larger series, [The Incomplete Guide to Lazy Evaluation (in Haskell)](https://hackhands.com/guide-lazy-evaluation-haskell/ "The Incomplete Guide to Lazy Evaluation (in Haskell)")._

## Haskell’s Non-Strict Semantics – What Exactly does Lazy Evaluation Calculate?

Lazy evaluation is the most widely used method for executing Haskell program code on a computer. In previous tutorials, we have looked at [how lazy evaluation works in detail](https://hackhands.com/lazy-evaluation-works-haskell/) and [how it can help with making our code simpler and more modular](https://hackhands.com/modular-code-lazy-evaluation-haskell/). However, the main drawback of lazy evaluation is that it can be quite hard to understand all details of how exactly the program is evaluated. How, then, we can understand what the program does in the first place?

In this tutorial, I want to explain how we can understand a Haskell program without having a clue about lazy evaluation at all. This is possible thanks to Haskell’s **non-strict semantics**. First, we will introduce the concept of **denotational semantics** and the special value “⊥”, which is called “**bottom**”. Next, we will look at the concepts of **strictness** and **non-strictness**, which often show up in discussions about the optimization of Haskell programs. Afterwards, we will learn how to understand infinite lists without using lazy evaluation. Finally, we will consider the illuminating example of the children’s game “rock-paper-scissors” – which I learned from the classic book [An Introduction to Functional Programming](http://books.google.de/books?id=Op5QAAAAMAAJ) by R. Bird and P. Wadler – to see how this method works in practice.

The key message is that there are different levels at which we can understand a program. Lazy evaluation tells us **_how_** a Haskell program is executed, which is very useful for understanding its time and memory usage. However, we are also interested in **_what_** a program calculates, which is important for showing that the program is correct and free of bugs. In fact, I would say that the latter is more important than the former – there is no point in calculating a wrong result very fast. In most traditional imperative languages, the _what_ and the _how_ are very similar, but in Haskell, these are two different and complementary viewpoints. Often, the _what_ part is easier to understand than the _how_ part, because for the former, we may replace any expression by one that is _equal_ to it, whereas for the latter, no two expressions are equal, their evaluation always proceeds in slightly different ways. In a way, the idea of separating the _how_ from the _what_ is what purely functional programming is all about.

The _how_ of a program also goes under the monicker of **operational semantics**, whereas the _what_ is also called **denotational semantics**. The word **semantics** simply refers to the “meaning” of a program. In this tutorial, we are concerned with the denotational semantics of Haskell and how it relates to operational semantics, i.e. lazy evaluation.

## Expressions, their Denotations and Bottom

Consider any Haskell expression, for example

``` haskell
40 + 2
```

The meaning or **denotation** of this expression is the number _42_. The expressions

``` haskell
42
2*21
sum [1..8] + 6
```

also **denote** the same value, _42_. Note that we carefully distinguish between an _expression_, which is a syntactically legal piece of Haskell code, written in typewriter face, and a _value_, which is a mathematical object, written in italics. So, the expression `42` denotes the value _42_, but these are different things. This may seem like hair-splitting, but it’s a very useful distinction to keep in mind. The point is that different expressions, like `42`, `40+2` and `2*21`, may denote the same thing, _42_. In this case, we write an _equal_ sign

``` haskell
_42 = 40+2 = 2*21_
```

to indicate that while the expressions are different, they denote the same value.

The rule is that two expressions denote the same thing if they evaluate to the same normal form. When running a Haskell program, the normal form is the final result that we care about. It represents _what_ is being calculated.

However, the beautiful thing about denotational semantics is that we can also give meaning to expressions that do _not_ have a normal form, for instance because their evaluation goes into an infinite loop. For example, evaluation of the expression

``` haskell
let loop = loop + 1 in loop
```

will never terminate. What value do we assign to it? The solution is very simple: We introduce a new value written “⊥” and called “**bottom**”, which represents the value of a computation that, well, “does not have a value”, for instance because it goes to an infinite loop or because it throws an exception. In our example, the expression `loop` denotes the value ⊥, because its evaluation never terminates. Another example would be the expression

``` haskell
head []
```

which also denotes the value ⊥ (albeit of a different type), because the head of an empty list is undefined and trying to evaluate this expression will throw an exception. In fact, Haskell has a predefined expression

``` haskell
undefined
```

which always throws an exception and thus denotes the value ⊥.

For each type, we can make a list of possible semantic values. For instance, an expression of type `Bool` may denote one of the values ⊥, `_False_` or `_True_`. An expression of type `Integer` may denote one of the values ⊥, `_1_`, `_2_`, … and so on. Essentially, we just have to add the extra value ⊥ to each type, at least for simple types like `Bool` and `Integer`. Later, however, we will see that more complex types, like the list type `[Int]` will contain additional extra values.

## Strict Functions and Non-Strict Semantics

Once again, let us consider the example of the logical AND, whose definition is

``` haskell
(&&) :: Bool -> Bool -> Bool
True && x = x
False && x = False
```

To understand _what_ this function does, we can write down a truth table. However, this time, we also have to consider the new value ⊥, which indicates that an argument does not terminate. So, the truth table we consider here has not just 4, but 9 entries. The first rows of the table are:

``` haskell
⊥ && ⊥ = ⊥
⊥ && True = ⊥
⊥ && False = ⊥
```

The reasoning is that since the function performs a pattern match on the first argument, the function will not terminate if the evaluation of the first argument does not terminate. The next rows are

``` haskell
True && ⊥ = ⊥
True && True = True
True && False = False
```

These simply mimic the first equation in the definition of the function. The final rows of the truth table are the most interesting:

``` haskell
False && ⊥ = False
False && True = False
False && False = False
```

These rows mimic the second equation of the function definition. Note that thanks to lazy evaluation, the function returns the result `False` in all cases, even when the second argument is ⊥. In contrast, in a language with eager evaluation, like LISP or O’Caml, the first line would have to read “`False && ⊥ = ⊥`”, because arguments are always evaluated first, and evaluation of the function application does not terminate if evaluation of the argument does not terminate. We say that these languages have **strict semantics**, whereas Haskell has **non-strict semantics**.

(A word of caution: The equations above look like we might be able to “pattern match” on the value ⊥. But this is not possible because ⊥ is not a legal Haskell _expression_; it only exists in the denotational semantics. Remember: we write expressions in typeface, but values in italics.)

Where do the monickers _strict_ and _non-strict_ come from? Here is the main definition: A function `f` is called **strict** if it satisfies

``` haskell
f ⊥ = ⊥
```


In other words, evaluation of the function does not terminate if evaluation of the argument does not terminate. In most cases, this is because the function performs a pattern match on the argument. For a function `g` with, say, two arguments, we say that it is strict in its second argument whenever `g x ⊥ = ⊥` for any first argument `x`; likewise for other positions.

Now, in Haskell, the function `(&&)` is _not_ strict in its second argument, because

``` haskell
False && ⊥ = False ≠ ⊥
```

We also say that it is **lazy** in its second argument. Haskell is said to have _non-strict semantics_ because it allows such functions, whereas in a language with _strict semantics_, all functions are strict.

The fact that this function is not strict also means that the function does not always evaluate its second argument, because otherwise, well, the result would have to be ⊥. This is the connection between lazy evaluation, the _how_, and non-strict denotational semantics, the _what_. The latter only cares about end results, though, and is thus easier to work with. In fact, the [Haskell standard](https://www.haskell.org/onlinereport/haskell2010/haskellch1.html#x6-90001) only specifies that a compiler needs to respect the non-strict semantics, it does not say that Haskell needs to be evaluated using lazy evaluation. In other words, Haskell compilers are free to employ some other means, but in practice, the [Glasgow Haskell Compiler (GHC)](https://www.haskell.org/ghc/) does use lazy evaluation (with some minor adjustments). Clearly, this is important for understanding time and space usage of a program compiled with GHC.

Even though they are part of denotational semantics, the notions of strict functions and lazy functions are often used to discuss matters from operational semantics. For instance, [bang patterns](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/bang-patterns.html) are an extension of the Haskell language that allow you to specify that some function arguments are to be evaluated to weak head normal form (see the [previous tutorial on how lazy evaluation works](https://hackhands.com/lazy-evaluation-works-haskell/) for more on this normal form) before the function body is evaluated, mimicking eager evaluation. If you look at the [GHC user manual](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/bang-patterns.html), you will find that this is often described in terms of “strict” and “lazy”. For instance, it says that the bang pattern in the function definition

``` haskell
f1 !x = True
```

makes the function strict, so that we now have `_f1 ⊥ = ⊥_` instead of `_f1 ⊥ = True_`. No statement in the manual is incorrect, but I would like to caution the reader again that strictness is a concept from denotational semantics, whereas bang patterns affect the operational semantics. In informal discussions, the word “strict” is often used to mean that a function evaluates its argument to [WHNF](https://hackhands.com/lazy-evaluation-works-haskell#normal-form-and-weak-head-normal-form) before proceeding, but “strictly speaking”, this is not a correct use of the word “strict”.

Another point where you may encounter the notion of strictness is an optimization pass called [strictness analysis](https://en.wikipedia.org/wiki/Strictness_analysis). Here, the idea is that the compiler tries to figure out which functions are strict, and compiles them to use eager evaluation instead of lazy evaluation. This is safe to do, because it does not change their denotational semantics. The reason for doing so is that lazy evaluation does incur a certain administrative overhead which can be avoided by using eager evaluation; the code will run faster.

## Partial and Infinite Lists

We have seen that simple types like `Integer` contain both the ordinary values and the special value ⊥. In the case of more complex types, however, the value ⊥ can appear in many more places. For example: What value does the expression

``` haskell
2 : undefined
```

have? It appears to be a list whose head element is `_2_`, but whose tail is undefined. And that’s indeed the case, its value is

``` haskell
2 : ⊥
```

The point is that the list _constructor_ is _not_ strict, so the “totally undefined” list ⊥ is different from the “partially defined list” `2 : ⊥`. For instance, if we apply the function `head`, we get

``` haskell
head ⊥ = ⊥
head (2 : ⊥) = 2
```

In Haskell, all constructors are lazy (unless indicated otherwise by a [strictness annotation](https://www.haskell.org/onlinereport/haskell2010/haskellch4.html#x10-710004.2.1)).

We have already [discussed the usefulness of infinite lists](https://hackhands.com/modular-code-lazy-evaluation-haskell/), but so far, we could only explain how lazy evaluation allows us to write down expressions for “potentially infinite” lists. Now, denotational semantics, and the just introduced concept of partial lists, allows us to talk about _actually infinite_ lists. The idea is that an infinite list is to be understood as a _limit_ of partial lists. Just like the number _π=3.1415…_ is mathematically a limit of successive approximations _3_, _3.1_, _3.14_ and so on, an infinite list, like the list of all positive integers,

``` haskell
1 : 2 : 3 : …
```

is a limit of partial lists

``` haskell
⊥
1 : ⊥
1 : 2 : ⊥
1 : 2 : 3 : ⊥
…
```

and so on. To see how this works, consider the expression

``` haskell
filter (< 3) [1..]
```

To understand the denotation of this expression, we apply the function _filter_ to the partial lists that approximate the infinite argument list:

``` haskell
filter (< 3) ⊥ = ⊥
filter (< 3) (1 : ⊥) = 1 : ⊥
filter (< 3) (1 : 2 : ⊥) = 1 : 2 : ⊥
filter (< 3) (1 : 2 : 3 : ⊥) = 1 : 2 : ⊥
filter (< 3) (1 : 2 : 3 : 4 : ⊥) = 1 : 2 : ⊥
…
```

All this has been calculated by applying the definition of the function `filter`. As you can see, the results stabilize after the third approximation, because there are no other positive integers smaller than the number 3\. Hence, the limit is a partial list

``` haskell
filter (< 3) [1..] = 1 : 2 : ⊥
```

And indeed, if you fire up a Haskell interpreter like [GHCi](https://downloads.haskell.org/~ghc/latest/docs/html/users_guide/ghci.html) and try to evaluate this expression, the interpreter will first output

``` haskell
> filter (< 3) [1..]
[1,2
```

and then not respond to inputs anymore; this is what happens when you try to print the partial list `1 : 2 : ⊥`.

The main message here is that using partial lists, we can understand infinite lists in a way that does not involve lazy evaluation. We had already noted in [our previous discussion of infinite lists](https://hackhands.com/modular-code-lazy-evaluation-haskell/) that the latter adds many additional complications, like the need to distinguish between the expressions `(0+1)` and `1`, and so on. Working on the level of denotational semantics allows us to understand infinite lists in a more practical and intuitive way as actually infinite.

Of course, the denotational semantics do mimic the reductions steps taken by lazy evaluation to some extend. In particular, the value ⊥ does feel a bit like an “unevaluated” expression, as it gets refined from ⊥ to `1 : ⊥` to `1 : 2 : ⊥` and so on. It is a good idea to update our intuition of the value ⊥, and to think of it as a “computation that has not finished yet”. Sometimes, this means that the computation may never finish, because it goes into an infinite loop. At other times, it is better to imagine that it may yield more information after some time, for instance when the list ⊥ is refined to the list `1 : ⊥`. From this point of view, a strict function is a function that cannot reveal any information about its result before it has gained more information about its argument.

## Rock-Paper-Scissors

To gain more practice with infinite lists, we will now consider the children’s game “[rock-paper-scissors](https://en.wikipedia.org/wiki/Rock-paper-scissors)”, a classic example which I learned from the book [An Introduction to Functional Programming](http://books.google.de/books?id=Op5QAAAAMAAJ) By R. Bird and P. Wadler.

You probably know the rules of the game. In every round, each player makes one hand sign: either “rock”, “paper” or “scissors”. We represent this as an algebraic data type

``` haskell
data Sign = Rock | Paper | Scissors
```

![](assets/300px-Rock-paper-scissors.svg_.png)
_(Image CC-BY-SA from [Wikipedia](https://en.wikipedia.org/wiki/File:Rock-paper-scissors.svg))_

Each sign is beaten by another sign

``` haskell
beat :: Sign -> Sign
beat Rock = Paper
beat Paper = Scissors
beat Scissors = Rock
```

and we give a point to the player whose sign beats the other player’s sign

``` haskell
points :: (Sign, Sign) -> (Int,Int)
points (x,y)
    | x == y = (0,0) -- draw
    | x == beat y = (1,0)
    | y == beat x = (0,1)
```

Now, we assume that each player’s strategy is represented by a type `Strategy` and that there is a function

``` haskell
rounds :: (Strategy, Strategy) -> [(Sign, Sign)]
```

which takes two strategies and returns an infinite list of rounds where they play against each other. Then, we can use the following function to play a match of `n` rounds and tally the points:

``` haskell
match :: Int -> (Strategy, Strategy) -> (Int, Int)
match n =
    pair sum . unzip . map points . take n . rounds
    where
    pair f (x,y) = (f x, f y)
```

### Strategies 1

What is a strategy? It is a method used by one player to decide which sign to play, based on all the previous moves by the other player. We can represent this by a function type

``` haskell
type Strategy = [Sign] -> Sign
```

Given a list of the opponent’s previous moves, this function calculates the player’s current move. For instance, the strategy that always plays “paper” is represented as

``` haskell
alwaysPaper :: Strategy
alwaysPaper signs = Paper
```

Another example is the strategy that plays “paper” first, and then plays the move that would have beaten the opponent in the previous round:

``` haskell
whatYouDid :: Strategy
whatYouDid [] = Paper
whatYouDid signs = beat (last signs)
```

Finally, we need to write the function `rounds`, which plays two strategies against each other and generates an infinite lists of sign pairs. We can use the function [iterate](http://www.letu.edu/people/jaytevis/Programming-Languages/Haskell/tourofprelude.html#iterate) to repeatedly append a new move to the list of previous moves

``` haskell
rounds :: (Strategy, Strategy) -> [(Sign, Sign)]
rounds (player1, player2) =
    map lastOfPair $ tail $ iterate update ([],[])
    where
    update (signs1, signs2) =
        ( signs1 ++ [player1 signs2]
        , signs2 ++ [player2 signs1] )
    lastOfPair (x,y) = (last x, last y)
```

Unfortunately, this implementation of the `Strategy` type has a serious drawback: Usually, a strategy needs _O(n)_ time to analyze the previous _n_ moves. For example, this is the case for the strategy `whatYouDid`. Then, to calculate the first _n_ moves with this strategy, the function `rounds` will need _1 + 2 + … + n = O(n²)_ time. In other words, it takes quadratic time to calculate a match of _n_ rounds. Certainly, we can do better?

### Strategies 2

For a more efficient implementation of the `Strategy` type, consider the following idea: Instead of calculating just a single move from a list of the opponent’s previous moves, we calculate the infinite list of all moves from the infinite list of all of the opponent’s moves.

``` haskell
type Strategy = [Sign] -> [Sign]
```

For instance, the strategy `alwaysPaper` ignores the opponent’s moves and just returns an infinite lists consisting entirely of the sign `Paper`

``` haskell
alwaysPaper :: Strategy
alwaysPaper signs = repeat Paper
```

The strategy `whatYouDid` first returns `Paper` and then applies the function `beat` to all moves of the opponent

``` haskell
whatYouDid :: Strategy
whatYouDid signs = Paper : map beat signs
```

The key point is that now, this strategy only needs _O(1)_ time to calculate the next move, instead of the _O(n)_ time needed to inspect a list of the _n_ previous moves.

The implementation of the function `rounds` that plays one strategy against the other is particularly interesting:

``` haskell
rounds :: (Strategy, Strategy) -> [(Sign, Sign)]
rounds (player1, player2) = zip signs1 signs2
    where
    signs1 = player1 signs2
    signs2 = player2 signs1
```

The value `signs1` is the infinite list of moves of the first player. Note that it depends on the list `signs2`, which in turn depends on the list `signs1` again. This is also known as **mutual recursion**. How does it work? For concreteness, let us consider the example where the strategy `alwaysPaper` plays against the strategy `whatYouDid`.

The rule is that for any (mutually) recursive definition, we start with the approximation

``` haskell
_signs1 = ⊥
signs2 = ⊥
```

which means that we know nothing about the results yet. Then, we calculate the right-hand side under these assumptions. The result for the list `signs2` is

``` haskell
signs2 = whatYouDid signs1
       = whatYouDid ⊥
       = Paper : map beat ⊥
       = Paper : ⊥
```

(Remember that we consider the example where `player2` is `whatYouDid`.) For the list `signs1`, we get

``` haskell
signs1 = alwaysPaper signs2
       = alwaysPaper ⊥
       = Paper : Paper : Paper : …
```

With this calculation, our knowledge about the result is refined to

``` haskell
signs1 = Paper : Paper : Paper : …
signs2 = Paper : ⊥
```

This is our second approximation to the end result. Note that the result for the first strategy is already complete. To get an even better approximation, we once again calculate the right-hand side, but this time using the new approximation:

``` haskell
signs2 = whatYouDid signs1
       = whatYouDid (Paper : Paper : …)
       = Paper : map beat (Paper : Paper : …)
       = Paper : Scissors : Scissors : …
```

Apparently, our new approximation is already the complete result

``` haskell
signs1 = Paper : Paper : Paper : …
signs2 = Paper : Scissors : Scissors : …
```

Hence, the denotation of the expression `rounds (alwaysPaper, whatYouDid)` is

``` haskell
rounds (alwaysPaper, whatYouDid)
    = (Paper, Paper) : (Paper, Scissors)
    : (Paper, Scissors) : …
```

To summarize, this example shows how we can calculate the result of a lazy computation without using lazy evaluation. The main advantage is that instead of having to track the _how_ of the computation in detail, we can concentrate on the _what_, which means that we can treat different expressions as _equal_, as long as they have the same denotation, something we cannot do when considering how they are evaluated.

### Cheating

The efficient implementation of the `Strategy` type has a serious drawback: It allows strategies that can _cheat_. For instance, the strategy

``` haskell
cheat :: Strategy
chat signs = map beat signs
```

plays exactly the move that would beat the opponent’s move in each round. This is now possible, because the list `signs` contains _all_ moves of the opponent, not just the previous ones.

If we play the strategy `cheat` against the strategy `whatYouDid`, we will get the following sequence of approximations:

``` haskell
signs1 = ⊥
signs2 = ⊥
```

and then

``` haskell
signs1 = cheat ⊥ = ⊥
signs2 = whatYouDid ⊥ = Paper : ⊥
```

and then

``` haskell
signs1 = cheat (Paper : ⊥) = Scissors : ⊥
signs2 = whatYouDid ⊥ = Paper : ⊥
```

and then

``` haskell
signs1 = cheat (Paper : ⊥) = Scissors : ⊥
signs2 = whatYouDid (Scissors : ⊥) = Paper : Rock : ⊥
```

and so on, with the end result being

``` haskell
signs1 = Scissors : Paper : Rock : …
signs2 = Paper : Rock : Scissors : …
```

The strategy `cheat` wins every round, because it “knows” all the moves of the opponent.

Of course, the cheating strategy will fail when `cheat` plays against itself. In this case, the computation will go into an infinite loop and the result will be `signs1 = ⊥` and `signs2 = ⊥`. (Exercise: Show this!)

How can we ensure that a strategy does not cheat? First, we have to think about what exactly cheating is. A strategy does not cheat if it does not “look into the future”, i.e. if it produces the _n_-th hand sign only by looking at the first _n-1_ hand signs of the input list. In other words, a strategy `player` is **fair** if it satisfies the equation

``` haskell
last (take n (player signs)) =
    last (player (take (n-1) signs
```

for all possible lists `sign`. The strategy `cheat` is not fair, as the following example shows

``` haskell
last (take 1 (cheat (Paper : ⊥))) = Scissors
last (cheat (take 0 (Paper : ⊥))) = ⊥
```

Unfortunately, there is no way to automatically check whether a strategy is fair, because we can’t test an infinite number of rounds and also because we cannot even check whether a strategy will terminate at all. The best we can do is to let strategies play as long as they behave fairly, but return ⊥ as soon as one of them starts to cheat. The following function is very helpful for that:

``` haskell
sync :: [a] -> [b] -> [a]
sync [] [] = []
sync (x:xs) (_:ys) = x : sync xs ys
```

The idea is that `sync` essentially returns its first argument, but when the second argument is a partial list, then only a corresponding part of the first argument is returned. Example:

<pre>_sync (Paper : Scissors : ⊥) ('a' : ⊥) = Paper : ⊥_ </pre>

Using this helper function, we can implement the function `rounds` as

``` haskell
rounds :: (Strategy, Strategy) -> [(Sign, Sign)]
rounds (player1, player2) = zip signs1 signs2
    where
    signs1 = player1 (sync signs2 signs1)
    signs2 = player2 (sync signs1 signs2)
```

By “synchronizing” the outputs of the two strategies, each strategy may only inspect as many moves of the opponents as the strategy itself has already decided upon. Any attempt to cheat will result in ⊥ for the next move.

With this, we come to the end of today’s tutorial. I hope the “rock-paper-scissors” example was as helpful to you as it was helpful to me back when I first learned about lazy evaluation and infinite lists.

