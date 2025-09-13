---
link: https://blog.danielh.cc/blog/sat
excerpt: where my words occasionally escape /dev/null
slurped: 2025-09-01T11:14
title: Accidentally writing a fast SAT solver | Blog
tags:
  - solver
---

In my freshman year of college, I found myself staring at the screen looking over this for the first time:


Instead of being assigned to recommended classes in high school, I had the pleasure (and burden) of figuring out the schedule for my next 4 months on my own. After going over required courses, recommended courses, optional courses, and what others are taking, I finally determined the perfect mix of courses to sign up for — only to find out that sections were filled, waitlists were closed, and the courses I wanted had time conflicts.

I settled for a schedule that wasn't great — some classes conflicted with other sections I wanted, others were full by my registration time, and a few were at terrible times. Each semester was the same story: staring at registration pages, checking waitlists, and juggling different combinations of sections, hoping something would work out.

## Seeking simple schedules [​](https://blog.danielh.cc/blog/sat#seeking-simple-schedules)

Next semester, I tried to plan everything out. There were many websites that had access to UMD's course data, so I could pick classes and see exactly what my schedule would look like. This took a lot of time — but it didn't end up being of much use either, since as time passed, more classes would fill up before my registration date, and I had to start all over again.

![xkcd 1319 (Automation)](https://blog.danielh.cc/assets/Dc0wuZ2i.png)

Like many before me, I decided to automate this. I wrote a simple brute force program that considered all valid combinations of sections from a given list of classes. With 5 classes with 4 sections each, this would only need to consider 625 combinations. However, this wasn't great. I had to figure out exactly which classes to take, and many of them would have a lot more than just 4 sections. I needed to try a different approach.

## Building better backtracking [​](https://blog.danielh.cc/blog/sat#building-better-backtracking)

To understand how to build a better solver, let's look at a similar problem. There's a well known [eight queens puzzle](https://en.wikipedia.org/wiki/Eight_queens_puzzle):

> Place 8 chess [queens](https://en.wikipedia.org/wiki/Queen_\(chess\)) on a chessboard where no queen is attacking another queen.

Similar to how signing up for a course would "block off" parts of the week from other classes (otherwise, there would be a time conflict), placing a queen on a chessboard would "block off" squares on the board. A simple way would be to solve this by brute force - since there will be exactly one queen placed on each rank on the board, trying every combination would result in a solution. However, this would quickly spiral out of control if the puzzle was extended to a larger board — a 20x20 board with 20 queens placed would be much harder to find using the same technique.

Here's a simple example of a brute force solver for a smaller version of this puzzle on a 4x4 board:

python

```
for x1 in range(4):
    for x2 in range(4):
        for x3 in range(4):
            for x4 in range(4):
                if is_valid([x1, x2, x3, x4]):
                    print(x1, x2, x3, x4)
                    return
```

Here, the values `x1`, `x2`, `x3`, and `x4` would be the positions of the queens in each row.

INFO

On this board, `x1 = 1`, `x2 = 3`, `x3 = 0`, and `x4 = 2`.

![Board with 4 queens](https://blog.danielh.cc/assets/B1nBX_zZ.png)

Notice how this solver is doing a lot of unnecessary work. For example, on the first attempt with `x1 = 0` and `x2 = 0`, even before entering the loop with `x3`, the board looks like this:

![Board with 2 queens](https://blog.danielh.cc/assets/DgeEFUKo.png)

Without going through `x3` and `x4`, there's already a conflict! However, the solver would check all combinations for `x3` and `x4` anyway, even though we already know that it wouldn't find any solution.

Knowing this, the solver can be improved like this:

python

```
for x1 in range(4):
    if is_valid([x1]): 
        for x2 in range(4):
            if is_valid([x1, x2]): 
                for x3 in range(4):
                    if is_valid([x1, x2, x3]): 
                        for x4 in range(4):
                            if is_valid([x1, x2, x3, x4]):
                                print(x1, x2, x3, x4)
                                return
```

With these extra checks, if there's a conflict that shows up in the top few rows, the solver would know that it isn't worth it to search the remaining rows, since there couldn't be any solution.

How can this be generalized to an arbitrary board size instead of a fixed size? The original brute force solver can be done recursively:

python

```
def solve(size, rows, amount_left):
    if amount_left == 0:
        if is_valid(rows):
            print(rows)
            exit()
    else:
        for r in range(size):
            solve(size, [r] + rows, amount_left - 1)

solve(4, [], 4)
```

How does the recursion work?

If this conversion doesn't make sense, think of a recursive call as "pasting" a copy of the function body in place of the call. For this, assume that `size = 4`. If `amount_left` is zero, then `solve` would look like this:

python

```
def solve_0(rows):
    if is_valid(rows):
        print(rows)
        exit()
```

Then, continuing with `amount_left = 1`:

python

```
def solve_1(rows):
    # We change the name to `x4` to avoid name collision later on
    for x4 in range(4):
        solve_0([x4] + rows)
```

Now we "paste" in the code for `solve_0`:

python

```
def solve_1(rows):
    for x4 in range(4):
        if is_valid([x4] + rows): 
            print([x4] + rows) 
            exit() 
```

Next, with `amount_left = 2`:

python

```
def solve_2(rows):
    # Changing the name again for the same reason
    for x3 in range(4):
        solve_1([x3] + rows)
```

"Pasting" in `solve_1`:

python

```
def solve_2(rows):
    for x3 in range(4):
        for x4 in range(4): 
            if is_valid(rows + [x3, x4]): 
                print(rows + [x3, x4]) 
                exit() 
```

Building up, we eventually arrive at the following:

python

```
def solve_4(rows):
    for x1 in range(4):
        for x2 in range(4):
            for x3 in range(4):
                for x4 in range(4):
                    if is_valid(rows + [x1, x2, x3, x4]):
                        print(rows + [x1, x2, x3, x4])
                        exit()
```

However, the initial call (`solve(4, [], 4)`) would be `solve_4([])`, so `rows` can be removed entirely:

python

```
def solve_4():
    for x1 in range(4):
        for x2 in range(4):
            for x3 in range(4):
                for x4 in range(4):
                    if is_valid([x1, x2, x3, x4]):
                        print([x1, x2, x3, x4])
                        exit()
```

This would match the original solver!

To eliminate unnecessary work, we can add the improvement back:

python

```
def solve(size, rows, amount_left):
    if amount_left == 0:
        if is_valid(rows):
            print(rows)
            exit()
    else:
        for r in range(size):
            if is_valid([r] + rows): 
                solve(size, [r] + rows, amount_left - 1)

solve(4, [], 4)
```

This is _backtracking_. The idea here is that when a conflict is found (even if some of the rows don't have values yet), the solver "goes back" (either to an outer loop in the nested loop version, or to a previous call in the recursive version) and changes its guesses for earlier values.

Going back (no pun intended) to the scheduling problem, the solution would be similar. Each "row" would be a class, and from each class, a section would be chosen for the class schedule. By changing `is_valid` to check for time conflicts instead of queens, the same code can be used to build a schedule.

INFO

Using these ideas, I built [UMDB](https://github.com/danielhuang/umdb), a schedule builder that extends this concept further. Beyond just handling required courses (like placing all queens on a square board), it can handle optional courses too — similar to placing queens on a rectangular board where not every row needs a queen. The source code is available if you're interested in the implementation details.

## Linking logical lessons [​](https://blog.danielh.cc/blog/sat#linking-logical-lessons)

After building (and using) this scheduler, I realized it was solving something far more general than just course conflicts.

Another well-known problem (specific to computer science) is [boolean satisfiability](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem) (SAT). This problem involves _satisfying_ (making it `true`) a _boolean formula_ (operations on boolean variables). For example, the formula `a && b && !c` has 3 variables, and is _satisfiable_ since with `a = true`, `b = true`, and `c = false`, the whole formula evaluates to `true`.

It turns out that any formula can be converted into [conjunctive normal form](https://en.wikipedia.org/wiki/Conjunctive_normal_form) — that is, it's a bunch of disjunctions of atomic propositions connected into a conjunction:

- An _atomic proposition_ is a boolean variable or its negation: `a`, `!b`, and `c` would be atomic propositions. (A lot of other sources don't consider negations of variables to be atomic, but I will here for the purpose of brevity.)
- A _disjunction_ is a combination of propositions connected by `||` (or): `a || !b || c` is a disjunction of atomic propositions.
- A _conjunction_ is a combination of propositions connected by `&&` (and): `a && !b && c` is a conjunction of atomic propositions.
- A formula in _conjunctive normal form_ is a conjunction of disjunctions of atomic propositions: `(a || !b || c) && (d || e)` is in conjunctive normal form.

A formula in conjunctive normal form can then be converted into a course catalog and a set of requirements where a class schedule can be made if and only if the formula can be satisfied.

Details

In order to convert a formula into class schedule requirements, first list a class for each disjunction in the formula. Each class would have a section for each variable in the disjunction. For example, for the disjunction `a && !b`, "Class 1" can be added to the catalog with 2 sections: "Section 1" and "Section 2".

For a formula in conjunctive normal form, like `(a || !b || c) && (!a || !c)`, the course catalog would look like this:

- Class 1 (`a || !b || c`)
    - Section 1 (`a`)
    - Section 2 (`!b`)
    - Section 3 (`c`)
- Class 2 (`!a || !c`)
    - Section 1 (`!a`)
    - Section 2 (`!c`)

Now, the sections for each class will need to determine when to meet for class. For each variable (`a`, `b`, and `c` in this example), find a different time of the week to schedule a lecture. Here, `a` will be assigned to Monday, `b` will be assigned to Tuesday, and `c` will be assigned to Wednesday.

Here, Class 1 Section 1 (`a`) will meet on Monday. Class 1 Section 2 (`!b`) will meet on Tuesday.

A student now wants to take both classes in the catalog, and has to choose a section for each class. In this example, it's possible by choosing Section 1 for Class 1, and Section 2 for Class 2 (classes will meet on Monday and Wednesday). This corresponds to setting `a = true` (since Class 1 Section 1 is `a`), and setting `c = false` (since Class 2 Section 2 is `!c`), making the formula true (whether `b` is true or false doesn't matter, since both options will satisfy the formula anyway).

Another formula is `(a) && (!a)`. The course catalog will look like this:

- Class 1 (`a`)
    - Section 1 (`a`)
- Class 2 (`!a`)
    - Section 1 (`!a`)

There is only one option for each class, but this wouldn't work — they both meet on Monday, so there's a time conflict! This is just like how the formula isn't satisfiable, since `a` cannot be both true and false at the same time.

As a result, in order to determine if a formula is satisfiable, first convert it to conjunctive normal form, then convert the new formula into a course catalog. Put this into a schedule builder, and you know the answer.

## Discovering digital destiny [​](https://blog.danielh.cc/blog/sat#discovering-digital-destiny)

Just like how Fleming discovered penicillin or how Spencer invented the microwave (okay, not that drastic), what started as a simple tool to help with course registration turned out to be very different than expected. [UMDB](https://github.com/danielhuang/umdb), originally designed to handle course schedules with optional classes, can be used to solve any boolean satisfiability problem through this conversion process. The backtracking algorithm that helped avoid time conflicts in schedules is the same one that efficiently searches for satisfying assignments in boolean formulas - solving even complex requirements in under 100ms.

INFO

Of course, this wouldn't necessarily work for all possible inputs — since boolean satisfiability (SAT) is [NP-complete](https://en.wikipedia.org/wiki/NP-completeness), unless [P=NP](https://en.wikipedia.org/wiki/P_versus_NP_problem), it's always possible to come up with a boolean formula that would take a long time to check, and therefore would form a course listing that would take a long time to build a schedule for.

The journey from frustrating course registration to discovering an efficient SAT solver highlights something interesting about problem-solving in computer science. Sometimes, solving an everyday problem leads us to reinvent something fundamental. The same principles that help arrange classes without time conflicts turn out to be perfect for determining whether complex logical statements can be satisfied. This discovery also shows how different problems can be more connected than they appear. Whether you're trying to place queens on a chessboard, schedule university classes, or solve boolean formulas, the core challenge remains similar — finding valid combinations while ruling out impossible ones early. The only thing that changes is just how we represent the problem.

In the end, the frustration of manual course registration led to building something more interesting than expected. What started as a simple schedule planner turned into an efficient SAT solver hiding in plain sight — all from just trying to solve an everyday problem.

## Appendix [​](https://blog.danielh.cc/blog/sat#appendix)

You might have been wondering how the `is_valid` function worked. One way would be to implement it by checking every pair of queens to see if they lie on the same column or diagonal:

python

```
def is_valid(rows):
    for i in range(len(rows)):
        for j in range(i + 1, len(rows)):
            # Same column
            if rows[i] == rows[j]:
                return False
            # Same diagonal (the distance in rows equals the distance in columns)
            if abs(rows[i] - rows[j]) == j - i:
                return False
    return True
```

This would work, but it's actually quite inefficient! With the backtracking method, the only way for `rows` to contain multiple numbers is after the other previous rows have already been checked. So it really only needs to check on the most recently placed queen instead of all of them:

python

```
def is_valid(rows):
    for i in range(1, len(rows)):
        if rows[0] == rows[i]:
            return False
        if abs(rows[0] - rows[i]) == i:
            return False
    return True
```

There's another place for improvement. Looking back at the code with backtracking:

python

```
def solve(size, rows, amount_left):
    if amount_left == 0:
        if is_valid(rows):
            print(rows)
            exit()
    else:
        for r in range(size):
            if is_valid([r] + rows):
                solve(size, [r] + rows, amount_left - 1)

solve(4, [], 4)
```

The `[r] + rows` operation would take extra time, since it creates a copy of the entire list. To fix this, first change the order to `rows + [r]` and change `is_valid` to use the last value in list as the newly placed queen:

python

```
def is_valid(rows):
    for i in range(len(rows) - 1): 
        if rows[-1] == rows[i]: 
            return False
        if abs(rows[-1] - rows[i]) == len(rows) - i - 1: 
            return False
    return True

def solve(size, rows, amount_left):
    if amount_left == 0:
        if is_valid(rows):
            print(rows)
            exit()
    else:
        for r in range(size):
            if is_valid(rows + [r]): 
                solve(size, rows + [r], amount_left - 1) 

solve(4, [], 4)
```

Now, instead of copying the list, placing a new queen can reuse the existing list:

python

```
def solve(size, rows, amount_left):
    if amount_left == 0:
        if is_valid(rows):
            print(rows)
            exit()
    else:
        for r in range(size):
            rows.append(r) 
            if is_valid(rows):
                solve(size, rows, amount_left - 1)
            rows.pop() 

solve(4, [], 4)
```

There's just one final step to make this program as fast as possible:

rust

```
fn is_valid(rows: &[usize]) -> bool {
    for i in 0..(rows.len() - 1) {
        if rows[rows.len() - 1] == rows[i] {
            return false;
        }
        if rows[rows.len() - 1].abs_diff(rows[i]) == rows.len() - i - 1 {
            return false;
        }
    }
    true
}

fn solve(size: usize, rows: &mut Vec<usize>, amount_left: usize) {
    if amount_left == 0 {
        if is_valid(&rows) {
            dbg!(rows);
            std::process::exit(0);
        }
    } else {
        for r in 0..size {
            rows.push(r);
            if is_valid(rows) {
                solve(size, rows, amount_left - 1);
            }
            rows.pop().unwrap();
        }
    }
}

fn main() {
    solve(4, &mut vec![], 4);
}
```

And now we are done.