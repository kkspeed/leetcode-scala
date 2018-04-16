Searching
===

# Prerequisite
[Week 6 of Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1/home/week/6)
 talks about <tt>for</tt> expression, <tt>map</tt> and <tt>flatMap</tt>.
and [Week 2 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/2),
which talks about lazy streams.

# A Bit of Foreword
So, depth first search (DFS) and breadth first search (BFS). The difference between them is how the next state
is selected from currently known candidates. If we use a stack, the next state to explore is the most recently
pushed state. It's depth first search. If we use a queue, the next state is the earliest state available,
which is BFS. If we instead, use a priority queue and organize the states according to a certain metric, we
wind up getting <tt>A</tt>-search or <tt>A-star</tt> search.

## Representing Queues and Stacks
In functional world, stacks are naturally represented with lists. <tt>push(x, stack)</tt> results in <tt>x::stack</tt>.
For queues, we can use 2 lists to achieve constant amortized time for enqueue and dequeue.

[LeetCode 232](https://leetcode.com/problems/implement-queue-using-stacks/description/): Implementing queue using 
stacks. The idea is to use a pair of lists: <tt>(l1, l2)</tt>. An enqueue operation conses an element to one list. At
dequeue, if the other list is not empty, we just return the head of the other list. If it's empty, we pop all elements
from l1 to l2 and dequeue:

```scala
class MyQueue() {
  var queue = (List[Int](), List[Int]())

  /** Push element x to the back of queue. */
  def push(x: Int): Unit =
    queue = (x::queue._1, queue._2)

  /** Removes the element from in front of queue and returns that element. */
  def pop(): Int = {
    var elem: Int = 0
    queue match {
      case (_, y::ys) =>
        elem = y
        queue = (queue._1, ys)
      case (xs, Nil) =>
        queue = (Nil, xs.reverse)
        elem = pop()
    }
    elem
  }

  /** Get the front element. */
  def peek(): Int =
    queue match {
      case (_, y::ys) => y
      case (xs, Nil) =>
        queue = (Nil, xs.reverse)
        queue._2.head
    }

  /** Returns whether the queue is empty. */
  def empty(): Boolean = queue == (Nil, Nil)
}
```

Note that this implementation is not idiomatic functional programming, but it's required by the prototype of
LeetCode for this question.

# For Comprehension
## Ambiguous Coordinates
[LeetCode 816](https://leetcode.com/problems/ambiguous-coordinates/description/): We have a 2D coordinate
"(1, 3)" and then we remove all commas, spaces and decimal points. Given such string, print all the possible
coordinates subject to the following rule that original representation never had extraneous zeroes, so we never
started with numbers like "00", "0.0", "0.00", "1.0", "001", "00.01", or any other number that can be represented
with less digits.  Also, a decimal point within a number never occurs without at least one digit occuring before
it, so we never started with numbers like ".1".

This is an easy question. You'll just need to review the syntax of <tt>for</tt> comprehension and apply appropriate
filters:

```scala
def ambiguousCoordinates(S: String): List[String] = {
  val str = S.init.tail
  def allZero(s: String): Boolean = s.length > 1 && s.forall(_ == '0')
  def leadingZero(s: String): Boolean = s.length > 1 && s.startsWith("0")
  def numbers(s: String): List[String] =
    for {
      i <- (1 to s.length).toList
      (int, float) = s.splitAt(i)
      if !leadingZero(int) && !float.endsWith("0")
    } yield int + (if (float.isEmpty) "" else "." + float)
  for {
    i <- (1 until str.length).toList
    (part1, part2) = str.splitAt(i)
    if !allZero(part1) && !allZero(part2)
    n1 <- numbers(part1)
    n2 <- numbers(part2)
  } yield s"($n1, $n2)"
}
```

# Depth First Search
## Permutations
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

## Target Sum
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

## Word Pattern II
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

# Breadth First Search

## Bus Routes
[LeetCode 815](https://leetcode.com/problems/bus-routes/description/):
We have a list of bus routes. Each routes[i] is a bus route that the i-th bus repeats forever.
For example if routes[0] = [1, 5, 7], this means that the first bus (0-th indexed) travels in
the sequence 1->5->7->1->5->7->1->... forever.

We start at bus stop S (initially not on a bus), and we want to go to bus stop T. Travelling by
buses only, what is the least number of buses we must take to reach our destination?
Return -1 if it is not possible.

For this question, if we think of individual stop as node, it may not be very obvious what their
relationship should represent. If we instead, use route as node, then we could say that
<tt>node[i][j] = 1</tt> iff we can switch from route i to route j. So our starting states are
those routes that contain our starting node. The ending states are those routes that contain
the destination.

```scala
def numBusesToDestination(routes: Array[Array[Int]], S: Int, T: Int): Int = {
  import scala.collection.immutable.Queue
  val NOT_CONNECTED = -1
  val routesSet = routes.map(_.toSet)
  val routesDist = Array.tabulate(routes.length, routes.length) {
    (r1, r2) =>
      if (r1 != r2 && routesSet(r1).intersect(routesSet(r2)).nonEmpty) 1
      else NOT_CONNECTED
  }
  val startRoutes = routesSet.indices.filter(i => routesSet(i).contains(S))
  val endRoutes = routesSet.indices.filter(i => routesSet(i).contains(T)).toSet
  def minDist(queue: Queue[(Int, Int)], visited: Set[Int]): Int = {
    if (queue.isEmpty) NOT_CONNECTED
    else {
      val ((route, dist), q) = queue.dequeue
      if (endRoutes.contains(route)) dist
      else {
        val neighbors = routesDist(route).indices.filter(
          i => routesDist(route)(i) == 1 && !visited.contains(i))
        minDist(q ++ neighbors.map(n => (n, dist + 1)), visited ++ neighbors)
      }
    }
  }
  if (S == T) 0
  else startRoutes.map(s => minDist(Queue((s, 1)), Set(s))).min
}
```
