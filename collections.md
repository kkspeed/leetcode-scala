Collections
===
Scala provides powerful collection APIs to operate on collections. They include
but not limited to: applying a function to every member of the collection, 
filter the collection by some criteria, aggregate over the collection... This
section takes a looks at collections in reasonable detail.

# Prerequisite
[Week 4 and 5 of Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1/home/week/5)
has extensive coverage on lists.

Chapter 5 and 6 of [Scala Succinctly](https://www.syncfusion.com/ebooks/scala_succinctly)
and Chapter 8 presents pattern matching.

# Bulb Switcher
[LeetCode 319](https://leetcode.com/problems/bulb-switcher/description/).

A little bit observation reveals that for a number <tt>n</tt>, only if its number of
divisors is odd will it stay on. Only perfect square numbers satisfy this condition. The code
is:

```scala
def bulbSwitch(n: Int): Int = 
  (1 to n).count(x => Math.sqrt(x).toInt * Math.sqrt(x).toInt == x)
```

<tt>1 to n</tt> in Scala creates an iterator that iterates over <tt>[1..n]</tt>. On the other hand,
<tt>1 until n<tt> iterates over <tt>[1..n - 1]</tt>. <tt>count</tt> takes in a predicate and returns
the number of elements in the collection that satisfy such predicate.

Of course, there is easier solution to this problem as discussed
[here](https://leetcode.com/problems/bulb-switcher/discuss/77104/Math-solution..).

# Output Contest Matches
[LeetCode 544](https://leetcode.com/problems/output-contest-matches/description/)

For this question, we just need to zip the first half of the list with the second half revered. We
repeat this until the length of the list is 1.

```scala
def findContestMatch(n: Int): String = {
  def construct[T](ls: List[T]): String =
    if (ls.length == 1) ls.mkString
    else construct(ls.take(ls.length / 2) zip ls.drop(ls.length / 2).reverse)
  construct((1 to n).toList)
}
```

# Can Place Flowers
[LeetCode 605](https://leetcode.com/problems/can-place-flowers/description/): Given an array of a number
of 0s and 1s. Determine if it could change additional <tt>n</tt> 0s to 1s and maintain that no 2 1s are
adjacent.

Observing the example:

```
3 zeros   10001   --> 10101     --> 1 additional flower
2 zeros   1001    --> 1001      --> 0 additional flower
4 zeros   100001  --> 101001    --> 1 additional flower
5 zeros   1000001 --> 1010101   --> 2 additional flower
```

we see that for consecutive N zeros, we could put maximum <tt>N / 2 - 1</tt> 1s if N is even and <tt>N / 2</tt>
if N is odd.

Thus the algorithm is to find the number of 0s between each pair of 1s and calcuate the maximum number of 1s
that could be inserted. The code is:

```scala
def canPlaceFlowers(flowerbed: Array[Int], n: Int): Boolean =
  (-2 +: flowerbed.indices.filter(x => flowerbed(x) == 1) :+
   flowerbed.length + 1).sliding(2).map { 
    case Seq(s, e) => {
      val num = e - s - 1 
      if (num % 2 == 0) num / 2 - 1 else num / 2
    }
  }.sum >= n
```

The code could be understood in multiple parts:

```scala
flowerbed.indices
```
is alias for <tt>1 until flowerbed.length</tt>. We filter it by <tt>x => flowerbed(x) == 1</tt>, which
in combination finds all the indices of 1s. We then prepend and append <tt>-2</tt> and <tt>flowerbed.length + 1</tt>
as hypothetical 1s to accomodate beginning and ending parts of the flowerbed, which normalizes the way we access
the index pairs.

<tt>Seq.sliding(2)</tt> creates another sequence that groups every 2 elements of the sequence. More examples are:

```scala
List(1, 2, 3, 4, 5).sliding(2)  evaluates to List(List(1, 2), List(2, 3), List(3, 4), List(4, 5))
Array(1, 2, 3, 4, 5).sliding(3) evaluates to Array(Array(1, 2, 3), Array(2, 3, 4), Array(3, 4, 5))
```

Then we use map to examine each of such pairs and compute the maximum number of 1s that can be inserted.

At last we sum the numbers up to get the total number of 1s to be inserted.

# Valid Parentheses 
[LeetCode 20](https://leetcode.com/problems/valid-parentheses/description/): Given a string containing '()[]{}',
determine if the input is valid.

We use a stack to maintain the opening parentheses. When we see a closing one, we check the top of stack. If it's
not a matching open parentheses, return false. Otherwise, if we have anything remaining on the stack when we've done
scanning the string, we also return false. The code:

```scala
def isValid(s: String): Boolean = {
  def valid(s: List[Char], stack: List[Char]): Boolean =
    (s, stack) match {
      case (Nil, Nil) => true
      case (x::xs, _) if "[{(".contains(x) => valid(xs, x::stack)
      case (x::xs, r::rs) if "]})".indexOf(x) == "[{(".indexOf(r) => valid(xs, rs)
      case _ => false
    }
  valid(s.toList, Nil)
}
```

# Swap Adjacent in LR String
[LeetCode 777](https://leetcode.com/problems/swap-adjacent-in-lr-string/description/)
In a string composed of 'L', 'R', and 'X' characters, like "RXXLRXRXL", a move consists of either replacing one
occurrence of "XL" with "LX", or replacing one occurrence of "RX" with "XR". Given the starting string start and
the ending string end, return True if and only if there exists a sequence of moves to transform one string to
the other.

The problem could solved based on 3 conditions:

1. Both strings have same length
2. Both strings have same number of 'L's and 'R's
3. All 'L's indices in <tt>end</tt> are smaller or equal to their indices in <tt>start</tt> and
   all 'R's indices in <tt>end</tt> are greater or equal to their indices in <tt>start</tt>

```scala
def canTransform(start: String, end: String): Boolean = {
  val sv = start.zipWithIndex.filterNot(_._1 == 'X')
  val ev = end.zipWithIndex.filterNot(_._1 == 'X')
  start.length == end.length && sv.length == ev.length &&
    (sv, ev).zipped.forall {
      case (('L', i1), ('L', i2)) => i1 >= i2
      case (('R', i1), ('R', i2)) => i1 <= i2
      case _ => false
    }
}
```

# Merge Intervals
[LeetCode 56](https://leetcode.com/problems/merge-intervals/description/): Given a collection of intervals, merge
all overlapping intervals.

We sort the intervals by their start element. Then:
1. Look at the first 2 intervals <tt>x1::x2::xs</tt>. If they overllap, we merge them. We add the merged intervals
   back for further examination
2. Otherwise, interval <tt>x1</tt> is safely ignored as all start elements of other intervals are guaranteed to be no
   less than the start element of <tt>x2</tt>

This naturally produces code:

```scala
def merge(intervals: List[Interval]): List[Interval] = {
  def doMerge(ints: List[Interval]): List[Interval] = {
    ints match {
      case x1::x2::xs if x2.start <= x1.end =>
        doMerge(new Interval(x1.start, x1.end max x2.end)::xs)
      case x::xs => x::doMerge(xs)
      case Nil => Nil
    }
  }
  doMerge(intervals.sortBy(_.start))
}
```

But the code will result in stack overflow due to the depth of recursion. One solution is to change the code to tail call:

```scala
def merge(intervals: List[Interval]): List[Interval] = {
  def doMerge(ints: List[Interval], result: List[Interval]): List[Interval] = {
    ints match {
      case x1::x2::xs if x2.start <= x1.end =>
        doMerge(new Interval(x1.start, x1.end max x2.end)::xs, result)
      case x::xs => doMerge(xs, x::result)
      case Nil => result
    }
  }
  doMerge(intervals.sortBy(_.start), Nil)
}
```

But we could further improve the code using <tt>foldLeft</tt>:

```scala
def merge(intervals: List[Interval]): List[Interval] =
  intervals.sortBy(_.start).foldLeft(List[Interval]()) {
    case (r::rs, i) if i.start <= r.end =>
      new Interval(r.start, i.end max r.end)::rs
    case (rs, i) => i::rs
  }
```

<tt>foldLeft</tt> is used as <tt>foldLeft(default)(binary op)</tt>, which repeatedly apply the operator
<tt>binary op</tt> to a collection:

```
A = List(1, 2, 3, 4, 5)
A.foldLeft(default)(op) =
  ((((default op 1) op 2) op 3) op 4) op 5
```

In our case, <tt>op</tt> is the operation to merge intervals.

# Next Greater Elements II
[LeetCode 503] (https://leetcode.com/problems/next-greater-element-ii/description/)

Given a circular array (the next element of the last element is the first element of the array), print the Next
Greater Number for every element. The Next Greater Number of a number x is the first greater number to its
traversing-order next in the array, which means you could search circularly to find its next greater number.
If it doesn't exist, output -1 for this number. 

The idea is to keep a stack of <tt>(elem, index)</tt> and the stack is kept in decreasing order. Whenever we 
encounter an element <tt>N</tt> that's greater than the stack's top element, we keep pop until the head element
in the stack is no less than current element. For these elements, we update their greater element to <tt>N</tt>.

For Scala, we could use a <tt>Map</tt> to keep the updates and populate it with Array.tabulate. Popping stack
is easily done with <tt>List.span</tt>, which is essentially <tt>(List.takeWhile, List.dropWhile)</tt>.

```scala
def nextGreaterElements(nums: Array[Int]): Array[Int] = {
  val (_, updates) = (nums ++ nums).zipWithIndex.foldLeft(
    List[(Int, Int)](), Map[Int, Int]().withDefaultValue(-1)) {
    case ((xs, ws), n) =>
      val (res, remain) = xs.span(c => c._1 < n._1)
      (n::remain, ws ++ res.map(p => (p._2, n._1)))
  }
  Array.tabulate(nums.length)(updates)
}
```

# Sequence Reconstruction:
[LeetCode 444](https://leetcode.com/problems/sequence-reconstruction/description/)

This quesstion is basically asking whether we can construct a unique topological order of the nodes.
We check 3 conditions. They are inlined in the comments:

```scala
def sequenceReconstruction(org: Array[Int], seqs: List[List[Int]]): Boolean = {
  // Condition 1: org and seqs contain same set of nodes
  seqs.flatten.toSet == org.toSet && {
    // Constructs the edge set. We pick edges by grouping every 2 nodes in the sequence
    val edges = seqs.filter(_.length >= 2).map(l => l zip l.tail).foldLeft(Set[(Int, Int)]())(_ ++ _)
    // We assume org is the topological order.
    val imap = Map(org.zipWithIndex: _*)

    // Condition 2: every pair in org should be an edge. This guarantees unique topological
    //   order: https://en.wikipedia.org/wiki/Topological_sorting#Uniqueness
    // Condition 3: every edge in seq should not be back edge, aka topological order of
    //   starting node should always be smaller than ending node.
    org.zip(org.tail).forall(edges.contains) && edges.forall(e => imap(e._1) < imap(e._2))
  }
}
```

# Employee Free Time
[LeetCode 759](https://leetcode.com/problems/employee-free-time/description/)

You could utilize that "employee's time is sorted" to do an N-way merge sort but
for this particular problem, simply flatten the time and sort them would successfully
pass the OJ.

```scala
def employeeFreeTime(schedule: List[List[Interval]]): List[Interval] =
  schedule.flatten.sortBy(_.start).foldLeft(List[Interval]()) {
    case (x::xs, i) if i.start < x.end => new Interval(x.start, i.end max x.end)::xs
    case (xs, i) => i::xs
  }.reverse.sliding(2).filter(_.length == 2).map {
    case List(i1, i2) => new Interval(i1.end, i2.start)
  }.toList.filterNot(i => i.start == i.end)
```

# The Skyline Problem
[LeetCode 218](https://leetcode.com/problems/the-skyline-problem/description/)

From the problem we observe that the skyline points are those at which the max
height changes. To keep track of the currently highest line segment, we use a 
map to keep a "ref counted value" for this height. Whenever we encounter a
segment starting point, we increase the ref count. Whenever we encounter a
segment ending point, we decrease the ref count. This is a lot like matching
balancing parentheses when there is only '(' and ')', for which we could use
a ref counter instead of a stack. Here for map we use tree map instead of
hash map as we'll need to keep track of the maximum value:

```scala
def getSkyline(buildings: Array[Array[Int]]): List[Array[Int]] = {
  import scala.collection.immutable.TreeMap
  buildings.flatMap(a => Array((a(0), 0, a(2)), (a(1), 1, a(2)))).sorted
    .foldLeft((List[Array[Int]](), TreeMap[Int, Int]().withDefaultValue(0))) {
      case ((result, map), (p, 0, h)) =>
        val newMap = map.updated(h, map(h) + 1)
        (Array(p, newMap.last._1)::result, newMap)
      case ((result, map), (p, _, h)) =>
        val newMap = if (map(h) == 1) map - h else map.updated(h, map(h) - 1)
        (Array(p, newMap.lastOption.map(_._1).getOrElse(0))::result, newMap)
    }._1.foldLeft(List[Array[Int]]()) {
      case (x::xs, r) if r(0) == x(0) => x::xs
      case (x::xs, r) if r(1) == x(1) => r::xs
      case (xs, r) => r::xs
  }
}
```

# Falling Squares
[LeetCode 699](https://leetcode.com/problems/falling-squares/description/)

On an infinite number line (x-axis), we drop given squares in the order they are given.

The i-th square dropped (positions[i] = (left, side\_length)) is a square with the
left-most point being positions[i][0] and sidelength positions[i][1].

The square is dropped with the bottom edge parallel to the number line, and from a
higher height than all currently landed squares. We wait for each square to stick
before dropping the next.

The squares are infinitely sticky on their bottom edge, and will remain fixed to any
positive length surface they touch (either the number line or another square). Squares
dropped adjacent to each other will not stick together prematurely.

Return a list ans of heights. Each height ans[i] represents the current highest height
of any square we have dropped, after dropping squares represented by positions[0],
positions[1], ..., positions[i]. 

The problem's "hard" tag is pretty misleading. Actually we just need to brutal force it.
For every newly dropped square <tt>(x, h)</tt>, find the highest square that intersects
this one (assume the height for that square is <tt>H</tt>, if none is found, <tt>H = 0</tt>)
then add this square into the set of squares with <tt>h = h + H</tt>:

```scala
def fallingSquares(positions: Array[Array[Int]]): List[Int] =
  positions.scanLeft(Seq[(Int, Int, Int)]()) {
    case (intervals, Array(x, h)) =>
      intervals.filter(i => !(i._2 <= x || x + h <= i._1)) match {
        case Seq() => (x, x + h, h) +: intervals
        case ints => (x, x + h, h + ints.maxBy(_._3)._3) +: intervals
      }
  }.tail.map(_.maxBy(_._3)._3).toList
```

# Flip Game
[LeetCode 822](https://leetcode.com/problems/card-flipping-game/description/)

On a table are N cards, with a positive integer printed on the front and back of each card (possibly different).

We flip any number of cards, and after we choose one card. 

If the number X on the back of the chosen card is not on the front of any card, then this number X is good.

What is the smallest number that is good?  If no number is good, output 0.

Here, fronts[i] and backs[i] represent the number on the front and back of card i. 

A flip swaps the front and back numbers, so the value on the front is now on the back and vice versa.

```
Example:
Input: fronts = [1,2,4,4,7], backs = [1,3,4,1,3]
Output: 2
Explanation: If we flip the second card, the fronts are [1,3,4,4,7] and the backs are [1,2,4,1,3].
We choose the second card, which has number 2 on the back, and it isn't on the front of any card,
so 2 is good.
```

This is actually a brain teaser. The idea is to find the smallest number that does not appear on both
sides. In such case, all other cards could be turned so that the number is not on the front side. So the
solution is just a 2 liner:

```scala
def flipgame(fronts: Array[Int], backs: Array[Int]): Int = {
  val both = fronts.zip(backs).filter(x => x._1 == x._2).map(_._1).toSet
  (fronts ++ backs).sorted.find(x => !both.contains(x)).getOrElse(0)    
}
```

# Meeting Rooms II
[LeetCode 253](https://leetcode.com/problems/meeting-rooms-ii/description/)

We sort the intervals by starting point. Then the minimum number of meeting rooms is just the 
maximum value of overlapping levels we ever encountered.

```
         |---------------|
     |-----------|
  |----------|
   1 | 2 | 3 | 2 | 1
```

The maximum levels could be determined by a counter. Whenever we enter a <tt>start</tt> of
an interval, we increment the counter an we decrement it when we exit. The idea is similar
to matching parentheses:

```scala
def minMeetingRooms(intervals: Array[Interval]): Int =
  intervals.flatMap(i => Seq((i.start, 1), (i.end, -1)))
     .sorted.scanLeft(0)(_ + _._2).max
```
