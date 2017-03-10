---
layout: post
title: Writing pure functions in Scala with mutable structures
comments: true
excerpt_separator: <!--more-->
---

Functional programming is a paradigm which can be difficult to define, is it because the language uses [First class function](https://en.wikipedia.org/wiki/First-class_function) like the name suggests? From [Wikipedia's definition](https://en.wikipedia.org/wiki/Functional_programming), I like to focus on the pure functions, mostly because it is the reason I like functional programming:

> In functional code, the output value of a function depends only on the arguments that are input to the function, so calling a function f twice with the same value for an argument x will produce the same result f(x) each time. Eliminating side effects, i.e. changes in state that do not depend on the function inputs, can make it much easier to understand and predict the behavior of a program, which is one of the key motivations for the development of functional programming.

Yet in [Scala], it is possible to use mutable variables like [`ListBuffer`](http://www.scala-lang.org/api/2.12.0/scala/collection/mutable/ListBuffer.html) or even a mutable reference like `var`. How is it possible to write pure functions with mutable structures?

<!--more-->

My first impression was: "I should not use it", but I came across the [`List#takeWhile`](https://github.com/scala/scala/blob/419a6394045a0615cb996152b04c92d25f9fb700/src/library/scala/collection/immutable/List.scala#L356-L364) function which uses those patterns:

```scala
class List[+A] {
  def head: A // Current element of the list
  def tail: List[A] // rest of the list

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


The `dropWhile` method is nicely crafted from a functional point of view: every variable is immutable, there are no side effects.
The recursive call could be a problem: It can fill the stack and send a [`StackOverflowError`](https://docs.oracle.com/javase/8/docs/api/java/lang/StackOverflowError.html), but the compiler is "smart enough" to optimize the [Tail recursion](https://en.wikipedia.org/wiki/Tail_call) into a classic iteration.

The `takeWhile` method "doesn't seem really functional" because it is using state but the important part is: the state is kept locally, the function is still pure, without side effect. A "more functional" implementation could look like that:

```scala
def takeWhile(f: A => Boolean): List[A]= {
  @tailrec
  def go(list: List[A], acc: List[A]): List[A] = {
    list match { // use pattern matching
      case x :: xs if f(x) => go(xs, x :: acc)
      case _ => acc
    }
  }
  go(this, List()).reverse
}
```

The implementation of these two "similar" functions are very different because `List` is designed as a linked list. Given its implementation keep the first item or the last item are two different things. For the `takeWhile` and it is necessary to re-create the list, in this example when calling the `reverse` method.

There are two metrics to take into account:

* [the memory footprint](https://en.wikipedia.org/wiki/Memory_footprint): it is the memory used by the particular function
* [the cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity): it is a quantitative measure of the number of linearly independent paths through a function's source code.

For a given list of n element, p element must be removed:

|                                    | memory footprint | cyclomatic complexity |
|------------------------------------|------------------|-----------------------|
| dropWhile                          | n                | p                     |
| takeWhile (using recursion)        | 2n + p           | 2p                    |
| takeWhile (using local state)      | n + p            | p                     |

In this case, the memory footprint is less relevant since only the references are copied but the execution time deserves to be measured.
I created a small [benchmark](https://gist.github.com/ThibautGery/4783181e00360832c27bcca46c274980) to compare the execution time of the software. I used a list of 100000 integers as input and tried to get the first 1, 50000 and 100000 elements. The simulation was run 10000 times, here are the results:


| elements taken                                     | 1      | 50 000   | 100 000   |
|----------------------------------------------------|--------|----------|-----------|
| takeWhile using recursion (in milliseconds)        | 11     | 5436     | 13694     |
| takeWhile using local state (in milliseconds)      | 11     | 4182     | 8699      |

As expected, the version used in the core library is faster than mine.

For core method like `dropWhile` optimization is important. The question is should I do it in my code? There is no clear answer to that but I try to follow this methodology by [Donald Knuth](https://en.wikipedia.org/wiki/Donald_Knuth)

> [premature optimization is the root of all evil](http://wiki.c2.com/?PrematureOptimization)

I prematurely optimize my code only if those conditions are fulfilled unless I will first measure and act if the product is slow:

* the optimization is straightforward: don't spend too much time on a function that might not even be critical
* the refactoring doesn't hurt the readability: if the code is fast but cannot be maintained, your product is dead
* the performance gain is significant: like reducing `n²` to `n log(n)`

### A few bad example using mutable state

The mutable state in Scala is really powerful

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

Every input and output structures should be immutable to avoid that kind of situation.

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
