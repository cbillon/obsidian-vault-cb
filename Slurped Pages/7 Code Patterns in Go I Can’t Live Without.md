---
link: https://medium.com/better-programming/7-code-patterns-in-go-i-cant-live-without-f46f72f58c4b
byline: Henry Huang
site: Better Programming
date: 2021-09-19T08:23
excerpt: Code patterns in Go to make your programs more reliable and efficient
twitter: https://twitter.com/@Happyholic1203
slurped: 2025-11-09T18:12
title: 7 Code Patterns in Go I Can’t Live Without
---

## Code patterns to make your programs more reliable, efficient, and make your life easier

I’ve been developing EDR solutions for 7 years.

That means I have to write long-running system software that’s both resilient and efficient.

I heavily use Go for this job, and I’d like to share some of the most important code patterns that you can count on to make your program more reliable and efficient.

## Use Maps as a Set

We often need to check the existence of something. For example, we might want to check if a file path/URL/ID has been visited before. In these cases, we can use `map[string]struct{}`. For example:

Press enter or click to view image in full size

[Gist link](https://gist.github.com/Happyholic1203/6e5e2455e9db5b09770d4f3727af75cc#file-set-go)

Using an empty struct, `struct{}`, means we don’t want the value part of the map to take up any space. Sometimes people use `map[string]bool`, but benchmarks have shown that `map[string]struct{}` [perform better both in memory and time](https://itnext.io/set-in-go-map-bool-and-map-struct-performance-comparison-5315b4b107b).

It’s also worth mentioning that map operations are generally considered to have `O(1)` time complexity ([StackOverflow](https://stackoverflow.com/questions/29677670/what-is-the-big-o-performance-of-maps-in-golang)), but go runtime provides no such guarantee.