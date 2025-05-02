---
link: https://medium.com/@ghivert/designing-api-in-elm-opaque-types-ce9d5f113033
byline: Guillaume Hivert
site: Medium
date: 2017-04-03T16:04
excerpt: If you already built a package for Elm, you probably saw the magic of the Elm compiler. If not, don’t you ever wonder why all Elm package are so well documented? Every exposed function has its…
twitter: https://twitter.com/@Medium
slurped: 2024-08-17T18:25
title: Designing API in Elm — Opaque types - Guillaume Hivert - Medium
tags:
  - elm
  - api
  - type
---

If you already built a package for Elm, you probably saw the magic of the Elm compiler. If not, don’t you ever wonder why all [Elm package](http://package.elm-lang.org/) are so well documented? Every exposed function has its signature, and most of the time, documentation is really great. Because the Elm compiler _forces you to write_ this documentation_._ When you expose functions in your package, the compiler refuses to compile [if requirements aren’t met](http://package.elm-lang.org/help/documentation-format).

But magic is not over. Elm compiler enforces [semver](http://semver.org/) for you. Change a public signature function, and bump! Major change.

And here’s comes the problem. How changing internals without changing signatures? Impossible, specially if you’re using records. One little change on one type, and you have to increase version number. To solve this problem, Elm has a built-in mechanism: opaque type.

## Using an opaque type

You can declare an opaque type like an union type; without exposing contents. For example, to declare a stack, you could write it with an alias:


```
type alias Stack a =  
  List a
```
Or you could write it with an opaque type:
```
type Stack a =  
  Stack (List a)
```
In the first case, the stack type is open and publicly available. When exposing it, you indicates to the user what is inside ‘Stack a’. In the second case, you’re hiding it in a union type, meaning that all implementation details are left to the developper, not the user. The user can’t access what is inside the Stack. By default, Elm does not expose what is in union types. In other words, ‘Stack a’ is available to the user, but the inner Stack can’t be accessed. It is not possible to write something like:
```
-- stack : Stack a  
case stack of  
  Stack s -> doSomeStuff s
```
If the compiler encounters something like this, it raises an error. How to use this stack if users can’t access data inside? Just provides functions dealing with such a type in your package:


```
empty : Stack a  
empty = Stack []isEmpty : Stack a -> Bool  
isEmpty (Stack s) = List.isEmpty spush : a -> Stack a -> Stack a  
push x (Stack s) **=** Stack (x **::** s)top : Stack a -> Maybe a  
top (Stack s) **=** List.head spop : Stack a -> (Maybe a, Stack a)  
pop (Stack s) **=**   
  ( List.head s  
  , Stack   
    (List.tail s   
      |> Maybe.withDefault [])  
  )
```
By doing this, you can change the implementation if you need. The user never knows about the content of a Stack. Only its signature (i.e. ‘Stack a’), and the functions to use it. You want to add something to the type? No problem, add it inside the opaque type, update public functions if needed, and voilà! No changes in the functions signatures, no changes in the public API. No major change in the version number.

Opaque types are powerful, and allows to design beautiful API. They avoid changes in public API (which is great for backward compatibility) and version number, and [Elm encourages to use opaque types](http://package.elm-lang.org/help/design-guidelines#keep-tags-and-record-constructors-secret). Just remember you can’t match and access opaque types internals outside of the file in which they are defined.