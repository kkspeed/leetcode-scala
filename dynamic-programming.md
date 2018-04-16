Dynamic Programming
===

# Foreword
Doing dynamic programming in Scala can be generally no different from doing it in 
other language. You can do imperative dynamic programming with array. You can do
top-down dynamic programming by memoization or in certain situations, you can utilize
laziness to do dynamic programming functionally.

# Prerequisite
Make sure you know laziness in Scala.
I find [Week 2 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/2)
very accessible 

# Imperative Dynamic Programming
# Top Down Dynamic Programming
# Lazy Dynamic Programming
The idea is from [this article](http://jelv.is/blog/Lazy-Dynamic-Programming/), which
talks about lazy dynamic programming in Haskell. With a little bit of tweak, we can
adapt it to Scala.

## A Lazy Container
You cannot put call-by-name type into array directly in Scala. You'll need to wrap it
in a custom-defined lazy type:

```scala
class Lazy[T] (expr : => T) {
  lazy val value = expr
  def apply(): T = value
}
object Lazy{ def apply[T](expr : => T) = new Lazy(expr) }
```

## Fibonacci Number
```scala
def fib(n: Int): Int = {
  def doFib(i: Int): Lazy[Int] = Lazy {
    if (i <= 2) 1
    else fibs(i - 1)() + fibs(i - 2)()
  }
  lazy val fibs = Array.tabulate[Lazy[Int]](n)(doFib)
  doFib(n)()
}
```

## Edit Distance
[LeetCode 72](https://leetcode.com/problems/edit-distance/description/)
Given two words word1 and word2, find the minimum number of steps required
to convert word1 to word2. (each operation is counted as 1 step.)
You have the following 3 operations permitted on a word:

1. Insert a character
2. Delete a character
3. Replace a character

```scala
def editDistance(s1: String, s2: String): Int = {
  def go(i: Int, j: Int): Lazy[Int] = Lazy {
    if (i == 0) j
    else if (j == 0) i
    else if (s1(i - 1) == s2(j - 1)) mem(i - 1)(j - 1)()
    else (mem(i - 1)(j)() + 1) min (mem(i)(j - 1)() + 1) min (mem(i - 1)(j - 1)() + 1)
  }
  lazy val mem = Array.tabulate[Lazy[Int]](s1.length + 1, s2.length + 1)(go)
  go(s1.length, s2.length)()
}
```

## Interleaving String
[LeetCode 97](https://leetcode.com/problems/interleaving-string/description/)
Given s1, s2, s3, find whether s3 is formed by the interleaving of s1 and s2.

For example, Given:

1. s1 = "aabcc",
2. s2 = "dbbca",

When s3 = "aadbbcbcac", return true.

When s3 = "aadbbbaccc", return false.


```scala
def isInterleave(s1: String, s2: String, s3: String): Boolean = {
  def go(i: Int, j: Int): Lazy[Boolean] = Lazy {
    (i, j) match {
      case (0, 0) => true
      case (0, _) => s2(j - 1) == s3(j - 1) && mem(0)(j - 1)()
      case (_, 0) => s1(i - 1) == s3(i - 1) && mem(i - 1)(0)()
      case _  if s1(i - 1) == s2(j - 1) && s1(i - 1) == s3(i + j - 1) =>
          mem(i - 1)(j)() || mem(i)(j - 1)()
      case _ if s1(i - 1) == s3(i + j - 1) => mem(i - 1)(j)()
      case _ if s2(j - 1) == s3(i + j - 1) => mem(i)(j - 1)()
      case _ => false
    }
  }
  lazy val mem = Array.tabulate[Lazy[Boolean]](s1.length + 1, s2.length + 1)(go)
  (s1.length + s2.length == s3.length) && go(s1.length, s2.length).value
}
```
