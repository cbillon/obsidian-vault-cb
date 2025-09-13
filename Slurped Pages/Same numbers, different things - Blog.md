---
link: https://blog.danielh.cc/blog/numbers
excerpt: where my words occasionally escape /dev/null
slurped: 2025-09-01T11:18
title: Same numbers, different things | Blog
tags:
  - absolute
  - relative
---

Here's a homework question:

> What is 5°C + 10°C?

The answer is obvious, right? But consider this — what if 5°C was the temperature outside last week, and 10°C is the temperature outside today? Then an answer of 15°C wouldn't make much sense — especially if you convert the units first! Reframing the question with the same values, adding 41°F and 50°F would result in 91°F — certainly very different than 15°C!

The difference here lies between _absolute_ and _relative_ quantities. A number can be _absolute_ — such as a timestamp, a location, or in this case, a temperature — or _relative_, such as a timestamp _delta_ (or duration), a location _offset_, or a temperature _difference_.

The first question would make sense if it was phrased like this:

> The temperature outside was 5°C last week. This week is 10°C warmer than last week. What is the temperature outside this week?

The quantity of 5°C would be the _absolute_ temperature, but the 10°C quantity would represent the temperature _difference_. Now an answer of 15°C makes sense, just by adding an _absolute_ and _relative_ quantity.

Similarly, the first question would make sense if it was phrased like this:

> The temperature on Tuesday is 5°C warmer than Monday, and the temperature on Wednesday is 10°C warmer than Tuesday. How much warmer is Wednesday compared to Monday?

Here, both 5°C and 10°C would be _relative_ quantities, and their sum would also be relative.

Putting everything together:

|Adding||becomes...|
|---|---|---|
|absolute|relative|absolute|
|relative|relative|relative|
|absolute|absolute|doesn't make sense!|

## In time [​](https://blog.danielh.cc/blog/numbers#in-time)

When dealing with time, the difference is clear with language — "December 25 at 11am" is absolute, but "2 hours later" is relative.

An issue prevalent in software (and in programming languages themselves) is the lack of distinction of absolute quantities and relative quantities. For example, JavaScript has `Date.now()` and Python has `time.time()`, but they can be misused easily. It surely doesn't make sense to use the value `Date.now() + Date.now()` — it would represent some time in the future (precisely, adding the amount of time since the beginning of 1970). Additionally, functions such as `setTimeout` and `time.sleep` would take a duration, but nothing is stopping you from writing `time.sleep(time.time())`.

Rust is one of the few languages that gets this right. Absolute timestamps are represented by either `SystemTime` or `Instant` (depending on the clock source), and relative durations are represented by `Duration`. `std::thread::sleep` takes in a `Duration`, so you can't write `sleep(Instant::now())` - the type system will stop you.

## In space [​](https://blog.danielh.cc/blog/numbers#in-space)

In Washington, D.C., "13th Street and U Street" represents an absolute position, while "2 blocks north and 1 block west" represents a relative offset. Like temperature and time, it doesn't make much sense to add an absolute position to another, but it certainly makes sense to add relative offsets.

However, Minecraft represents coordinates using vectors — an `(x, y, z)` triplet. A vector is used as relative offsets but also for absolute positions. But the problem doesn't end here:

- `BlockPos` (for representing absolute integer coordinates) extends class `Vec3i` (for representing relative integer coordinates)
- `Vec3i` has an `offset` method that takes in another `Vec3i`
- Nothing's stopping you from adding a `BlockPos` (_absolute_ position) to another `BlockPos` (another _absolute_ position) using `Vec3i.offset`, and getting back a `Vec3i` (a _relative_ offset)

It doesn't need to be like this! A simple fix would suffice:

- Make `BlockPos` no longer extend `Vec3i`
- Make `BlockPos.subtract` return `Vec3i`
- Make `BlockPos.offset` only take in a `Vec3i` and return `BlockPos`
- Everything in `Vec3i` can stay the same, since adding and subtracting relative values results in relative values

## Structural and nominal typing [​](https://blog.danielh.cc/blog/numbers#structural-and-nominal-typing)

A type system is _structural_ if types are equivalent if they have the same representation, while a type system is _nominal_ if types are equivalent only if they are explicitly declared to be equivalent.

TypeScript is (mostly) structural:

typescript

```
// These types are equivalent!
type Vector = { x: number, y: number, z: number }
type Point = { x: number, y: number, z: number }
```

C++, Java, Rust, and many others are nominal:

rust

```
// These types are different!

struct Vector {
    x: i32,
    y: i32,
    z: i32,
}

struct Point {
    x: i32,
    y: i32,
    z: i32,
}
```

How can these type systems deal with the difference between absolute and relative values? As seen above, nominal type systems can deal with this just fine — but can structural type systems do the same? Taking a look at TypeScript:

- Declaring a function that takes in a `Vector` (defined above) as a parameter is no different than a function that takes in a `Point`
- Returning a `Vector` from a function allows that value to be immediately used in place of a `Point`
- In fact, as seen above, the words `Vector` and `Point` are just aliases to the type structure (`{ x: number, y: number, z: number }`)

The only ways to deal with this are:

- Define classes in TypeScript (similar to Java), but this pattern is uncommon
- Inject more semantic value into the types at runtime by modifying the structure (such as changing the property names)

Python is a bit better:

python

```
from dataclasses import astuple, dataclass

@dataclass
class Point:
    x: int
    y: int
    z: int

    # This makes destructuring possible!
    def __iter__(self):
        return iter(astuple(self))
```

The `Point` class would be distinct from another type, even if they had the same fields.

## Subtraction [​](https://blog.danielh.cc/blog/numbers#subtraction)

In abstract algebra, subtraction is defined as adding the _additive inverse_. But this can pose some problems when dealing with absolute values:

- It certainly makes sense to subtract two timestamps — the difference between noon today and noon yesterday is exactly 24 hours
- It doesn't make sense for a timestamp to have an additive inverse, since it doesn't make sense to add timestamps together to begin with!

This makes subtraction different than addition. Putting it all together:

|Subtraction|becomes...|
|---|---|
|absolute - relative|absolute|
|relative - relative|relative|
|absolute - absolute|relative|
|relative - absolute|doesn't make sense!|

Notice how subtracting a relative value results in the same category as adding, since relative values can have an additive inverse. The only difference is subtracting an absolute value from another - this results in a relative value.

## The end [​](https://blog.danielh.cc/blog/numbers#the-end)

Next time you're dealing with numbers in code, think about whether they're absolute or relative. A position isn't just a triple of coordinates, and a timestamp isn't just milliseconds since 1970.

Think about whether your values are absolute or relative — your future self (and your computer) will thank you.