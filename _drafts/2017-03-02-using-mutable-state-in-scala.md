---
layout: post
title: Using mutable state in Scala
comments: true
excerpt_separator: <!--more-->
---
Functional programing is a programing paradigm which an be difficult to defined, is it because the language uses [First class function](https://en.wikipedia.org/wiki/First-class_function) like the name suggests ?
Wikipedia present it like this:

> In computer science, functional programming is a programming paradigm—a style of building the structure and elements of computer programs—that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data. It is a declarative programming paradigm, which means programming is done with expressions or declarations instead of statements. In functional code, the output value of a function depends only on the arguments that are input to the function, so calling a function f twice with the same value for an argument x will produce the same result f(x) each time. Eliminating side effects, i.e. changes in state that do not depend on the function inputs, can make it much easier to understand and predict the behavior of a program, which is one of the key motivations for the development of functional programming.

[[Source]](https://en.wikipedia.org/wiki/Functional_programming)

The passage that strikes me is about removing the side effect, enabling easier testing, readability:

> In functional code, the output value of a function depends only on the arguments that are input to the function, so calling a function f twice with the same value for an argument x will produce the same result f(x) each time

But in [Scala], it is possible to use mutable variable like [`ListBuffer`](http://www.scala-lang.org/api/2.12.0/scala/collection/mutable/ListBuffer.html) or even a mutable reference like `var`. How to keep [Scala] a functional language with that feature ?

<!--more-->

My first impression was: "I should not use it", but I came across the `List#takeWhile` function which uses those patterns:

```scala
class List[+A] {
  def head: A
  def tail: List[A]

  def takeWhile(p: A => Boolean): List[A] = {
    val b = new ListBuffer[A]
    var these = this
    while (!these.isEmpty && p(these.head)) {
      b += these.head
      these = these.tail
    }
    b.toList
  }

  def dropWhile(p: A => Boolean): List[A] = {
    @tailrec
    def loop(xs: List[A]): List[A] =
      if (xs.isEmpty || !p(xs.head)) xs
      else loop(xs.tail)

    loop(this)
  }
}
```
[[Source]](https://github.com/scala/scala/blob/419a6394045a0615cb996152b04c92d25f9fb700/src/library/scala/collection/immutable/List.scala#L356-L364)


The `dropWhile` method is nicely crafted from a functional point of view: every variables are immutable, it uses recursion but since the function is tail recursive the compiler is smart enough to convert it to a imperative one. Therefore the `StackOverflowError` can be avoided when working with big lists (> 1024 items).

The `takeWhile` method "doesn't seem really functional" because it is using state but the important part is: the state is kept locally, the function is still pure, there cannot be side effect. A "more functional" implementation could looks like that:

```scala
def takeWhile(f: A => Boolean): List[A]= {
  @tailrec
  def go(list: List[A], acc: List[A]): List[A] = {
    list match {
      case x :: xs if f(x) => go(xs, x :: acc)
      case _ => acc
    }
  }
  go(this, List()).reverse
}
```

The implementation of these two "similar" functions are very different because `List` is design as a linked list. Given its implementation keep the first item or the last item are two different thing. For the `takeWhile` and it is necessary to re-create the list, in this example when calling the `reverse` method. Therefore the [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) and the [memory footprint](https://en.wikipedia.org/wiki/Memory_footprint) for both functions is very different. For a given list of n element, p element must be removed:

|                                    | memory footprint | cyclomatic complexity |
|------------------------------------|------------------|-----------------------|
| dropWhile                          | n                | p                     |
| takeWhile (using recursion)        | 2n + p           | p + n                 |
| takeWhile (using some local state) | n + p            | p                     |

For a very used method like `dropWhile` it is important to optimized it. The question is should I do it in my code ? There is no clear answer to that but I try to follow this methodology

> [premature optimization is the root of all evil](http://wiki.c2.com/?PrematureOptimization)

As said by [Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth), measure the execution time (and memory in our case) your software and optimized when slow.

### A few bad example using mutable state

The mutable state in scala is really powerful

> with great power comes great responsibility

Use that power with care, you can create side effect

```scala
class Mutable {
  def addTwoToAll(list: ListBuffer[Int]): List[Int]= {
    list.map(_ + 2).toList
  }

  def main(args: Array[String]): Unit = {
    val list = ListBuffer(1, 2)
    print(addTwoToAll(list)) // will print List(3, 4)
    print(addTwoToAll(list)) // will print List(5, 6) Ouuups
  }
}
```

Every input and output type should be immutable to avoid that kind of situation.

```scala
class Mutable {

  var numberOfCall = 0

  def called: Int= {
    numberOfCall = numberOfCall + 1
    numberOfCall
  }

  def main(args: Array[String]): Unit = {
    print(called) // will print 1
    print(called) // will print 2 Ouuups
  }
}
```

Don't modify an attribute of an object. Create a new object with the new value for the attribute.



[Scala]: http://www.scala-lang.org/
