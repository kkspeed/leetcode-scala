Searching
===

# Prerequisite
[Week 6 of Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1/home/week/6)
 talks about <tt>for</tt> expression, <tt>map</tt> and <tt>flatMap</tt>.
and [Week 2 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/2),
which talks about lazy streams.

# Permutations
[LeetCode 46](https://leetcode.com/problems/permutations/description/): Generate
permutations of an array that contains no duplicates.

We can think recursively. Given an array A, if we take any element c out of this
array, we have an array A / c. Then we permute A / c. For each of the permutation,
we prepend c to it. This way we get the permutations of A. The code:

```scala
def permuteUnique(nums: Array[Int]): List[List[Int]] =
  nums match {
    case Array() | Array(_) => List(nums.toList)
    case xs => for {
      n <- xs.toList
      rest <- permuteUnique(nums diff Array(n))
    } yield n::rest
  }
```

[LeetCode 47](https://leetcode.com/problems/permutations-ii/description/): Generate
permutations of an array that contains duplicates.

If the array contains duplicates, all we need to pay attention to is that we cannot
blindly take an element c. We'll need to select c from the unique elements of A
to avoid duplications. The previous code only needs a minor change -- 
<tt>xs.toList --> xs.distinct.toList </tt>:

```scala
def permuteUnique(nums: Array[Int]): List[List[Int]] =
  nums match {
    case Array() | Array(_) => List(nums.toList)
    case xs => for {
      n <- xs.distinct.toList
      rest <- permuteUnique(nums diff Array(n))
    } yield n::rest
  }
```

If we fold up the <tt>for</tt> expression, the code can actually be shorten to:

```scala
  def permuteUnique(nums: Array[Int]): List[List[Int]] =
    if (nums.length <= 1) List(nums.toList)
    else nums.distinct.toList.flatMap(
      n => permuteUnique(nums diff Array(n)).map(n::_))
```
But it sacrifices readability and is not recommended.

# Target Sum
[LeetCode 494](https://leetcode.com/problems/target-sum/description/): Given an
array of positive numbers, you can add either <tt>+</tt> or <tt>-</tt> in front
of each number. Given a target sum S, how many ways to assign <tt>+ or -</tt> that
makes the sum equal to S.

Given that the maximum length of the array is 20, we can enumerate through all
the possibilities. The search could be pruned with a minor optimization:
if we see that we cannot make it to the target sum even if we add all <tt>+</tt>
or <tt>-</tt> sign to the rest of numbers, we can abort our search path. The
sum of "rest of the numbers" can be represented using a postfix array which can
be obtained by <tt>nums.scanRight(0)(_+_).init</tt>.

```scala
def findTargetSumWays(nums: Array[Int], S: Int): Int = {
  val postfix = nums.scanRight(0)(_+_).init
  def constructSum(i: Int, sum: Int): Int =
    if (i == nums.length && sum == S) 1
    else if (i == nums.length || sum + postfix(i) < S || sum - postfix(i) > S) 0
    else constructSum(i + 1, sum + nums(i)) + constructSum(i + 1, sum - nums(i))
  constructSum(0, 0)
}
```

# Word Pattern II
[LeetCode 291](https://leetcode.com/problems/word-pattern-ii/description/)

```scala
def wordPatternMatch(pattern: String, str: String): Boolean = {
  def isPatternMatch(pattern: List[Char], str: List[Char],
                     context: Map[Char, List[Char]],
                     used: Set[List[Char]]): Boolean =
    (pattern, str) match {
      case (Nil, Nil) => true
      case (_, Nil) | (Nil, _) => false
      case (p::ps, ss) if context.contains(p) =>
        (ss.take(context(p).length) == context(p)) &&
          isPatternMatch(ps, ss.drop(context(p).length), context, used)
      case (p::ps, ss) =>
        ss.inits.exists(i => !used.contains(i) &&
          isPatternMatch(ps, ss.drop(i.length), context + (p -> i), used + i))
    }
  isPatternMatch(pattern.toList, str.toList, Map(), Set(List()))
}
```
