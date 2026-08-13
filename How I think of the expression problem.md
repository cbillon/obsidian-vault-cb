---
link:
---
# how I think of the expression problem 

The expression problem is a fundamental problem in programming language design. It is extremely simple, unsolved, and every programming language exhibits it. It is also extremely simple!

However, it has a fearsome reputation. Many have tried to understand it and given up after bashing their head against one incomprehensible explanations after another. It's not their fault – explanations tend to drown a clear, simple, and even beautiful concept under a sea of unnecessary complexities.

So read on, and you will join the elite club of those who understand the problem and maybe even have opinions of how to solve it.

### Can existing explanations possibly be that bad? 

No, not really, I found [a good one here](https://eli.thegreenplace.net/2016/the-expression-problem-and-its-solutions/) after writing the majority of this article. But the ones I had run into prior to that only tended to confuse me or obscure the issue. Let's look at [the expression problem's Wikipedia article](https://en.wikipedia.org/wiki/Expression_problem):

> The **expression problem** is a challenging problem in [programming languages](https://en.wikipedia.org/wiki/Programming_language "Programming language") that concerns the extensibility and modularity of statically typed data abstractions. The goal is to define a data abstraction that is extensible both in its representations and its behaviors, where one can add new representations and new behaviors to the data abstraction, without recompiling existing code, and while retaining static type safety (e.g., no casts). The statement of the problem exposes deficiencies in [programming paradigms](https://en.wikipedia.org/wiki/Programming_paradigm "Programming paradigm") and [programming languages](https://en.wikipedia.org/wiki/Programming_language "Programming language"), and as of 2023 is still considered unsolved, although there are many proposed solutions.

First of all, I won't blame you if your eyes glazed over at the phrase "the extensibility and modularity of statically typed data abstractions". It uses a ton of syllables to convey absolutely nothing to people who don't already know what it's talking about.

And then it claims the problem has a requirement of doing something or other "without recompiling existing code"! If this gave you pause, you were right. What code is compiled is an implementation detail of the compiler that unrelated to the semantics of the programming language. I guess we will be very happy to know that a typescript interpreter solves the expression problem, since you can do whatever you want while retaining static type safety and without recompiling anything (since nothing is compiled).

In fact, the expression problem has absolutely nothing to do with static typing[[1]](https://publish.obsidian.md/#fn-1-9f2bb6a72c3565a2), and it has nothing to do with recompiling existing code. Furthermore, it fundamentally cannot be solved without radically changing how we think of programming languages. But before we can consider solutions, we must understand the problem.

Let's say you're writing a game. In your game you have 3 different sorts of entities – the player, enemies, and items.

Let's further suppose there are 3 things you need to implement for each sort of entity:

1. **Render**: How to render it (which will depend on what sort of entity it is).
2. **Update**: How to update its state between frames (which will depend on what sort of entity it is).
3. **Destroy**: What to do when it is removed from the game world (which will depend on what sort of entity it is).

So now we have 3 sorts of entities, and 3 behaviors that must be implemented for each one. So you must implement  implementations total. We can represent this as a 2D matrix:

||player|enemy|item|
|---|---|---|---|
|render||||
|update||||
|destroy||||

This pattern comes up very often, and there's a fundamental decision programming language designers need to make – how should the programmer express this?

## The Behavior-Oriented Approach 

Let's see how you'd implement this in Rust, which takes what I'll call the "behavior-oriented" approach:

```rust
enum Entity {
  Player,
  Enemy,
  Item
}

impl Entity {
  fn render(&self) {
    match self {
      Player => { /* Render player */}
      Enemy => { /* Render enemy */}
      Item => { /* Render item */ }
    }
  }

  fn update(&self) {
    match self {
      Player => { /* Update player */}
      Enemy => { /* Update enemy */}
      Item => { /* Update item */ }
    }
  }
  
  fn destroy(&self) {
    match self {
      Player => { /* Destroy player */}
      Enemy => { /* Destroy enemy */}
      Item => { /* Destroy item */ }
    }
  }
}
```

As expected, we have  implementations to write. **Notice how the implementations are grouped by the behavior.**

## The Object-Oriented Approach 

Let's see how we'd approach this in some fictional OOP language[[2]](https://publish.obsidian.md/#fn-2-9f2bb6a72c3565a2):

```C#
abstract class Entity {
  void render();
  void update();
  void remove();
}

class Player : Entity {
  void render() { /* Render player */ }
  void update() { /* Update player */ }
  void destroy() { /* Remove player */ }
}

class Enemy : Entity {
  void render() { /* Render enemy */ }
  void update() { /* Update enemy */ }
  void destroy() { /* Remove enemy */ }
}

class Item : Entity {
  void render() { /* Render item */ }
  void update() { /* Update item */ }
  void destroy() { /* Remove item */ }
}
```

As before, there are  implementations. But this time, **notice how the implementations are grouped by the entity!**

## Which is better? 

A reasonable question. But it has no answer. No matter which you choose, you will eventually curse your language for not giving you the other.

1. When you want to understand the render process holistically, you'll wish you had each of the `render` behaviors grouped next to one another, which would correspond to the "behavior-oriented" (rust-like) approach. Instead, in an object-oriented language, you'll have them spread out across 3 different class definitions. Probably in 3 different files.
2. When you want to understand the player holistically, you'll wish you had all the `player` behaviors grouped next to one another, which would correspond to the "object-oriented" approach. Instead, in a behavior-oriented (rust-like) language, you'll have them spread out across 3 different functions. Probably in 3 different files.

Either way, you have spaghetti code.

1. If you want to change everything about a player, the object-oriented approach makes that easy, but the behavior-oriented (rust-like) approach forces you to jump around between functions.
2. If you want to change everything about the render process, the behavior-oriented approach makes that easy, but the object-oriented approach forces you to jump around between 3 different classes.

## So what the hell was the wikipedia article talking about? 

The wikipedia article, when talking about the expression problem, mentioned:

> The goal is to define a data abstraction that is extensible both in its representations and its behaviors, where one can add new representations and new behaviors to the data abstraction, without recompiling existing code

Now it becomes slightly more clear what "extensible both in its representations and its behaviors" means. Recall the table from earlier:

||player|enemy|item|
|---|---|---|---|
|render||||
|update||||
|destroy||||

Imagine we want to add a row - a new behavior, like `take_damage`. With the behavior-oriented (rust-like) approach, you do this "locally", by defining one new function and implementing the behavior in there for all three sorts of entity. (If you're looking at the problem overly closely, you might think this doesn't require recompiling existing code.) With the object-oriented approach, you'd need to add a new method to `Entity` and implement it separately in every class. (Which I guess might reasonably require recompiling existing code. But again, the whole thing about recompiling code is missing the forest for the trees here.)

Of course, as we have seen multiple times, the behavior-oriented approach solves half the problem while the object-oriented approach solves the other half. Behavior-oriented is great when you want to add a new row (AKA – a new behavior), but it suffers when you want to add a new column (AKA – a new sort of entity). To do that, you would need to go through every behavior and add a new case for the new type of entity you just added.

## The fundamental source of the problem 

The reason this problem is so pernicious is simple to see now. We have this 2D matrix of behaviors and entities:

||player|enemy|item|
|---|---|---|---|
|render||||
|update||||
|destroy||||

But in every programming language, we have to lay it out one-dimensionally! And the two best options seem to be to either group by column or by row, but neither option is clearly better than the other.

**This is the expression problem**. Your language may not encourage one particular grouping over another, but problem asks: How can we design a language where you can get the best of both worlds and don't have to choose?

# The solution 

We need programming languages where the implementation of each behavior for each sort of entity is laid out as a table! (Let's be glad that the table ends up needing to be 2D and not 3D.)

Of course, this has the problem that editing a table in a text editor is horrible. So this would seem to necessitate a programming language which has to be edited via an interface that that looks more like a spreadsheet than a text editor. I fear that the juice may not be worth the squeeze there, so maybe the problem just can't be happily solved.

However, maybe there's another option. The programming language [Unison](https://www.unison-lang.org/) stores your code in a binary format, not a textual one. (It also does lots of other cool stuff, like content-addressing your code, which makes renaming a function a simple operation.)

The code in Unison is stored in a binary format, but you of course need some way of editing it. So you have a tool called the Unison Codebase Manager, and you tell it "show me this function" and it puts it in a "scratch file" for you. You edit this file in your text editor like normal. Then you tell the tool "commit the new contents of the scratch file", and it adjusts the function stored in the codebase.

With this, perhaps the expression problem could finally be solved. Imagine if you could say "show me all the behaviors for `Player`" and the codebase manager magically summons them into your scratch file for you. Alternatively, you could say "show me all the implementations of `Render`" and then see those as well. It seems to me that this would go most of the way toward solving the problem.

---

1. In my interpretation of it, anyway. I understand that other people may understand the problem differently and I think that the question of how to avoid recompiling existing code is a very interesting one[↩︎](https://publish.obsidian.md/#fnref-1-9f2bb6a72c3565a2)
    
2. Sorry, but the object-oriented languages I know ([C++](https://chadnauseam.com/coding/pltd/the-good-and-bad-of-cpp-as-a-rust-dev) and C#) would have enough syntactic noise that it would obscure the concept I'm trying to communicate.[↩︎](https://publish.obsidian.md/#fnref-2-9f2bb6a72c3565a2)