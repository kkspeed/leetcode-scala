Parsing
===
One type of problems on LeetCode is related to string parsing. Often you'll
need to write a mini-parser based on some state machine of the very specific
problem setting. Here we take a different approach by introducing a parser
framework based on Monad and present how it could be utilized to solve various
problems.

# Prerequisite
[Week 1 of Functional Program Design in Scala](https://www.coursera.org/learn/progfun2/home/week/1)
has an introduction for monads on Scala.

# Monads
Monads, at very rough level, can be understood as a data structure <tt>Monad[T]</tt> that
has the following two operations defined:

```
trait Monad[T] {
  def flatMap[A](T => Monad[A]): Monad[A]
}
def unit[T](): Monad[T]
```

<tt>unit</tt> basically injects a value into the monadic environment. <tt>flatMap</tt>
applies an operation to the monad and returns the same monad over a different type.

# Monad Parser
Here we present the parser.

```scala
// A parser is a wrapper over a function. The function takes in a list of characters and
// returns an either type:
// 
// - On parsing success, the type is a pair of parsed result and the remaining characters.
// - On parsing failure, it returns an error message:
case class Parser[+T](s: List[Char] => Either[(T, List[Char]), String]) {
  // the parse function runs the parser over an input
  def parse(stream: List[Char]): Either[(T, List[Char]), String] = s(stream)

  // the flatMap functions chains two parser. It will fail if either one fails. or will 
  // apply the result on the first parser.
  def flatMap[B](f: T => Parser[B]): Parser[B] = {
    Parser(
      x => parse(x) match {
        case Left(t) => f(t._1).parse(t._2)
        case Right(err) => Right(err)
      }
    )
  }

  // The map function converts a parser from one type to another by manipulating its value.
  def map[B](f: T => B): Parser[B] = flatMap(x => unit(f(x)))

  // The or_else combinator will use an alternate parser if the first fails.
  def or_else[B >: T](p: Parser[B]): Parser[B] = Parser {
    x =>
      parse(x) match {
        case Left(t) => Left(t)
        case Right(_) => p.parse(x)
      }
  }
}

// The unit type is trivial. It returns a parser that does not consume the stream and just
// returns the supplied value.
def unit[A](x: A): Parser[A] = Parser(s => Left(x, s))

// The fail parser will raise an error on any input.
def fail[A](s: String): Parser[A] = Parser(_ => Right(s))

// chr parser checks if the first char of the stream is the expected char.
def chr(c: Char): Parser[Char] = Parser {
  case (x :: xs) => if (x == c) Left((x, xs)) else Right("Expecting " + c)
  case Nil => Right("End of stream reached")
}

// oneOf parser tries a list of parsers and returns if the first one suceeded. It's defined
// in terms a fold with or_else.
def oneOf[T](ps: List[Parser[T]]): Parser[T] = {
  ps.foldLeft(fail[T]("empty string")) { (p, c) => p.or_else(c) }
}

// The digit parser identifies digits. Note that I've added the minus sign here. It's not
// perfect for handling numbers because it will accept things like 1-23 but for LeetCode
// this totally works (most parsing question assumes a well formed input).
val digit = oneOf("-1234567890".toList.map(chr))

// many1 parser returns a list of results by repeatedly applying the parser. It requires the
// parser to succeed at least once.
def many1[T](p: Parser[T]): Parser[List[T]] = {
  for {
    r <- p
    rs <- many1(p).or_else(unit(Nil))
  } yield r :: rs
}

// sepBy parses a delimited stream, separated by character c. This could parse things like:
// 1,2,3,4,5
def sepBy[T](c: Char, p: Parser[T]): Parser[List[T]] = {
  for {
    t <- p
    rs <- (for {
      _ <- chr(c)
      d <- sepBy(c, p)
    } yield d).or_else(unit(Nil))
  } yield t :: rs
}

// delimited represents a parser that's enclosed by a pair of characters. This could parse
// things like: [123]
def delimited[T](open: Char, close: Char, p: Parser[T]): Parser[T] =
  for {
    _ <- chr(open)
    r <- p
    _ <- chr(close)
  } yield r
```

Now we are ready to solve those parsing questions.

# Construct Binary Tree from String
[LeetCode 536](https://leetcode.com/problems/construct-binary-tree-from-string/description/)

For this question, we construct the following parser:

```scala
def str2tree(s: String): TreeNode = {
  def treeNode: Parser[TreeNode] = (for {
    num <- many1(digit).map(_.mkString.toInt)
    left <- delimited('(', ')', treeNode).or_else(unit(null))
    right <- delimited('(', ')', treeNode).or_else(unit(null))
  } yield {
    val node = new TreeNode(num)
    node.left = left
    node.right = right
    node
  }).or_else(unit(null))

  treeNode.parse(s.toList).left.get._1
}
```

This parser says a tree is a number with two delimited nodes. If any of the nodes fail to parse,
it's assumed to be null (given the input is always valid, this perfectly works).


# Decode String
[LeetCode 394](https://leetcode.com/problems/decode-string/description/)
Given an encoded string, return it's decoded string.

The encoding rule is: <tt>k[encoded\_string]</tt>, where the <tt>encoded\_string</tt> inside the
square brackets is being repeated exactly k times. Note that k is guaranteed to be a positive integer.

You may assume that the input string is always valid; No extra white spaces, square brackets are
well-formed, etc.

Furthermore, you may assume that the original data does not contain any digits and that digits are
only for those repeat numbers, k. For example, there won't be input like <tt>3a</tt> or <tt>2[4]</tt>.

For this question, we'll need yet another primitive parser: <tt>chars</tt> which basically accepts
all lower-case and upper-case characters:

```scala
// I just went on typing all the lower/upper case chars. You can in fact build another primitive
// parser: satisfy[T](p: Parser[T], predicate: T => Boolean): Parser[T] and use it as
// satisfy(chr, _.isLetter). I'll leave implementing satisfy as an exercise.
val chars = oneOf("ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz".toList.map(chr))
```

The integer and string parsers can be constructed directly from digits and chars.
```scala
val integer = many1(digit).map(_.mkString.toInt)
val string = many1(chars).map(_.mkString)
```

Now we define the abstract syntax tree for nested string:

```scala
// The base type for DecodeString syntax tree. It has a toString method, which
// re-prints out the string in deflated format.
trait DecodeString {
  override def toString: String
}
// It could be either a Repeated string like 3[a], which is Repeated(3, List(Single("a")))
case class Repeated(i: Int, s: List[DecodeString]) extends DecodeString {
  override def toString: String = List.fill(i)(s.map(_.toString).mkString).mkString
}
// Or a single string(e.g. abc is represented as Single("abc"))
case class Single(s: String) extends DecodeString {
  override def toString: String = s
}
```

Now we construct the parsers that parses nested string:

```scala
// for Single, we just map construct Single over the string parser
def singleString: Parser[DecodeString] = string.map(Single)

// a string list is a List of a single string or repeated string
def stringList: Parser[List[DecodeString]] = many1(singleString.or_else(repeated))

// parsers repeated string list.
def repeated: Parser[DecodeString] = for {
  i <- integer
  _ <- chr('[')
  ds <- stringList
  _ <- chr(']')
} yield Repeated(i, ds)

```

Finally, the parsers are invoked as:

```scala
if (s.isEmpty) ""
else Repeated(1, stringList.parse(s.toList).left.get._1).toString
```

Note that we could also add the empty string check to the parser itself. Can you do that
as an exercise?

# Parse Lisp Expression 
[LeetCode 736](https://leetcode.com/problems/parse-lisp-expression/description/):
This problem requires you to write an interpreter for a subset of Lisp expressions.
The expressions are:

1. Integer
2. '(' and ')' wrapped forms with fixed types:
  - <tt>(add x y ...)</tt> sums all the numbers 
  - <tt>(mult x y ...)</tt> calculates the product of the numbers
  - <tt>(let x <expr> y <expr> ... <last\_expr>)</tt> the let expression binds variables
    in a couple of expressions and use this new environment to evaluate the last expression.

We can start out by defining the AST:

```scala
// Base lisp expression trait
trait LispExpr {
  // evaluate the expression. It takes in a map of "environment" for variable look up.
  def evalu(env: Map[String, Int]): Int
  def getV: String = sys.error("not able to get var")
}
// Integer atom
case class IntExpr(i: Int) extends LispExpr {
  override def evalu(env: Map[String, Int]): Int = i
}
// Variable atom
case class VarExp(v: String) extends LispExpr {
  override def evalu(env: Map[String, Int]): Int = env(v)
  override def getV: String = v
}
// Sexp is an expression enclosed in '(', ')'
case class Sexp(exprs: List[LispExpr]) extends LispExpr {
  override def evalu(env: Map[String, Int]): Int = {
    exprs.head match {
      case VarExp("add") => exprs.tail.map(_.evalu(env)).sum
      case VarExp("mult") => exprs.tail.map(_.evalu(env)).product
      case VarExp("let") =>
        // Environment variable bindings are updated in let expression.
        // Potentially it could override the existing bindings.
        val newEnv = exprs.tail.init.grouped(2).foldLeft(env) {
          case (e, List(v, expr)) => e + (v.getV -> expr.evalu(e))
        }
        exprs.last.evalu(newEnv)
      case _ => sys.error("unexpected sexp: " + this)
    }
  }
}
```

The parsing part is written as:

```scala
def alphaNumeric: Parser[Char] = oneOf("abcdefghijklmnopqrstuvwxyz0123456789".toList.map(chr))

// Parses atom for integer and variable
def atom: Parser[LispExpr] = for {
  e <- many1(digit).or_else(many1(alphaNumeric))
  s = e.mkString
} yield {
  if (s(0) == '+' || s(0) == '-' || (s(0) >= '0' && s(0) <= '9'))
    IntExpr(s.toInt)
  else
    VarExp(s)
}

// Parse '(' and ')' wrapped sexp.
def sexp: Parser[LispExpr] = atom.or_else(for {
  _ <- chr('(')
  ss <- sepBy(' ', sexp)
  _ <- chr(')')
} yield Sexp(ss))
```

Now we the evaluation could be written as:

```scala
sexp.parse(expression.toList) match {
  case Left((expr, _)) => expr.evalu(Map())
  case Right(s) => sys.error(s)
}
```
