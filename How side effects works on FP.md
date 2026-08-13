---
link: https://chadnauseam.com/coding/random/how-side-effects-work-in-fp
---
# How Side-Effects Work in FP 

When you think about functional programming, you might think of banning mutation, or of functions without side effects. But in my opinion, the fundamental idea of FP is _equational reasoning_. That is, this code:

```js
x = foo()
y = bar(x)
z = bar(x)
```

Should mean the same thing as:

```js
y = bar(foo())
z = bar(foo())
```

Put explicitly: If you write `x = foo()`, that means you should be able to replace any occurrence of `x` with `foo()` without changing the meaning. That you can't have mutation or side-effects follows from this principle.

1. If `foo()` had side effects, then the first example (where we call `foo` once) would have different behavior than the second (where we call it twice).
    
2. If `bar(x)` could mutate `x`, then changing `bar(x)` to `bar(foo())` would change what your program means.
    

By reducing what you can do, we allow a way of reasoning about programs that would otherwise be basically impossible. Math traditionally uses equational reasoning a lot, and not for nothing – it's really useful!

## Can we still have effectful programs? 

I'm going to be using a Rust-inspired syntax for this post, where you can write something like this to describe an enum with two variants:

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
}
```

So you could create a value of type `Message` by writing `Move {x: 3, y: 5}` or `Quit`.

Also, you can define anonymous functions with syntax like `|x| x + 10`. That describes a function that takes a value `x` and returns `x + 10`.

With the restrictions of functional programming, it's not obvious how to express programs that, y'know, actually do stuff. But it's not as hard as it seems. You just have a special variable, called `main` or something like that, and allow the user to set `main` to what I'll call an "action value" that _describes_ the behavior of a program. What could this data structure look like?

1. Start out with an action value that doesn't really do anything, like `Done`.
    
    If you write `let main = Done`, your program will naturally immediately terminate without doing anything.
    
    So we're starting out with:
    
    ```rust
    enum ActionValue {
      Done
    }
    ```
    
    Ideally we would want to be able to write more interesting programs than that!
    
2. Let's extend our `ActionValue` type a little bit:
    
    ```rust
    enum ActionValue {
      Done,
      Print {to_print: String, then: ActionValue}
    }
    ```
    
    Now you can write `main = Print {to_print: "Hello world", then: Done}`, and your program will print `Hello world` and then be done!
    
    Or you could write:
    
    ```rust
    let main = Print {
      to_print: "Hello world",
      then: Print {
        to_print: "Goodnight moon",
        then: Done
       }
     }
    ```
    
    And that would print `Hello world`, then print `Goodnight moon`, then be done. Ok, we're most of the way there, but we still need a way to write programs that take input from the outside world!
    
3. Let's extend our type one final time.
    
    ```rust
    enum ActionValue {
      Done,
      Print {to_print: String, then: ActionValue},
      Input {then: String -> ActionValue}
    }
    ```
    
    Now `Input`'s `then` parameter takes a function of type `String -> ActionValue`, instead of a plain action value like `Print`'s does. Which means we can write something like this:
    
    ```rust
    let main = Input {
      then: |what_the_user_typed| Print {
        to_print: "Hello, " + what_the_user_typed,
        then: Done
       }
     }
    ```
    
    First, the computer sees `Input`, so it gets a line of input from the user. Then it passes what the user typed to the `then` parameter, and executes the action value it returns. So the result is the classic program that allows the user to type their name and greets them.
    

That's all we need! So why do languages like Haskell have their dreaded "monads"? Turns out all they do is simplify the construction of action values. (They're useful for other things too, but that's why they were introduced.) And it turns out with an sufficiently advanced compiler, it's not even that hard to convert an action value to pretty-efficient imperative code. So we gain equational reasoning without sacrificing any expressive power or even very much performance. Neat!

## I don't know why people try to explain the IO monad without explaining this first 

Monads often are just a convenient way to build up action values. In Haskell, the end result is that you can write code like this:

```haskell
main =
  getline >>= (\what_the_user_typed ->
  print ("Hello, " ++ what_the_user_typed))
```

`>>=` is a built-in operator for combining action values, or anything else with a monad structure.

Or even

```haskell
main = do
  what_the_user_typed <- getline
  print ("Hello, " ++ what_the_user_typed)
```

And you'll get something that's semantically identical to the above. The `<-` works like an `=`, except it signals that equational reasoning _doesn't_ apply to this value. You can't replace `what_the_user_typed` with `getline` - your program won't compile. That's not a design feature that had to be added – it's inherent to the world of monads!

Links to this page

[are-dependent-types-actually-erased](https://chadnauseam.com/coding/pltd/are-dependent-types-actually-erased)

[faq](https://chadnauseam.com/faq)