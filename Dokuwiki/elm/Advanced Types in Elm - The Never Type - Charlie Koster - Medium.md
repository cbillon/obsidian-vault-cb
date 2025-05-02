---
link: https://medium.com/@ckoster22/advanced-types-in-elm-the-never-type-ca9b3297bbd4
byline: Charlie Koster
site: Medium
date: 2017-09-06T02:28
excerpt: Ok, ok. I’m bending the definition of what “advanced type” means with
  this post, especially since we’re reviewing a type found in the Basics module.
  Regardless, its usage is “advanced” as someone…
twitter: https://twitter.com/@ckoster22
slurped: 2024-08-17T19:38
title: Advanced Types in Elm - The Never Type - Charlie Koster - Medium
---

## Part III: The Never Type

Ok, ok. I’m bending the definition of what “advanced type” means with this post, especially since we’re reviewing a type found in the _Basics_ module. Regardless, its usage is “advanced” as someone coming from a Javascript background since Javascript doesn’t have the concept of a Never type.

## What is the Never type?

The Never type is _a type that doesn’t have any values_. It’s a type that can be specified in a type annotation, but you can’t construct a Never value because it is valueless.

If you are coming from a language like Javascript this is a bit strange. It’s a type that doesn’t have any values? Let’s reiterate a point made by the [docs](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Basics#Never).

- A Bool has two values: `True` and `False`
- The unit type has one value: `()`
- The Never type has no values

How can this be? This starts to make more sense if we look at how Never is defined in the [source](https://github.com/elm-lang/core/blob/5.1.1/src/Basics.elm#L628).

Let’s try and figure out how to use Never as a value. We’d need to use `JustOneMore` and pass it some additional information.

Ok we need to fill in the blank. JustOneMore expects a Never. Easy.

Hmm. Never is _recursively defined_. This might take a while..

Hopefully it’s a little more clear that because Never is recursively defined it is impossible to use it as a value. Additionally, Never is an [opaque type](https://github.com/elm-lang/core/blob/5.1.1/src/Basics.elm#L12) with no constructor functions exposed. It couldn’t be constructed outside of the Basics module even if it wasn’t recursively defined! There’s more info on opaque types in a [previous post](https://medium.com/@ckoster22/advanced-types-in-elm-opaque-types-ec5ec3b84ed2).

## Wait a sec.. I’m still confused

Yea, I was too when I first started doing research for this post. **How can there be an annotated type without a corresponding value?**

The answer is simple. **Not all types in a type annotation require a corresponding value in the function implementation.**

The code above demonstrates that we see this sort of thing every day. In that example **alwaysNothing** is annotated as returning a `Maybe String`, yet the body of the function doesn’t contain a string value. _There’s an annotated type without a corresponding value._

Likewise, I could also update the annotation to return a `Maybe Never`.

This function compiles just fine. The really neat part is if I change `Nothing` to `Just "any type here"` then I get a compiler error because **Never** doesn’t have any values. It forces this function to actually always return `Nothing` by leveraging the type system!

> Note that the docs for Never assert that “generally speaking, you do not want Never in your return types” so take this example with a grain of salt.

It seems obvious in hindsight, but honestly a lot of this didn’t click for me until I realized that **Never** will likely be used in conjunction with _parameterized types_ like `Task` or `Html`.

## When is it useful?

As is the case with [extensible records](https://medium.com/@ckoster22/advanced-types-in-elm-extensible-records-67e9d804030d), the **Never** type is useful for _restricting function arguments_.

The [Task](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Task) module does a good job of demonstrating this. There are two functions in that module that produce a `Cmd msg`. The first function is `attempt`.

This function takes two pieces of information and returns a `Cmd`. The two arguments include:

- A task that might fail
- A function that takes the `Result` from executing the task and turns it into a `msg`

This is all reasonable. But what about tasks that can _never_ fail? For example, [Time.now](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Time#now) can’t fail. If a task cannot fail then we don’t need to handle the error case. In fact, there shouldn’t be any `Result` to handle at all!

Well, that’s what `perform` does.

The `perform` function operates on tasks that can never fail. Because a Task can be annotated with **Never** (the type of the error is Never) this function is guaranteed to never be called with a task that can fail. And since the task can never fail the first argument to the function can shrink from `(Result x a -> msg)` to `(a -> msg)`. Never is _restricting the function arguments_.

If you’d like to play around with how the Task module uses **Never** I put together a small example with `attempt`, `perform`, a task that can fail, and a task that will never fail.

_Note that you can replace the usage of_ `_taskCanFail_` _with_ `_taskWillNotFail_`_, but not the other way around._

## Conclusion

The Never type is a valueless type and like extensible records the Never type can be used to restrict function arguments, and that’s all there is to it!

Next up.. a look into [phantom types](https://medium.com/@ckoster22/advanced-types-in-elm-phantom-types-808044c5946d)..

_Shoutout to the folks on the #beginners slack channel for answering my questions and adding clarification in a friendly way. You’re all incredibly helpful!_