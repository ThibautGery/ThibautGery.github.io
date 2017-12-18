---
layout: post
title: Filter a string using Javascript
comments: true
excerpt_separator: <!--more-->
---

Filtering a string in Javascript could have been straightforward but unfortunately it is not.

```javascript
"this_is_it".filter(c -> c != '_')
```

This code will throw the following exception

```
Uncaught TypeError: "this_is_it".filter is not a function
    at <anonymous>:1:14
```

<!--more-->


We can transform the string as a char array like this

```javascript
"this_is_it".split('').filter(c => c != '_').join('')
```

But there is a funnier way of doing but you should be aware of the pros and cons of this methods:

 * this is less readable
 * the element of the string are looped onto 2 times instead of 3 times like the previous implementations


*There is a good chance you will never need to do something like this. [Don't optimise prematurely](http://wiki.c2.com/?PrematureOptimization)*

```javascript
[].filter.call("this_is_it", c => c != '_').join('')
```

Let's deconstruct this complicated call:

* we get the [`filter`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) function from an empty array
* we [`call`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/call) the function but we override the `this` with the first argument then we pass the arguments to the function (here the predicated).
* at last we join the the char array together to get back a string


We can apply the `filter` function on the string by looking at its [polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter#Polyfill): the only feature used is `stringVar[index]` which exist for string.

Voilà.
