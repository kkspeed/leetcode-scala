Basic Recursion
===

Some problems in LeetCode do not require any knowledge on 
collections (Arrays, Lists etc). Thus they are the perfect
targets to be presented first.

# Prerequisite
Make sure you are comfortable with basic Scala syntax:
conditionals and function definitions. 
[Week 1 of Functional Programming Principles in Scala](https://www.coursera.org/learn/progfun1/home/week/1)
is the perfect source to the accustomed to those concepts.

# Square Root
[LeetCode 69](https://leetcode.com/problems/sqrtx/description/) requires you to
compute the square root of a non-negative integer x.

The solution here is to use Newton's method. The code is mentioned in
[Lecture 5, Week 1](https://www.coursera.org/learn/progfun1/lecture/FQDE1/lecture-1-5-example-square-roots-with-newtons-method).

The basic idea is that we start with a guess, say 0.1. We repeatedly calculate
<tt>(x / guess + guess) / 2</tt> until guess is close enough to <tt>sqrt(x)</tt>.

The sketch of the algorithm is listed below:

    if guess is good enough: return guess
    else guess = (guess + x / guess) / 2, continue

which converts to following code:


```scala
def mySqrt(x: Int): Int = {
  def goodEnough(guess: Double, x: Double): Boolean =
    Math.abs(guess * guess - x) < 0.001
  def improve(guess: Double, x: Double): Double =
    (guess + x / guess) * 0.5
  def sqrtIter(guess: Double, x: Double): Double =
    if (goodEnough(guess, x)) guess
    else sqrtIter(improve(guess, x), x)
  sqrtIter(0.001, x).toInt
}
```

we use <tt> |guess * guess - x| < 0.001</tt> as the criteria for "good enough".
For corner case, if <tt>x = 0</tt>, <tt>0.001 * 0.001 - 0 < 0.001</tt> so it will
return correctly.

# Ugly Number
[LeetCode 263](https://leetcode.com/problems/ugly-number/description/): A positive number 
is considered ugly whose prime factors are only 2, 3 and 5. 1 is considered to be
ugly number.

From the definition, we have the following 2 conditions:

1. if <tt>n <= 0</tt>, <tt>n</tt> is not ugly
2. if <tt>n == 1</tt>, <tt>n</tt> is ugly

Then for any <tt>n > 1</tt>, we know that it must be divisible by at least one of
2, 3 or 5. And the quotient must still be ugly as it cannot contain any other prime
factors. So we have the following code:

```scala
def isUgly(num: Int): Boolean = {
  if (num <= 0) false
  else if (num == 1) true
  else if (num % 2 == 0) isUgly(num / 2)
  else if (num % 3 == 0) isUgly(num / 3)
  else if (num % 5 == 0) isUgly(num / 5)
  else false
}
```

The code could be further simplified to:

```scala
def isUgly(num: Int): Boolean =
  num > 0 && (num == 1 || List(2, 3, 5).exists(x => num % x == 0 && isUgly(num / x)))
```

with collections.

# Same Tree
[LeetCode 100](https://leetcode.com/problems/same-tree/description/): Given two binary trees, 
check if they are the same.

Two binary trees are the same if:

1. both roots are null
2. the roots have same value and both left subtrees, right subtrees are the same.

Thus, the code is naturally recursive:

```scala
def isSameTree(p: TreeNode, q: TreeNode): Boolean =
  if (p == null && q == null) true
  else if (p == null || q == null) false
  else p.value == q.value && isSameTree(p.left, q.left) && isSameTree(p.right, q.right)
```

# Symmetric Tree
[LeetCode 101](https://leetcode.com/problems/symmetric-tree/description/): Given a binary tree,
check if it's symmetric.

Let's take a look at an example:

```
      2
   /      \
  1        1
 / \      / \
3   5    5   3

```

We see that a tree is symmetric if its root is null or is left subtree and right subtree are
mirrored. Futhermore, two trees are mirrored if:

1. Both roots are null, or
2. The roots' values are the same and the left subtree of one tree and right subtree of the other
   are mirrored.

The code thus is:

```scala
def isMirrored(tree1: TreeNode, tree2: TreeNode): Boolean =
  if (tree1 == null && tree2 == null) true
  else if (tree1 == null || tree2 == null) false
  else tree1.value == tree2.value && isMirrored(tree1.left, tree2.right) && isMirrored(tree1.right, tree2.left)

def isSymmetric(root: TreeNode): Boolean =
  if (root == null) true
  else isMirrored(root.left, root.right)
```

# Tail Recursion
A problem with recursion is that it might use up all the stack frames, causing stack overflow.
An example is:

```scala
// Calculating the sum of 1..n
def poorSum(n: Int): Int =
  if (n == 0) 0
  else n + poorSum(n - 1)
```

In each call, it will push a frame to stack so that the result could be used to compute
the next step (<tt>n + result of poorSum(n - 1)</tt>). Unfortunately, the size of stack
is pretty limited for JVM (most systems). TODO: add a figure.

It's possible to re-use current stack frame for future computation as long as the compiler
knows that the function will not need to "return to current context". Such function
is called tail-recursive function:

```scala
def betterSum(n: Int, sum: Int): Int =
  if (n == 0) sum
  else betterSum(n - 1, n + sum)
```

Here, betterSum is recursively called as the last computation, which means no further
calculation is needed.

In general, we should aim to use tail recursion if the stack is expected to grow large.
But as you'll see later, we'll often use existing functions to avoid explicit recursions.
