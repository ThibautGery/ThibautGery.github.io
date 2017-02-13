---
layout: post
title: My first impression with ELM
comments: true
excerpt_separator: <!--more-->
---

I wanted to give the [Elm] language a try because:

* The [Javascript fatigue](http://thefullstack.xyz/javascript-fatigue/) I just can keep up with all framework and library in JS to setup a website, I has became so complex, maybe there is a more stable and easier platform,
* I love functional programming and Javascript is only partly functional, the language is not fully immutable (even if it's better with `const` keyword in EcmaScript6),
* I wanted to feel like a hipster again.


**TL:DR:** My expectations are partly achieved: they are not a lot of libraries but you still need too much tools around your project. The language being close to [Haskell] is definitely functional, there is no state (at least they are "hidden") and the language rely on Message to exchange informations. It feels sometimes a bit of a burden but only because I am a beginner.


*Note: I discovered the [Awesome List of Elm](https://github.com/isRuslan/awesome-elm) with all the needed pointer*

<!--more-->

First of all I chose the following tutorial [www.elm-tutorial.org](https://www.elm-tutorial.org/en/) and the produced code is available [on GitHub](https://github.com/ThibautGery/elm-tuto). I just copy/paste some code while reading the explanation then I started to implement some functionality by myself.

### Here are a list of my first impressions:

> ###### First of all the compilation and the unit test are lightning fast.

This may be biased since I am currently coding in Scala and its compiler is extremely slow, but it is real pleasure to code. You don't have the time to slack off on reddit or hacker news.

> ###### The compiler is really strict

It's the first time I like a compiler, I usually don't compliment compiler, yet:

 * it is strictly typed and the compiler does some type inference (nothing new in functional programing but yet)
 * so far the compiler has been able to warn me everyone of my error. For example when using pattern matching, the compiler assure that you don't have forgot one the type.
On the landing page, the documentation state "No Runtime Exceptions" and it looks like like they are not lying.
I feel like I don't really need to test that my component are integrating well with each others. I do need to test my business logic.

> ###### The community is small but very active

The community is quite small but really active. Yet it means one things: you just can't copy/paste your exception and expect to find the answer on [stackoverflow]. For example, to have a better understanding of the testing framework, I took a look at the core library tests.
One thing that surprised me is the number of IDE available integrations.

> ###### Steep learning curve

The learning curve is pretty steep, especially  if you are not familiar with functional programming, Haskell are definitely a plus. The web application is based on a message architecture which is not trivial to implement.

> ###### Weird indentation

The code formatting standard  from creating the object feels weird

```haskell
new : Player
new =
    { id = "0"
    , name = ""
    , level = 1
    }
```

After a few days it actually make sense:

* if you want to add a line, don't add the comma on the previous line,
* if you are commenting the last line don't bother with the comma.

> ###### Still too much tools needed

This is one of my deception: I was expecting to get rid of npm, bower, webpack, gulp & co but it ain't that simple:

* most of the project use `npm` like this one [`elm-css`] because the test's runner is a node binary.
* if you want to integrate external CSS stylesheet like [Semantic UI](http://semantic-ui.com/) or even your own you can use [Webpack] and use the Elm loader. I am not sure this is enough to add a new tool (you could use a direct link to a CDN)


> **TODO** Unit test

**TODO**


> ###### No Whaou effect

It might be for the best since I have always been deceived after that initial effect because it would only mean that is some kind of dark magic going on under the hood. I feel like the language is not too high level.


*Disclaimer: I have only played with the language for 3 days.*

[Elm]: http://elm-lang.org/
[Haskell]: https://www.haskell.org/
[stackoverflow]:http://stackoverflow.com/questions/tagged/elm
[`elm-css`]: https://github.com/rtfeldman/elm-css
[Webpack]: https://webpack.github.io/
