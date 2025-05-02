---
link: https://ckoster22.medium.com/advanced-types-in-elm-phantom-types-808044c5946d
byline: Charlie Koster
site: Medium
date: 2017-11-20T18:59
excerpt: The fourth part in this series is Phantom Types and as someone coming from a background in the C family of languages this concept is especially intriguing. Given that background, this post will not…
twitter: https://twitter.com/@ckoster22
slurped: 2024-08-17T19:27
title: Advanced Types in Elm - Phantom Types - Charlie Koster - Medium
tags:
  - elm
  - type
---

## Part IV: Phantom Types

The fourth part in this series is Phantom Types and as someone coming from a background in the C family of languages this concept is especially intriguing.

Given that background, this post will not be an exhaustive description of phantom types, their use cases, and trade-offs. Like the post on [extensible records](https://medium.com/@ckoster22/advanced-types-in-elm-extensible-records-67e9d804030d) I’ll instead discuss a potential use case and encourage the reader to explore more.

## What are Phantom Types?

A phantom type is **a parameterized type where one or more parameters on the left hand side do not appear on the right hand side**.

```
type APhantomType a  
    = Tag1 String  
    | Tag2
```

> [!NOTE]
> In the above code there is a parameter `a` that does not appear on the right hand side of the expression.

A type is technically a phantom type if any parameterized types do not appear on the right hand side which means that this is also a phantom type.

type APhantomType a b  
    = Tag1 b  
    | Tag2

Again, even though `b` is used on the right hand side `a` isn’t which qualifies this as a phantom type.

## When is this useful?

Phantom types are useful for **restricting function arguments** _(I’m beginning to think I should have named this series “Restricting function arguments in Elm”)_.

Phantom types do this by allowing functions to _carry more information at the type level_. Let’s look at an example involving form validation without phantom types.

The above example allows a user to enter a keyword that will be used to search for github repositories containing that keyword. However notice what happens when a search is performed without any keywords present.

This form lacks validation. The user is attempting to perform a search while a keyword is actually required. This form shouldn’t even make an Http request.

There are a few obvious ways to resolve this, the most common would be creating a function that returns a Bool which indicates whether the input is valid. But a lot of these obvious strategies only get us _runtime guarantees_. We can actually use phantom types to get guarantees at _compile time_.

> Phantom types can be used to obtain guarantees at compile time.

Let’s dive into this. The function in question is **fetchRepos**.

This function will accept any possible string and always produce a `Cmd`. That function signature is far too trusting that developers will send it a valid string. Can we update the function annotation such that it will only ever receive valid data? Yep! First, let’s define our types.

Here I’ve defined a few types.

- **FormData** is a phantom type. It has a parameter `a` that isn’t used in its enumerated values.
- **Valid** and **Invalid** are [opaque types](https://medium.com/@ckoster22/advanced-types-in-elm-opaque-types-ec5ec3b84ed2). Their constructor functions should be hidden. These are also the types that will take the place of `a` in **FormData**.
- **FormField** is what it sounds like. It contains the information for each field in a form.

> Since this example only has a single input I chose to have **FormData** contain a single **FormField**, but it would more likely be a **List FormField**.

Not only is **FormData** a phantom type but we also want it to be an [opaque type](https://medium.com/@ckoster22/advanced-types-in-elm-opaque-types-ec5ec3b84ed2). As a result we need functions to construct a **FormData**.

> Note: I want to allow a developer to construct an invalid form but I want to disallow developers from manually constructing valid forms. We’ll expose a **validate** function later to do that.

Here it’s revealed that the phantom type will be used in one of two ways. **FormData** will either be valid or it will be invalid. Let’s pause here and update **fetchRepos**.

The important part here is the annotation for **fetchRepos**. What used to be a **String** has been replaced by a **FormData Valid**. We have _restricted the function arguments_ such that **fetchRepos** can only be invoked with valid form data. Attempting to invoke it with an invalid form **will result in a compiler error**.

It’s worth pausing and understanding how neat this really is. We changed a function that used to produce a failure result at runtime. Rather than updating it with a guarded check we have instead enlisted the compiler to force consumers of this function to only send it valid data!

The last piece of the puzzle to show is how an invalid and valid **FormData** are constructed.

A **FormData Invalid** can be constructed by invoking the **invalidFormData** function above.

A **FormData Valid** can only be constructed by invoking another exposed function defined below.

Given an invalid form, this function will return a **Result** containing the valid form if it is in fact valid.

Here is the example with all of the pieces put together.

Ellie doesn’t support defining separate modules and therefore opaque types are difficult to show so I’ve added a comment where there would be a module separation.

## Should you use phantom types?

Based on some offline and online discussions I gather that phantom types aren’t incredibly common. They do have their use cases but you likely won’t see them used as frequently as some of the other advanced types in this series.

I also haven’t personally used them extensively and the above form validation example will make that obvious. So I encourage readers to read more about phantom types on their own and continue to ask questions in Elm Slack.

