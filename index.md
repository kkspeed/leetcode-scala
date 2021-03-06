Learn Scala by Practicing on LeetCode
===

# Introduction
## Why
[LeetCode](http://leetcode.com) is arguably one of the best known site for
(potential) engineers to sharpen their skills on interviews. The most popular
languages are, without any doubt, C++, Java and Python. But LeetCode also
supports other languages, like Kotlin, Swift, Go and Scala. Although most
of these languages have provided more-or-less functional programming
capability, Scala is the only one that places such high emphsize on that.
Thus, solving problems in Scala can be very different from doing that in
any other language available on LeetCode -- for the better or worse. This
documentation tries to explore how Scala can be used to solve the interview
problems. I'll try to present solutions as functional as possible but
unavoidably, some impure code would slip in, either due to the nature of the
problem or due to my limited knowledge of the functional world.

## Structure of this documentation
There are two dimensions I want to explore:

1. Organizing problems by features of Scala, from easiest to more advanced
2. Organizing problems by algorithms used, from easy ones to more tricky ones

As for reference materials, I highly recommend Functional Programming in Scala
courses on Coursera. The first 2 courses -- 
[Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1)
and [Functional Program Design in Scala](https://www.coursera.org/learn/progfun2)
are the most important ones. In fact, a lot of the solutions are directly
based on ideas from these 2 courses. I'll make recurring reference at each chapter
to these two courses as well. As a secondary quick reference, the book
[Scala Succinctly](https://www.syncfusion.com/ebooks/scala_succinctly) by SyncFusion
is recommended.

The documentation is (planned to be) divided into the following sections:


1. [Basic Recursion](basic-recursion.md): 
    This section talks about trivial problems using only very basic function calls
2. [Collections](collections.md): This section talks about Scala collections (hopefully) in depth
3. [Priority Queue](priority-queue.md): Introduces a functional priority implementation.
4. [Search](search.md): This section explores mostly depth-first search and breadth-first search
5. [Dynamic Programming](dynamic-programming.md): This section talks about how to solve dynamic
   programming problems in Scala
6. [Parsing](parsing.md): This section explores unique approach to parsing problems with Parser Monad
7. Functional Reactive Programming: This section presents FRP approach to a small number of
   (currently only one :p) problems.

## Moral
Most of the examples in this documentation will be selected from LeetCode. I try to use
free problems as much as possible. For these problems, I may or may not directly present
them in this documentation. For those problems that currently require subscription, I'll
provide link to LeetCode without mentioning the problem in detail in this doc.

## The Pitfall of Scala on LeetCode
There are a few drawbacks to using Scala on LeetCode:
1. Most structures are directly translated from Java. Thus, a tree is a class with a value,
   left, right pointers instead of a case class. Most inputs are arrays, strings instead
   of lists, making pattern matching a bit painful.
2. Compilation can sometimes fail. This possibly is related to Scala's lengthy compilation time.
   When this happens, most likely a re-submit could fix it.

## The Good Part of Using Scala
1. It's fun! It's really a joy to come up with a functional solution.
2. Scala's powerful collection APIs and features like pattern matching often results in
   compact and easy to read code.

# License
The documentation is licensed under
[Creative Commons License](https://creativecommons.org/licenses/by/4.0/).
The code is licensed under [MIT](https://opensource.org/licenses/MIT) license.
