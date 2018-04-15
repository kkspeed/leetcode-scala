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


# Can Place Flowers
[LeetCode 605](https://leetcode.com/problems/can-place-flowers/description/): Given an array of a number
of 0s and 1s. Determine if it could change additional <tt>n</tt> 0s to 1s and maintain that no 2 1s are
adjacent.

Observing the example:

```
10001   --> 10101
1001    --> 1001
100001  --> 101001
1000001 --> 1010101
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


# Merge Intervals

# Valid Parentheses 
