---
link: https://antonz.org/go-concurrency/#audience-and-approach
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Interactive book on concurrent programming with many exercises.
twitter: https://twitter.com/@ohmypy
slurped: 2025-11-11T10:07
title: "Gist of Go: Concurrency"
---

Many Go concurrency books and tutorials are like: here's goroutine, here's channel, here's select — use them as you like. Others just throw concurrent patterns at you without really explaining them.

This is not very helpful: the most important thing in concurrent programming is not pure knowledge, but the understanding and ability to apply concurrency primitives.

That's why I've created an interactive book that teaches Go concurrency from the ground up through practical exercises. These exercises are simple enough to be solved with a single page of code yet are closely aligned with real-world scenarios.

[Audience and approach](#audience-and-approach) • [Contents](#contents) • [About the author](#about-the-author)

## Audience and approach

The book is intended for programmers who are already familiar with Go, from language basics to interfaces and errors. You don't need any prior knowledge of goroutines and channels — we'll cover concurrency tools from the ground up.

Since the book targets experienced programmers, I've chosen a presentation style that I prefer: concise, focused, with exercises of moderate difficulty. Maybe you'll like it too.

Concurrent programming is a tricky beast, but I explain complex topics clearly, making them easier to understand. There are lots of examples and no dry theory.

There are dozens of interactive exercises that run directly in the browser. Each problem has an automated test suite that provides immediate feedback and a reference solution with an explanation.

There is no AI-generated or copied content. The book is 100% original.

## Contents

Part 1. Concurrency basics
[[Gist of Go - Goroutines]]
[[Gist of Go - Channels]]
[[Gist of Go - Pipelines]]
[[Gist of Go - Time]]
[[Gist of Go - Context]]

Part 2. Synchronization
[[Gist of Go - Wait groups]]
[[Gist of Go - Data races]]
[[Gist of Go - Race conditions]]
[[Gist of Go - Semaphores]]
[[Gist of Go - Signaling]]
[[Gist of Go - Atomics]]

Part 3. Other topics

- Testing
- Internals

You can read the chapters using the links in the table of contents above, or purchase the book for full access to the interactive exercises (+ a PDF version).

The book is a work in progress (11/13 chapters finished).

Last updated: 2025-09-27

Go version: 1.25

[Pre-order for $10](https://antonz.gumroad.com/l/go-concurrency?wanted=true)   or [read online](app://obsidian.md/goroutines/)

![](app://obsidian.md/assets/antonz.jpg)

I'm Anton Zhiyanov. I work on open source and write interactive technical books.

In 2022 I launched a course on Go concurrency. It now has 250 graduates and an average rating of 5 stars based on 100 student reviews.

This book is based on the original course.

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.