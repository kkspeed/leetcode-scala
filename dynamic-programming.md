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

# Trivial Dynamic Programming
Some dynamic programming only requires taking a look at recent finite number of inputs.
Such question can be solved with easy <tt>array.sliding(n).foldLeft()</tt> or use
<tt>scanLeft</tt> if you need intermediate results (e.g. check max, min etc).

## Max Consecutive Ones
[LeetCode 485](https://leetcode.com/problems/max-consecutive-ones/description/):
Given a binary array, find the maximum number of consecutive 1s in this array.

This problem could be trivially solved with one-liner:

```scala
def findMaxConsecutiveOnes(nums: Array[Int]): Int =
  nums.scanLeft(0)((m, x) => if (x == 0) 0 else m + 1).max
```

## Min Cost Climbing Stairs
[LeetCode 746](https://leetcode.com/problems/min-cost-climbing-stairs/description/):
On a staircase, the i-th step has some non-negative cost cost[i] assigned (0 indexed).
Once you pay the cost, you can either climb one or two steps. You need to find minimum
cost to reach the top of the floor, and you can either start from the step with index 0,
or the step with index 1. 

Notice that we are only interested in state i - 1 and i - 2, since they are the only states
that determines i. Thus, we can use <tt>cost.sliding(2)</tt> to look at <tt>i - 1, i - 2</tt>.
Then for accumulated cost, we update the costs accordingly:

```
cost[i] = min(cost[i - 1] + c[i - 1], cost[i - 2] + c[i - 2])
```

```scala
def minCostClimbingStairs(cost: Array[Int]): Int =
  cost.sliding(2).foldLeft((0, 0)) {
    case((a0, a1), Array(b0, b1)) => (a1 min a0 + b0, a0 + b0 min a1 + b1)
  }._2
```

We just need to memorize the number of consecutive ones at current point. Then select
the max value.

# Imperative Dynamic Programming
Scala's array can be mutable. Thus you can write dynamic programming code in the traditional
way by using <tt>for</tt> loops.

## Coin Path
[LeetCode 656](https://leetcode.com/problems/coin-path/description/)

For each location, you can store the smallest path (ordered by number of jumps and / or lexical
order of the path).

```scala
def cheapestJump(A: Array[Int], B: Int): List[Int] = {
  import scala.math.Ordering.Implicits._
  val dp = A.map(_ => (Int.MaxValue, List[Int]())).updated(0, (A(0), List(0)))
  for (i <- A.indices; if A(i) != -1 && dp(i)._2.nonEmpty;
       j <- 1 to B; if i + j < A.length && A(i + j) != -1)
    dp(i + j) = dp(i + j) min (dp(i)._1 + A(i + j), dp(i)._2 :+ (i + j))
  dp.last._2.map(_ + 1)
}
```

# Top Down Dynamic Programming
Dynamic programming formulas are typically written in the form like <tt>f(n) = f(n-1) + ...(fn-i)</tt>.
Thus, it's natural to consider a top-down approach.

## Russian Doll Envelops
[LeetCode 354](https://leetcode.com/problems/russian-doll-envelopes/description/):
Given a list of envelopes in form of <tt>(width, height)</tt>. An envelope <tt>A</tt> can fit into <tt>B</tt>
iff <tt>A.width < B.width && A.height < B.height</tt>.

This problem is naturally expressed as the following formula:
```
Let E[R] represents the number of envelopes that can be put into R.
E[R] = max{E[R1], E[R2]...} + 1 if Ri can fit into R.
```

To implement a topdown memoization, we use a mutable map to memorize the answer to the subproblem:

```scala
def maxEnvelopes(envelopes: Array[Array[Int]]): Int = {
  if (envelopes.isEmpty) return 0
  val dolls = envelopes.map(m => (m(0), m(1))).toList
  val map = scala.collection.mutable.Map[(Int, Int), Int]()
  def envel(d: (Int, Int)): Int =
    if (map.contains(d)) map(d)
    else {
      val candidates = dolls.filter { case (x, y) => x < d._1 && y < d._2 }
      if (candidates.isEmpty) map(d) = 1
      else map(d) = 1 + candidates.map(envel).max
      map(d)
    }
  dolls.map(envel).max
}
```

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

Now lazy dynamic programming just follows the pattern:

```scala
def go(i: Int, j: Int, ...): Lazy[Type] = Lazy {
    do computation
}

lazy val mem = Array.tabulate[Lazy[Type]](indices...)(go)
```

## Fibonacci Number
A trivial example that computes the nth Fibonacci number.

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

We use A[i, j] to denote the edit distance of word1[:i] and word2[:j].
Then we have:

```
A[i, j] = j if i == 0
A[i, j] = i if j == 0
A[i, j] = A[i - 1, j - 1] if word1[i] == word2[j]
A[i, j] = min(A[i, j - 1], A[i - 1, j], A[i - 1, j - 1]) + 1 otherwise
```

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

## Binary Tree with Factors
[LeetCode 823](https://leetcode.com/problems/binary-trees-with-factors/description/)

Given an array of unique integers, each integer is strictly greater than 1.

We make a binary tree using these integers and each number may be used for any number of times.

Each non-leaf node's value should be equal to the product of the values of it's children.

How many binary trees can we make?  Return the answer modulo 10 ** 9 + 7.

```
Example 1:
Input: A = [2, 4]
Output: 3
Explanation: We can make these trees: [2], [4], [4, 2, 2]

Example 2:
Input: A = [2, 4, 5, 10]
Output: 7
Explanation: We can make these trees: [2], [4], [5], [10], [4, 2, 2], [10, 2, 5], [10, 5, 2].
```

Let <tt>mem(i)</tt> denote the number of trees with root <tt>A(i)</tt>. Then:

```
mem(i) = max(1, sum(for k in [0 .. i - 1] A(k) * mem(indexOf(A(i) / A(k)))  if A(i) / A(k) is in A))
```

It means if we find A(k) * A(j) = A(i), then there must exist binary tree(s), with A(i) at root and
A(k), A(j) being the roots of left and right subtrees.

We use lazy dynamic programming:

```scala
def numFactoredBinaryTrees(A: Array[Int]): Int = {
  val S = A.sorted
  val revMap = S.zipWithIndex.toMap
  val mod = 1000000007l
  def doNumFactored(i: Int): Lazy[Long] = Lazy {
    (0 until i).filter(j => S(i) % S(j) == 0 && revMap.contains(S(i) / S(j)))
      .map(j => (mem(j)() * mem(revMap(S(i) / S(j)))()) % mod)
      .foldLeft(1l)(_+_) % mod
  }
  lazy val mem = Array.tabulate(S.length)(doNumFactored)
  (mem.map(_()).sum % mod).toInt
}
```

## Maximum Length of Pair Chain 
[LeetCode 646](https://leetcode.com/problems/maximum-length-of-pair-chain/description/)

You are given n pairs of numbers. In every pair, the first number is always smaller than the second
number.

Now, we define a pair (c, d) can follow another pair (a, b) if and only if b < c. Chain of pairs can be
formed in this fashion.

Given a set of pairs, find the length longest chain which can be formed. You needn't use up all the given
pairs. You can select pairs in any order.  

The idea is to use dynamic programming. Chain[[a, b]] = max Chain[[c, d]] where d < a + 1.

```scala
def findLongestChain(pairs: Array[Array[Int]]): Int = {
  val sorted = pairs.sortBy(_(0))
  def doLongest(i: Int): Lazy[Int] = Lazy {
    ((0 until i).filter(sorted(_)(1) < sorted(i)(0)) match {
      case Seq() => 0
      case xs => xs.map(mem(_)()).max
    }) + 1
  }
  lazy val mem = Array.tabulate(sorted.length)(doLongest)
  mem.map(_()).max
}
```

This problem could be solved with greedy:

- Sort by ending
- For each two intervals, if they intersect, drop the latter one.

```scala
def findLongestChain(pairs: Array[Array[Int]]): Int =
  pairs.sortBy(_(1)).foldLeft[List[Array[Int]]](Nil) {
    case (x::xs, i) if (i(0) <= x(1)) => x::xs
    case (xs, i) => i::xs
  }.length
```
