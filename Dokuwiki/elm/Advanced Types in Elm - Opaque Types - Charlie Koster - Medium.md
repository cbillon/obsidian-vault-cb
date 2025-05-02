---
link: https://ckoster22.medium.com/advanced-types-in-elm-opaque-types-ec5ec3b84ed2
byline: Charlie Koster
site: Medium
date: 2017-08-28T03:37
excerpt: There are a handful of ways to use types in Elm that you might not see in a Hello World app, but you may more commonly see in published packages and production applications. This post introduces…
twitter: https://twitter.com/@ckoster22
slurped: 2024-08-17T18:21
title: Advanced Types in Elm - Opaque Types - Charlie Koster - Medium
tags:
  - elm
  - type
---

## Part I: Opaque Types

[

![Charlie Koster](https://miro.medium.com/v2/resize:fill:88:88/1*MIhMxl88uo7fCu84SRfqsg.jpeg)



](https://ckoster22.medium.com/?source=post_page-----ec5ec3b84ed2--------------------------------)


There are a handful of ways to use types in Elm that you might not see in a _Hello World_ app, but you may more commonly see in published packages and production applications. This post introduces **opaque types**.

## What are Opaque Types?

Opaque types are types that hide their internal implementation details within a module. While this statement seems benign on its surface, it’s an incredibly important concept in an ecosystem that _enforces semantic versioning_.

## The Problem

Let’s assume we aren’t using opaque types and we create a SortedLabels module meant to track labels for other records.

Let’s pay attention to a few lines of code. First, the **SortedLabels** alias.

Here we are identifying that this module has two important pieces of data: a list of strings for the labels, and a union type that will indicate the sort direction.

Next, let’s look at the implied invariant.

The intent behind this function is for consumers of this module to invoke it so that it can return a sorted list of string labels for a given **SortedLabels** record.

Lastly, let’s look at the module definition.

The **asList** function is exposed, which is to be expected, as is **SortedLabels**. But notice that the **SortDirection** type has its constructor functions exposed (the _(..)_ part). This is necessary because in order for consumers of this module to create a record that fits the **SortedLabels** shape a **SortDirection** is also needed.

Here’s an example of how the SortedLabels module might be consumed.

This.. is not the greatest looking Elm code. There are a few nasty parts to point out.

- The implementation details of **SortedLabels** have leaked into the **addArticle** and **addKeyword** functions.
- The **addKeyword** function is overly verbose and closely resembles nested record update syntax.
- The implied invariant (the labels should be sorted) is not upheld. If we don’t call the **asList** function then we can use **SortedLabels** in an unsorted way.
- The above code isn’t protected from breaking changes. Let’s say we realize that the labels should be unique and rather than using a **List** to represent the labels we want to use a **Set**. We _should_ be able update the **SortedLabels** module without causing any breaking changes, but we can’t with how it’s currently implemented.

## The Solution

Let’s refactor the **SortedLabels** modules to use **opaque types**.

Ok, that’s more code than we had before. Let’s analyze this.

- The **SortedLabels** alias became a type with a single constructor function. _It’s not uncommon to see the constructor function name match the type name._
- The constructor function for **SortedLabels** _is not_ exposed in the module definition. That type is _opaque_. Instead, **ascendingLabels** and **descendingLabels** are functions that are exposed to construct the opaque type.
- **SortDirection** no longer needs to be exposed at all. It’s purely a type used internally to this module.
- There is only one way to get the list of labels and that’s through the **asList** function. This gives us greater confidence that the “sorted” invariant is being upheld. This also means that **addLabel** doesn’t need to care about adding new labels in the correct sort order since they will be sorted by the **asList** function, the only function that allows you to see the labels.

> Note that opaque types do not have any special syntax. They are essentially types (not type aliases) whose constructor functions **are not exposed** from the module.

The code consuming this module also cleans up a little.

We traded off a more verbose module implementation for more readable code that consumes that module, and the “sorted” invariant can’t be broken by the consuming code anymore.

There’s one more benefit. Remember the part about not duplicating labels? If **SortedLabels** was implemented with a **Set** under the hood that would be a nice way to prevent duplicates.

This refactor changed a lot and it in fact added more lines of code. We changed **List String** to **Set String** and flipped the arguments in the **SortedLabels** type as a matter of convenience. That cascaded changes in the constructor function invocation in **ascendingLabels** and **descendingLabels**. It also caused a minor change to any function doing pattern matching on **SortedLabels**. We also sprinkled in some **Set.toList**, **Set.fromList**, and **Set.insert** where necessary.

That seems like a lot, and it is. But what didn’t change?

Well, **the module definition didn’t change**. We may have changed the underlying implementation and types but this module is still exposing the same API. As a result, **the code consuming this module also didn’t change**.

This is great because now there is a clean separation between what consumers of the module need to know and the module’s internal implementation.

As stated earlier, this is also important in Elm’s package ecosystem because semantic versioning is enforced. When you use opaque types you can more confidently change implementation details of your packages without being concerned about bumping the major version of your package.

More importantly, you don’t need to be concerned that changing the implementation details of a module or package will cause rippling effects to code consuming that module or package.

Next up, [extensible records](https://medium.com/@ckoster22/advanced-types-in-elm-extensible-records-67e9d804030d).