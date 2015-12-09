_Originally, I wanted to write a complete guide to lazy evaluation, but then._

Lazy evaluation is the most widely used method for executing Haskell program code on a computer. It determines the time and memory usage of Haskell programs, and it allows new and powerful ways to write modular code. To make full use of purely functional programming, a good understanding of lazy evaluation is very helpful.

Here at HackHands, we have a series of tutorials to help you get a thorough introduction to this important topic:

1. [How does Lazy Evaluation Work in Haskell?](https://hackhands.com/lazy-evaluation-works-haskell/ "How Lazy Evaluation Works in Haskell") — Start here. This tutorial explains how expressions are evaluated using lazy evaluation, and what that means for time and memory usage.
2. [Modular Code and Lazy Evaluation in Haskell](https://hackhands.com/modular-code-lazy-evaluation-haskell/ "Modular Code and Lazy Evaluation in Haskell") — Why use lazy evaluation in the first place? This tutorial explains the power of lazy evaluation and how it can help us to write clear and modular code. In particular, it will tell you about **infinite lists**.
3. [Haskell's Non-Strict Semantics – What Exactly does Lazy Evaluation Calculate?](https://hackhands.com/non-strict-semantics-haskell) – This tutorial explains how we can avoid to do lazy evaluation in detail when trying to understand a Haskell program, via a method called **denotational semantics**. It also covers the important notion of a **strict** function, which shows up in many discussions about optimizing Haskell programs.

Let me know in the comments if you have questions or would like me to cover additional topics. Of course, you can always ask for live help as well.

