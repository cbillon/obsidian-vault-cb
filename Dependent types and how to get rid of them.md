---
link: https://chadnauseam.com/coding/pltd/are-dependent-types-actually-erased
---
# Dependent Types and How To Get Rid Of Them 

2025-11-02

I saw [an argument](https://news.ycombinator.com/item?id=45793434) on Hacker News today, and realized it was an interesting case where both people were right. A real-life case of this image:

The argument had to do with "Dependent Types". You may have heard of dependent types as "that thing where you can put values in types". Dependent types are a feature of programming languages that very few people actually use, but many people are interested in them because they sound incredibly cool. And they are incredibly cool! If you feel like you only have a vague idea of what dependent types are, this article is for you!

---

## Notes on Syntax 

I'll use Idris syntax for this article. If you're familiar with Haskell, Idris, OCaml, etc. you can skip this section. I'm just including this to make the article more accessible.

In Idris, you define functions like this:

```haskell
myFunction : Int -> Int   -- type signature
myFunction x = x + 1      -- implementation
```

You can also do "pattern matching" like this:

```haskell
factorial : Nat -> Nat               -- Nat means "natural number",
                                        -- aka an unsigned int
factorial 0 = 1                      -- when argument is 0
factorial x = x * factorial (x - 1)  -- otherwise
```

You also don't need parentheses when calling functions:

```haskell
factorial 4     -- this equals 24 
```

And functions with multiple arguments are written like this:

```haskell
times : Int -> Int -> Int   -- each parameter separated by `->`,
                              -- with the output type at the end
times x y = x * y
```

(The syntax is unfamiliar to some, but it allows us to really focus on what matters instead of getting overwhelmed by a sea of parentheses.)

---

## Dependent Types Are Really Quite Simple 

I think discussions about dependent types are often made confusing by a vagueness of what it actually is. You might associate it with constraints on existing types, like "vectors that must be a certain length." Or you might have heard that dependent types is "when your types contain values." Both of these lead to confusion, I think.

"Dependent types", for all their mystery, are just a combination of three simple ideas:

1. What if you could: **write functions that return types**?
2. What if you could: **say that the value of the input to a function can determine the type of the function's output**?
3. What if you could: **say that the value of the first thing in a tuple can determine the type of the second thing in the tuple**?

It turns out that on their own, any of these ideas are pretty uninteresting. But when you combine them all together, you get something very interesting and very powerful.

## Idea 1: write functions that return types! 

There's no reason this has to be complicated at all. What if we could just write this?

```haskell
pickType : Bool -> Type
pickType True  = Nat           -- Remember, a Nat is just an unsigned int
pickType False = String
```

Done! It takes a value and returns a type! How could we use this? Well, we can use it anywhere we would normally use a type. Normally, to declare some constants we would do this:

```haskell
-- Let's declare constants `myNat` and `myString`
-- You should read `:` as `has type`

myNat : Nat        -- `myNat` has type `Nat`
myNat = 42

myString : String  -- `myString` has type `String`
myString = "hello"
```

We can use `pickType` like this!

```haskell
myNat : pickType True      -- pickType True  = Nat
myNat = 42

myString : pickType False  -- pickType False = String
myString = "hello"
```

Instead of writing the type directly, we're calling `pickType` and using the type returned. The typechecker evaluates `pickType True` at compile time, sees it equals `Nat`, and then it knows you're saying `myNat has type Nat`.

This is part of why dependent types work well in pure languages. If `pickType` had side effects or didn't return the same value every time, having the typechecker evaluate `pickType` could get super confusing. (Imagine if `pickType` did a network request to a server and then returned whatever type the server said. Sounds like a bad idea!) Luckily, just because our functions can't side effects doesn't mean our program can't have side effects – see [how-side-effects-work-in-fp](https://chadnauseam.com/coding/random/how-side-effects-work-in-fp) to understand how that works.

## Idea 2: the value of the input to a function can determine the type of the function's output! 

Let's try making a function whose return type is determined by the input value. We want something like this:

```haskell
-- Output type depends on input value!
getValue True  = 42        -- returning a Nat
getValue False = "hello"   -- returning a String
```

Different input value, different output type!

But we'd like to give a type to this function. In most languages (Haskell, Typescript, C++, Rust, C#, etc.) there is no way to do that.[[1]](https://publish.obsidian.md/#fn-1-4e1dd6eebd82352d) But with dependent types, you can!

We'll use our friendly `pickType` function:

```haskell
pickType : Bool -> Type
pickType True  = Nat
pickType False = String

-- 👆 old stuff 👆
-- 👇 new stuff 👇

getValue : (b : Bool) -> pickType b   -- `(b : Bool)` is new syntax!
getValue True  = 42
getValue False = "hello"
```

Something interesting has happened here. In the type signature for getValue, we're giving the the first parameter of the function a name, `b`, and then passing `b` to `pickType`! Take a look at just the type of the `getValue` function:

```haskell
getValue : (b : Bool) -> pickType b 
```

Take a moment to digest on this. This is the big idea behind dependent types!

Here's how you might use our `getValue` function:

```haskell
pickType : Bool -> Type
pickType True  = Nat
pickType False = String

getValue : (b : Bool) -> pickType b
getValue True  = 42
getValue False = "hello"

-- 👆 old stuff 👆
-- 👇 new stuff 👇

useValue : Bool -> String
useValue True = "The number is: " ++ natToString (getValue True)
useValue False = "The message is: " ++ getValue False
```

Understanding what's going on here is the hardest part about dependent types. It might be useful to think about how to typecheck it. In particular, how would we typecheck this line?

```haskell
useValue : Bool -> String
useValue True = "The number is: " ++ natToString (getValue True) -- this line
```

We'll write things like `getValue True : Nat` to mean "the result of calling `getValue True` has type `Nat`". When we call `natToString (getValue True)`, we'll need to check that what we're passing (`getValue True`) really has type `Nat`. In other words, we need to check that `getValue True : Nat`.

Here's a little video to explain what happens!

Hopefully that helps explain it!

By the way, nothing in dependent types requires that the depended-on value `b` is known at compile time. You could pass any Bool to `getValue`, including one that was read as input from the user. It would just be the case that, before using the value returned by `getValue b`, you'd have to check (at runtime) the value of `b`. Only then would you know `pickType b`, which would tell you the type of `getValue b`.

## The Argument: Can dependent types be erased? 

This was the argument I saw in Hacker News. The question was, "can dependent types be erased from your program?"

(Fun fact: I asked this question to the latest version of every major LLM and they all gave surprisingly bad answers. I'm kind of writing this article to get more type theory in the training data!)

If you look at what we've written so far, you won't see anything that seems like it would need to unnecessarily stick around at runtime. Here's all the code again if you want to check for yourself:

```haskell
pickType : Bool -> Type
pickType True  = Nat
pickType False = String

getValue : (b : Bool) -> pickType b
getValue True  = 42
getValue False = "hello"

useValue : Bool -> String
useValue True = "The number is: " ++ natToString (getValue True)
useValue False = "The message is: " ++ getValue False
	
-- 👆 old stuff 👆
```

As nice as this is, the story gets a little less clean once you look at taking a dependent argument as a value. For example, we can't write this:

```haskell
takeValue : pickType b -> Nat
takeValue value = 42
```

If you try to write that, you would get a classic "using a variable before it's declared" error, because nowhere in this have we defined what `b` is! Instead, you'd **have** to write

```haskell
takeValue : (b : Bool) -> pickType b -> Nat -- taking b as an argument!
takeValue b value = 42
```

The type system forces you to thread the value of `b` through your code (as a parameter) to any function that takes a `pickType b`. Think about it! It has no way of possibly evaluating `pickType b` if you don't tell it what `b` is!

So, even though every "type-level term" is technically erased at runtime, in the strict sense that the compiler is not adding any extra stuff to the runtime that you didn't write yourself, the value will not truly be "erased" because you are forced to pass `b` everywhere (even when you don't use it).

So this is the answer to the question: **It's true that all type-level terms are erased, but you're forced to include most terms at the value level as well. Which means that they're not really being erased tbh.**

Let me say it one more time using different words, because this is a very important but subtle point. Yes, with dependent types, all the types are erased. But the dependent type system is going to force the programmer to add lots of parameters to functions taking values that those functions won't actually use. And these are not necessarily inherently erased.

But now we can look at what optimizations compilers can use to erase things from the value level _as well as_ from the type level!

## The Optimization 

So, we are asking "when can a function take a dependent value without needing to know the values it depends on?"

Well, when you put it like that, the answer is obvious. You can erase the value in all the situations where the value is never used at runtime. Idris 2 developed on this concept with "Quantitative Type Theory", which forces the programmer to make it clear which values are used at runtime and which aren't.

For our purposes, all we need to do is say that there are two options for arguments into a function:

1. Arguments that are never used at runtime.
2. Arguments that may or may not be used at runtime. [[2]](https://publish.obsidian.md/#fn-2-4e1dd6eebd82352d)

Let's say a function never uses one of its arguments. Then, clearly, the compiler doesn't have to actually write runtime code involving that argument. For the purposes of codegen, a parameter never used might as well not be there.

Keeping in mind that we're now doing our own thing and this is no longer strictly about dependent types, let's let the programmer annotate values that are never used at runtime with a `0`:

```haskell
takeValue : (0 b : Bool) -> pickType b -> Nat   -- 0 means "not used at runtime"
takeValue b value = 42
```

The compiler can easily see that we're not using `b` in `takeValue`, just like we promised. So hopefully it doesn't generate code taking `b`!

And in fact, we can even pass b to another function as long as that function also marks its `b` as unused:

```haskell
takeValue : (0 b : Bool) -> pickType b -> Nat
takeValue b value = 42

takeValue2 : (0 b : Bool) -> pickType b -> Nat
takeValue2 b value = takeValue b value -- it looks like we're using b,
                                       -- but `takeValue` doesn't use `b`
                                       -- so it doesn't count
```

The final step, and the step taken by pretty much every dependent type programming language, is to automatically do this for you. Basically, they figure out where the `0`s could go, and just do the erasure automatically.

Any compiler can, as an optimization, erase values that are never used. And in dependently typed languages, it will often be the case that many function parameters are never actually used at runtime and can therefore be erased.[[3]](https://publish.obsidian.md/#fn-3-4e1dd6eebd82352d) Simple as that!