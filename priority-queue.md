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

# Remark
You don't have to re-invent the wheel in the real world. Scala has
<tt>scala.collection.mutable.PriorityQueue</tt>.

# References
[Pairing Heap](https://en.wikipedia.org/wiki/Pairing_heap)
[Exercise of Week 3 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/3)
