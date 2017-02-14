---
layout: post
title: My first impressions with ELM
comments: true
excerpt_separator: <!--more-->
---

I wanted to give the [Elm] language a try because:

* The [Javascript fatigue](http://thefullstack.xyz/javascript-fatigue/) I just can keep up with all framework and library in JS to setup a website, I has became so complex, maybe there is a more stable and easier platform,
* I love functional programming and Javascript is only partly functional, the language is not fully immutable (even if it's better with `const` keyword in EcmaScript6),
* I wanted to feel like a hipster again.


**TL:DR:** My expectations are partly achieved: they are not a lot of libraries but you still need too much tools around your project. The language being close to [Haskell] is definitely functional, there is no state (at least they are "hidden") and the language rely on Message to exchange informations. It feels sometimes a bit of a burden but only because I am a beginner.


*Note: I discovered the [Awesome List of Elm](https://github.com/isRuslan/awesome-elm) with all the needed pointers.*

<!--more-->

First of all I chose the following tutorial [www.elm-tutorial.org](https://www.elm-tutorial.org/en/) and the produced code is available [on GitHub](https://github.com/ThibautGery/elm-tuto). I just copy/paste some code while reading the explanation then I started to implement some functionality by myself.

### Here are a list of my first impressions:

> ###### First of all the compilation and the unit test are lightning fast.

This may be biased since I am currently coding in Scala and its compiler is extremely slow, but it is real pleasure to code.

You don't have the time to slack off on reddit or hacker news. It seems even faster than babel.

> ###### The compiler is really strict

It's the first time I like a compiler, I usually don't compliment compiler, yet:

 * it is strictly typed and the compiler does some type inference (nothing new in functional programing but yet)
 * so far the compiler has been able to warn me everyone of my error. For example when using pattern matching, the compiler assure that you don't have forgot one the type. The following code will produce an error since I have forgotten the `GeneratePlayer` in the pattern matching.

```haskell
type Msg
    = GeneratePlayer
      | ShowPlayers

update : Msg -> String
update message =
    case message of
      ShowPlayers ->
          "the players are: "
```

On the landing page, the documentation state "No Runtime Exceptions" and it looks like like they are not lying.
I feel like I don't really need to test that my component are integrating well with each others. Nonetheless, I do need to test my business logic.

> ###### The community is small but very active

The community is quite small but really active. Yet it means one things: you just can't copy/paste your exception and expect to find the answer on [stackoverflow]. For example, to have a better understanding of the testing framework, I look at the core library tests.
However the number of IDE integration is pretty high.

> ###### Steep learning curve

The learning curve is pretty steep, especially  if you are not familiar with functional programming, Haskell is definitely a plus. At first sight the following expression is not very clear

```haskell
List.foldr (+) 0 [1, 2, 3]
```
The architecture was new in many way for me:
 * The changes are sent via message to a dispatcher
 * The application is organized as components (like [react])

The web application is based on a message architecture which is not trivial to implement.

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

> ###### Html in Code, WTF ?!?

I am still surprised by using HTML method in my code, I still don't have a strong opinion with that feature but I got used to JSX in [react]. You can find a example in the following snippet.

```haskell
btnLevelDecrease : Player -> Html Msg
btnLevelDecrease player =
    a [ class "btn ml1 h1", onClick (ChangeLevel player.id -1) ]
        [ i [ class "fa fa-minus-circle" ] [] ]
```

> ###### Still too much tools needed

This is one of my deception: I was expecting to get rid of npm, bower, webpack, gulp & co but it ain't that simple:

* most of the project use `npm` like [`elm-css`] because the test's runner is a node binary.
* if you want to integrate external CSS stylesheet like [Semantic UI](http://semantic-ui.com/) or even your own you can use [Webpack] and use the Elm loader. I am not sure this is enough to add a new tool (you could use a direct link to a CDN for example)


> ###### Damn it's easy to write unit test

Since all the code is immutable, it's a pleasure to write unit test: no need to mock, no side effect. The tests are easily understanding like the following [snippet from my project](https://github.com/ThibautGery/elm-tuto/blob/master/tests/Players/UpdateTests.elm#L9-L31)

```haskell
layers: List Player
players = [
    { id="1"
    , name= "toto"
    , level = 1
    }]

tests : Test
tests =
    describe "The Player module"
        [ describe "deletePlayer"
            [ test "should remove no player if not in list" <|
                \() ->
                    let
                        actual = deletePlayer "3" players
                    in
                        Expect.equalLists actual players
```

I have found too limitation:
* the tooling is not optimized: there is two `elm-package.json` one for the tests and one the application, the tests need the dependencies of the tested function, so you have to duplicate most of your dependencies.
* I still need to figure how to test the function using pattern command to share a state like the following snippet

```haskell
deleteRequest : Player -> Http.Request PlayerId
deleteRequest player =
    Http.request
        { body = memberEncoded player |> Http.jsonBody
        , expect = Http.expectStringResponse (\_ -> Ok player.id)
        , headers = []
        , method = "DELETE"
        , timeout = Nothing
        , url = "http://localhost:4000/players/" + player.id
        , withCredentials = False
        }


delete : Player -> Cmd Msg
delete player =
    deleteRequest player
        |> Http.send OnDelete
```

> ###### No Whaou effect

It might be for the best since I have always been deceived after that initial feeling because it would only mean that there is some kind of dark magic going on under the hood.

So far, I like to language and the developer experience even if I know I won't be able to code with this framework for my client.


*Disclaimer: I have only played with the language for 3 days.*

[Elm]: http://elm-lang.org/
[Haskell]: https://www.haskell.org/
[stackoverflow]:http://stackoverflow.com/questions/tagged/elm
[`elm-css`]: https://github.com/rtfeldman/elm-css
[Webpack]: https://webpack.github.io/
[react]: https://facebook.github.io/react/
