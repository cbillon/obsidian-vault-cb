---
link: https://ckoster22.medium.com/advanced-types-in-elm-extensible-records-67e9d804030d
byline: Charlie Koster
site: Medium
date: 2017-09-02T03:35
excerpt: In Part I we learned how opaque types can help separate module implementation details and avoid unnecessary breaking changes. This post covers another so-called advanced type, extensible records. An…
twitter: https://twitter.com/@ckoster22
slurped: 2024-08-17T18:23
title: Advanced Types in Elm - Extensible Records - Charlie Koster - Medium
tags:
  - elm
  - type
---

## Part II: Extensible Records

[

![Charlie Koster](https://miro.medium.com/v2/resize:fill:88:88/1*MIhMxl88uo7fCu84SRfqsg.jpeg)



](https://ckoster22.medium.com/?source=post_page-----67e9d804030d--------------------------------)

In [Part I](https://medium.com/@ckoster22/advanced-types-in-elm-opaque-types-ec5ec3b84ed2) we learned how _opaque types_ can help separate module implementation details and avoid unnecessary breaking changes. This post covers another so-called advanced type, **extensible records**.

## What is an extensible record?

An extensible record is a record defined as _“having at least certain fields”_. For example, a record can be defined as having at least an _id_ and _createdBy_ field.

The **attribution** function can be invoked with either of these two records.

A **BlogPost** is a record that has a title, contentMarkdown, id, and createdBy field. A **Memo** is defined as having a recipients, message, id, and createdBy field. Both contain an **id** and **createdBy** field because both are _extensions_ of the Authored extensible record.

Extensible records can be used (and abused) for a few different use cases. This post will specifically cover how extensible records can help with _narrowing function arguments_. Most of this content derives from part of Richard Feldman’s Elm Europe talk available below.

## The Problem

Let’s assume we’re working with a somewhat large, flat model.

Now consider a function that will update the fullName field.

The above code may not seem too terrible, but there are at least two reasonable criticisms of that function.

1. The argument types for this function are too broad. Specifically, this function is only concerned with updating the fullName field but it accepts and returns an entire **Model**. As a result, this is one more function to rule out when hunting down bugs relating to the model.
2. The unit test for the **updateFullName** function will be relatively tedious to write and maintain because we have to mock up an entire **Model** to satisfy the compiler.

Here is an example unit test that will quickly make it apparent just how tedious unit testing a function with broad arguments can be.

That unit test has a lot of unnecessary details. We had to mock up an entire **Model** because the arguments to **updateFullName** were not restrictive enough. Also note that we are only making an assertion on the **fullName** field. All of the other mock data is ignored.

Later on if we decide to add more fields to **Model** then we have to update this unit test (even though this unit test is unrelated to the newly added fields) and come up with more mock data that will ultimately be discarded.

## The Solution

We can change the **updateFullName** function to instead use an _extensible record_ which will in turn _restrict the arguments_ to this function.

The only thing that changed here is the **Model** argument and return value were swapped out with an extensible record called **Named**. It was swapped out with something that says _“here is a record that has a fullName field”_.

And that’s it! This function can still be invoked with a **Model** and a new **Model** will be returned with the **fullName** field updated.

One really great advantage to using extensible records to narrow the arguments of your functions is it also narrows which part of the program could be causing a bug. The old **updateFullName** function returned a **Model**. If a bug was discovered that was related to the **cardNumber** field then **updateFullName** is in the list of candidate functions that have to be checked for being the cause of the bug.

On the other hand, when **updateFullName** is using an extensible record then there is no way it can be responsible for a bug with **cardNumber**. It doesn’t work with any records that contain the **cardNumber** field!

Additionally, because we’ve narrowed the arguments for this function the unit test became more focused and less tedious to write and maintain.

If another field is added to **Model** then we don’t need to concern ourselves with updating this unit test. It is completely isolated from other parts of the model.

## Conclusion

Extensible records are a great construct for narrowing the types of your functions. They focus the argument types which in turn can help with debugging your application as well as writing and maintaining your unit tests.

Again, a big shout out to Richard for inspiring the content of this post. Be sure to check out his video from Elm Europe on [_Scaling Elm Apps_](https://www.youtube.com/watch?v=DoA4Txr4GUs) and also drop him a note of gratitude on [Twitter](https://twitter.com/rtfeldman).

Next up, the [Never type](https://medium.com/@ckoster22/advanced-types-in-elm-the-never-type-ca9b3297bbd4).