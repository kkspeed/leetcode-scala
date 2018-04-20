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
