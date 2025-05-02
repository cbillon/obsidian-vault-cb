---
link: https://thoughtbot.com/blog/triangle-of-separation
byline: Joël Quenneville
site: thoughtbot
date: 2025-03-03T00:00
excerpt: Write code that's easier to read, remix, and modify by following this trio of principles.
twitter: https://twitter.com/@thoughtbot
slurped: 2025-03-12T08:47
title: Triangle of Separation
tags:
  - elm
---

Over the years, I’ve accumulated a set of three principles that help me write code that’s more modular, readable, and modifiable. I apply them when writing new code to try to make things easier for my colleagues, and I use them when refactoring existing code that’s gotten too complex to allow tacking on one more edge case.

Over time, I’ve recognized that many heuristics I use for code quality are just specializations of these principles. Here they are in one place.

## [Write code at a single level of abstraction](https://thoughtbot.com/blog/triangle-of-separation#write-code-at-a-single-level-of-abstraction)

![visualization of the idea to write code at a single level of abstraction](https://images.thoughtbot.com/1455lyuiw3ha5s7c82qyvrilxoco_single-level-abstraction.png)

Avoid writing code that is a mix of high and low level concepts. Instead, write top-level methods entirely in terms of other methods, that themselves do the work. These will often be private methods. This style of writing code will often lead **iceberg files** with 90% of the lines being in the private section and just a bit showing in the public section.

That way the reader of the high-level method can focus on _what_ the method does rather than _how_. This makes code **easier to read.**

This style is particularly helpful when [writing system/feature/acceptance tests](https://thoughtbot.com/blog/acceptance-tests-at-a-single-level-of-abstraction).

I also heavily use this style when [writing view code in Elm](https://github.com/JoelQ/ecosystem/blob/7f79ccb17b8ca6fedaaed23aaf5e1c79461aad70/src/Main.elm#L513-L864). Ideally readers of my top-level `view` function don’t need to care about the specifics of my rendering layer (HTML? Canvas?) and instead focus on what UI elements are present (e.g. a loading screen, main content, a summary, etc). Consider the following code to render a survey. All the pieces are written at a high level.

```
survey
  [ surveySection "Toppings"
    [ yesNoQuestion "Pineaple on Pizza?" PoP "pop"
    , yesNoQuestion "Anchovies?" Anchovies "ancho"
    ]
  , surveySection "Crust"
    [ pickOne
      [ ("Thin Crust", Thin)
      , ("Thick Crust", Thick)
      , ("Chicago Style", Chicago)
      ]
    ]
  ]
```

## [Push conditionals up the decision tree](https://thoughtbot.com/blog/triangle-of-separation#push-conditionals-up-the-decision-tree)

![visualization of the idea of pushing conditional logic up the decision tree](https://images.thoughtbot.com/09iww53aycr32qhcw61oztywmv73_push-conditions-up.png)

As programmers, we often like to write code that attempts to follow a single-ish main path through our code. To do this, we try to have side-branches either terminate quickly, or merge back into the main path. This leads to duplicated checks, deeply nested if/else, and unnecessarily complex conditional code.

Instead, [pushing conditions up the decision tree](https://matklad.github.io/2023/11/15/push-ifs-up-and-fors-down.html) means that you have to handle much of the uncertainy and edge cases of your code at the boundaries of your program. This leads to much more [confident code](https://www.youtube.com/watch?v=T8J0j2xJFgQ).

For the remaining conditions, branching early forces us to be honest about the fact that there is more than one main path through our program. Flat conditionals at the top of your decision tree are much easier to read, reason about, and debug.

Considier the following complex conditional from the [noisy animal kata](https://github.com/thoughtbot/noisy-animals-kata):

```
# BEFORE
def make_noise(loud: true)
  if is_bird && !loud
    make_bird_noise(false)
  end
  if loud
    if is_mammal
      2.times { puts mammal_noise }
    end
    make_bird_noise(true) if is_bird
  elsif  is_mammal
    puts mammal_noise
  end
end
```

Applying this principle removes a lot of the accidental complexity in the method. Now we can easily see how logic flows. It is also much easier to scan the method and see some other inconsistencies and refactors we could apply.

```
# AFTER
def make_noise(loud: true)
  if is_bird && loud
    make_bird_noise(true)
  elsif is_bird && !loud
    make_bird_noise(false)
  elsif is_mammal && loud
    2.times { puts mammal_noise }
  elsif is_mammal && !loud
    puts mammal_noise
  end
end
```

This principle has come in particularly clutch for me when [refactoring gnarly multi-step form code](https://thoughtbot.com/blog/structuring-conditionals-in-a-wizard), both on the front-end and back-end.

## [Separate branching from doing code](https://thoughtbot.com/blog/triangle-of-separation#separate-branching-from-doing-code)

![visualization of the idea to separate branching code from doing code](https://images.thoughtbot.com/5o3w22bsvr8mipaoubbyikvzwedu_separate-branching-logic.png)

This principle says that code that branches doesn’t get to have implementation details. If a method contains a conditional, each branch may only call out to another method. This guideline and the [Sandi Metz rule of max 5 lines per method](https://thoughtbot.com/blog/sandi-metz-rules-for-developers#five-lines-per-method) naturally lead to a similar place by minimizing how much you can fit in an `if/else` expression.

This principle emphasizes **compositionality** and **reuse** by breaking apart your algorithm at its natural joints.

I find myself applying specialized variants of this rule all the time, such as my recommendation to [separate expensive calculations and the memoization of those calculations](https://thoughtbot.com/blog/ruby-memoization-and-alternatives#separate-caching-and-calculation) to separate methods, or my recommendation to [separate code that checks for presence from code that calculates values](https://thoughtbot.com/blog/problem-solving-with-maybe#extracting-functions) when working with `Maybe` in Elm.

Consider the following smelly code:

```
# SMELLY
def save(for_real:)
  if for_real
    File.open("#{@title.downcase}.txt", "w") do |file|
      file.puts @title
      file.puts @body
    end
  else
    $stdout.puts "PREVIEW"
    $stdout.puts @title
    $stdout.puts @body.slice(0, 240)
  end
end
```

now compare with the alternative that applies the principle:

```
# BETTER
def save(for_real:)
  if for_real
    save_to_file
  else
    output_preview
  end
end
```

Now it’s easier to remix the various parts of the method. If someone knows they want to save to a file for sure, they don’t need to hardcode a boolean `save(for_real: true)` (thats a form of [control coupling](https://thoughtbot.com/blog/types-of-coupling#control-coupling)), instead they call the path they want to use directly as `save_to_file`.

If you plan on having sub-classes, particularly if you are using a [template method pattern](https://refactoring.guru/design-patterns/template-method), this style decomposes your method in a natural way that provides most of the necessary hooks for sub-classes to override.

## [Triangle of separation](https://thoughtbot.com/blog/triangle-of-separation#triangle-of-separation)

![diagrams representing each of the 3 principles overlayed on the vertices of a triangle labeled "triangle of separation"](https://images.thoughtbot.com/6400cj233tculk1epho0zbu0r00h_triangle-of-separation.png)

You may have been reading some of the examples and thought to yourself “this feels like it would have been a better example for one of the other principles”. That’s because all of of the principles converge on the same kind of code. They are 3 sides of a coin, err… triangle?

It’s the same idea viewed from a few different angles. I’ve nicknamed this trio the **triangle of separation**, because it’s all about giving you heuristics for discovering seams for separating code.

Taken together, these ideas help you write code that’s easier to read, easier to change, easier to extend, and easier to remix.