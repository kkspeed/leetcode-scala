Priority Queue
===
Priority Queue allows us to keep elements ordered by their priority.
They are typically heap-backed. Functional heap is an interesting
topic. Here we present one implementation called pairing heap. It
has good performance: O(1) for findMin, O(log n) for deleteMin,
O(1) for insert, O(1) for merge.

# Heap
## Heap Trait
The heap is defined by the following methods:

```scala
trait Heap {
  type H  // the heap type
  type A  // the element type
  val ord: Ordering[A]

  def empty: H  // empty heap
  def isEmpty(h: H): Boolean

  def insert(x: A, h: H): H
  def meld(h1: H, h2: H): H  // merge 2 heaps

  def findMin(h: H): A
  def deleteMin(h: H): H
}
```

## Pairing Heap 
Here we present the implementation. Please refer to
[Wikipedia Page](https://en.wikipedia.org/wiki/Pairing_heap) for
detailed explanation.

```scala
trait PairHeap extends Heap {
  // A pairing heap contains a smallest element and a list of
  // subheaps.
  case class PairingTree(elem: A, subheaps: List[PairingTree])

  abstract class PairingHeap
  case class Tree(tree: PairingTree) extends PairingHeap
  case object Empty extends PairingHeap

  override type H = PairingHeap

  override def empty: H = Empty
  override def isEmpty(h: H): Boolean =
    h match {
      case Empty => true
      case _ => false
    }

  override def insert(x: A, h: H): H =
    meld(Tree(PairingTree(x, Nil)), h)
  // meld merges two PairingHeap by taking the minimum of the
  // two as the minimum element and merges list of trees.
  override def meld(h1: H, h2: H): H =
    (h1, h2) match {
      case (Empty, p2) => p2
      case (p1, Empty) => p1
      case (Tree(t1), Tree(t2)) =>
        if (ord.lt(t1.elem, t2.elem))
          Tree(PairingTree(t1.elem, t2::t1.subheaps))
        else
          Tree(PairingTree(t2.elem, t1::t2.subheaps))
    }

  def findMin(h: H): A =
    h match {
      case Empty => sys.error("empty heap")
      case Tree(PairingTree(e, _)) => e
    }
  // deleteMin is implemented with a helper mergePairs which
  // merges the list of trees into one heap.
  def deleteMin(h: H): H =
    h match {
      case Empty => sys.error("deleting empty heap")
      case Tree(ts) => mergePairs(ts.subheaps)
    }

  private def mergePairs(trees: List[PairingTree]): PairingHeap =
    trees match {
      case Nil => Empty
      case List(l) => Tree(l)
      case t1::t2::ts => meld(meld(Tree(t1), Tree(t2)), mergePairs(ts))
    }
}
```

## Example: Nth Ugly Number 
[LeetCode 264](https://leetcode.com/problems/ugly-number-ii/description/):
Return the n-th ugly number. Ugly numbers are of form
```
2^a * 3^b * 5^c, where a >= 0, b >= 0, c >= 0
```

We can use a heap to get the currently minimum ugly number <tt>n</tt>,
then we generate <tt>n * 2, n * 3, n * 5</tt> and push them into
the priority queue. Note that duplicates could occur in this process, so
we'll need a <tt>Set</tt> to keep track of occurred elements.


```scala
def nthUglyNumber(n: Int): Int = {
  // Implement PairHeap, extending it with element type Long and ordering
  // trait Ordering[Long].
  object LongHeap extends PairHeap {
    override type A = Long
    override val ord: Ordering[Long] = Ordering[Long]
  }
  def nth(n: Int, queue: LongHeap.H, visited: Set[Long]): Long =
    if (n == 1) LongHeap.findMin(queue)
    else {
      val k = LongHeap.findMin(queue)
      val nexts = List(k * 2, k * 3, k * 5)
        .filter(i => !visited.contains(i))
      nth(n - 1, nexts.foldRight(LongHeap.deleteMin(queue))(LongHeap.insert),
        visited ++ nexts)
    }
  nth(n, LongHeap.insert(1, LongHeap.empty), Set(1)).toInt
}
```
# IPO
[LeetCode 502](https://leetcode.com/problems/ipo/description/)

We iterate <tt>k</tt> times. For each time, we put every project that could be done under 
current budge into the heap. And we pick the one that maixmize our profit. We continue this
process until we either have no projects within budget to do or we've done with <tt>k</tt>
times. The code is:

```scala
def findMaximizedCapital(k: Int, W: Int, Profits: Array[Int], Capital: Array[Int]): Int = {
  val queue = scala.collection.mutable.PriorityQueue()(Ordering[(Int, Int)])
  def iter(k: Int, w: Int, projects: Array[(Int, Int)]): Int =
    if (k == 0) w
    else {
      val (canDo, rest) = projects.span(_._2 <= w)
      queue.enqueue(canDo: _*)
      if (queue.isEmpty) w
      else {
        val profit = queue.dequeue
        iter(k - 1, w + profit._1, rest)
      }
    }
  iter(k, W, Profits.zip(Capital).sortBy(_._2))
}
```

Sorting needs <tt>O(nlogn)</tt>. We at most will put <tt>N</tt> elements into the heap, which 
could be <tt>O(nlogn)</tt> if we are using a binary heap. Then we perform <tt>deleteMin</tt>
<tt>k</tt> times, which is <tt>O(klogn)</tt> so the total runtime is <tt>O(nlogn)</tt> assume
<tt>k < n</tt>.

We could use our functional PairingHeap as well. Since PairingHeap's insertion time is
<tt>O(1)</tt>, it actually performs better than the mutable PriorityQueue based implementation.

```scala
def findMaximizedCapital(k: Int, W: Int, Profits: Array[Int], Capital: Array[Int]): Int = {
  object ProfitHeap extends PairHeap {
    override type A = (Int, Int)
    override val ord: Ordering[A] = Ordering[A].reverse
  }
  def iter(k: Int, w: Int, projects: Array[(Int, Int)], heap: ProfitHeap.H): Int =
    if (k == 0) w
    else {
      val (canDo, rest) = projects.span(_._2 <= w)
      val choices = canDo.foldLeft(heap)((h, i) => ProfitHeap.insert(i, h))
      if (ProfitHeap.isEmpty(choices)) w
      else iter(k - 1, w + ProfitHeap.findMin(choices)._1, rest,
        ProfitHeap.deleteMin(choices))
    }
  iter(k, W, Profits.zip(Capital).sortBy(_._2), ProfitHeap.empty)
}
```

For a shorter code, we could augment our heap and make it more forgiving when handling empty heap.
Then the code could be more concise:

```scala
def findMaximizedCapital(k: Int, W: Int, Profits: Array[Int], Capital: Array[Int]): Int = {
  object ProfitHeap extends PairHeap {
    override type A = (Int, Int)
    override val ord: Ordering[A] = Ordering[A].reverse
    override def findMin(h: H): A = if (isEmpty(h)) (0, 0) else super.findMin(h)
    override def deleteMin(h: H): H = if (isEmpty(h)) h else super.deleteMin(h)
  }
  (1 to k).foldLeft((W, Profits.zip(Capital).sortBy(_._2), ProfitHeap.empty)) {
    case ((w, future, heap), _) =>
      val (canDo, rest) = future.span(_._2 <= w)
      val choices = canDo.foldLeft(heap)((h, i) => ProfitHeap.insert(i, h))
      (w + ProfitHeap.findMin(choices)._1, rest, ProfitHeap.deleteMin(choices))
  }._1
}
```

Note that in this implementation, we do not short circuit if we find nothing to do, which might
incur additional overhead under certain input. Though LeetCode OJ's data do not seem to have
this type of test case.

## Split Array into Consecutive Subsequences
[LeetCode 659](https://leetcode.com/problems/split-array-into-consecutive-subsequences/description/)

You are given an integer array sorted in ascending order (may contain duplicates), you need to split
them into several subsequences, where each subsequences consist of at least 3 consecutive integers.
Return whether you can make such a split.

```
Example 1:
Input: [1,2,3,3,4,5]
Output: True
Explanation:
You can split them into two consecutive subsequences : 
1, 2, 3
3, 4, 5

Example 2:
Input: [1,2,3,3,4,4,5,5]
Output: True
Explanation:
You can split them into two consecutive subsequences : 
1, 2, 3, 4, 5
3, 4, 5

Example 3:
Input: [1,2,3,4,4,5]
Output: False
```

We scan the array from left to right. For each number encountered, we append it to the
shortest subgroup that expects this number (adding this number to the group would maintain
the property that the group of numbers are consecutive). Thus, naturally we'll need a map
from Int <tt>N</tt> to Heap <tt>H</tt> that maintains the groups of numbers that expects 
number <tt>N</tt>. We can further simplify that by just putting length of such groups on
the heap.

```scala
def isPossible(nums: Array[Int]): Boolean = {
  object NumHeap extends PairHeap {
    override type A = Int
    override val ord: Ordering[A] = Ordering[Int]
    // We augment the heap to prevent errors on deleteMin and findMin
    override def findMin(h: H): A =
      if (isEmpty(h)) 0 else super.findMin(h)
    override def deleteMin(h: H): H =
      if (isEmpty(h)) h else super.deleteMin(h)
  }
  // Add an implicit class for Pimp My Library pattern that extends Map
  // with a function updateF(k, f) that updates the map by applying function f
  // at key k.
  implicit class UpdateMap[A, B](map: Map[A, B]) {
    def updateF(key: A, f: B => B): Map[A, B] = map.updated(key, f(map(key)))
  }
  nums.foldLeft[Map[Int, NumHeap.H]](Map().withDefaultValue(NumHeap.empty)) {
    (map, n) =>
      map.updateF(n + 1, h => NumHeap.insert(NumHeap.findMin(map(n)) + 1, h))
      .updateF(n, NumHeap.deleteMin)
  }.values.filterNot(NumHeap.isEmpty).forall(x => NumHeap.findMin(x) >= 3)
}
```

## Course Schedule III
[LeetCode 630](https://leetcode.com/problems/course-schedule-iii/description/)

There are n different online courses numbered from 1 to n. Each course has some duration(course length)
t and closed on dth day. A course should be taken continuously for t days and must be finished before or
on the dth day. You will start at the 1st day.

Given n online courses represented by pairs (t,d), your task is to find the maximal number of courses that
can be taken.

```
Example:
Input: [[100, 200], [200, 1300], [1000, 1250], [2000, 3200]]
Output: 3
Explanation: 
There're totally 4 courses, but you can take 3 courses at most:
First, take the 1st course, it costs 100 days so you will finish it on the 100th day, and ready to take the next course on the 101st day.
Second, take the 3rd course, it costs 1000 days so you will finish it on the 1100th day, and ready to take the next course on the 1101st day. 
Third, take the 2nd course, it costs 200 days so you will finish it on the 1300th day. 
The 4th course cannot be taken now, since you will finish it on the 3300th day, which exceeds the closed date.

Note:

    The integer 1 <= d, t, n <= 10,000.
    You can't take two courses simultaneously.
```

We order the courses by their deadline.

Consider we already have a set of courses -- if we need to add another one, there are 2 cases:

1. We can add this course since the length of previous courses + current courses' length \< current course's deadline
2. The above condition could not be met. We compare the longest course of previous courses. If the length of that
   course is longer than the length of current course, we swap that course with current course. Thus, we should use
   a max heap to maintain the set of courses.

In this implementation, we use the built-in PriorityQueue.

```scala
def scheduleCourse(courses: Array[Array[Int]]): Int = {
  val ord: Ordering[Array[Int]] = Ordering.by(_(0))
  val pq = scala.collection.mutable.PriorityQueue[Array[Int]]()(ord)
  courses.filterNot(a => a(1) < a(0)).sortBy(_(1)).foldLeft((0, 0, pq)) {
    case ((count, sum, queue), course) =>
      if (sum + course(0) <= course(1)) {
        queue.enqueue(course)
        (count + 1, sum + course(0), queue)
      } else {
        val largest = queue.dequeue
        val smaller = ord.min(largest, course)
        queue.enqueue(smaller)
        (count, sum - largest(0) + smaller(0), queue)
      }
  }._1
}
```

## Most Profit Assigning Work
[LeetCode 826](https://leetcode.com/problems/most-profit-assigning-work/description/)

We have jobs: difficulty[i] is the difficulty of the ith job, and profit[i] is the profit
of the ith job. 

Now we have some workers. worker[i] is the ability of the ith worker, which means that this worker can only complete
a job with difficulty at most worker[i]. 

Every worker can be assigned at most one job, but one job can be completed multiple times.

For example, if 3 people attempt the same job that pays $1, then the total profit will be $3.  If a worker cannot
complete any job, his profit is $0.

What is the most profit we can make?

The greedy strategy is to give every worker the job that's within their capability and makes maximum profit. We need
to sort the workers ability and use priority queue to track maximum profit of the jobs that's within their capability.

```scala
def maxProfitAssignment(difficulty: Array[Int], profit: Array[Int], worker: Array[Int]): Int =
  worker.sorted.foldLeft((difficulty.zip(profit).sorted, scala.collection.mutable.PriorityQueue[Int](), 0)) {
    case ((works, candidates, sum), w) =>
      val (canDo, rest) = works.span(_._1 <= w)
      candidates.enqueue(canDo.map(_._2): _*)
      (rest, candidates, sum + (if (candidates.nonEmpty) candidates.head else 0))
  }._3
```

But taking a look again, we only need the maximum profit. So we could get rid of the priority queue as well.

```scala
def maxProfitAssignment(difficulty: Array[Int], profit: Array[Int], worker: Array[Int]): Int =
  worker.sorted.foldLeft((difficulty.zip(profit).sorted, 0, 0)) {
    case ((works, max, sum), w) =>
      val (canDo, rest) = works.span(_._1 <= w)
      val m = max max (0 +: canDo.map(_._2)).max
      (rest, m, sum + m)
  }._3
```

# Remark
You don't have to re-invent the wheel in the real world. Scala has
<tt>scala.collection.mutable.PriorityQueue</tt>.

# References
[Pairing Heap](https://en.wikipedia.org/wiki/Pairing_heap)
[Exercise of Week 3 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/3)
