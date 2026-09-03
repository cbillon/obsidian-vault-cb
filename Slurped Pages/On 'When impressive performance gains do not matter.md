---
link: https://lalitm.com/post/on-when-impressive-performance/
byline: Lalit Maganti
site: Lalit Maganti
date: 2026-06-30T17:25
excerpt: When impressive performance gains do not matter is a very nice article
  covering some ways in which going after performance alone is not sufficient
  without considering the wider picture. It resonated a …
slurped: 2026-08-24T08:48
title: On "When impressive performance gains do not matter"
---

[When impressive performance gains do not matter](https://blog.colinbreck.com/when-impressive-performance-gains-do-not-matter/) is a very nice article covering some ways in which going after performance alone is not sufficient without considering the wider picture. It resonated a lot with how I think about performance.

> If there are multiple bottlenecks in the pipeline—and with these systems, this is common—the overall throughput will not improve until every last bottleneck is removed.

His focus is on distributed systems bottlenecks, but I’ve hit the same “do-nothing” speedups when optimizing client side programs. Usually this comes from spending a lot of time _thinking_ something was the bottleneck when it wasn’t. CPU profiling is where this bites me most: it tells me “function X is taking 30% of the cycles” and I think “oooo, there’s a lot of gains to be made there”. I build a microbenchmark for X, optimize it and there’s only a marginal gain at the high level. While disappointing, I’ve become used to it over time and internalized that performance is highly non-linear and actually knowing where the problem lies is really hard.

One specific pattern I’ve run into several times: if you are processing a lot of data and that requires you to hit main memory, you can do quite a bit of computation at the same time on that data for free. This is because out-of-order processors are very good at doing computation while stalled on RAM. So removing instructions in paths like that won’t actually matter; what matters much more is your data structures and reducing how much memory you’re touching.

While I’m on this topic, I have to shout out Matt Kulukundis’s [CppCon talk on Abseil’s flat hash map](https://www.youtube.com/watch?v=ncHmEUmJZf4), as it’s what helped me distill my experiences into actual rules of thumb around building performant systems. I would highly recommend you watch it if you are remotely interested in performance!

As for the post, I also like the closing reflection:

> Performance work can be incredibly challenging, but it is also a discipline for intimately understanding complex systems and engineering better products.

That matches my own experience: the lasting value isn’t any single speedup, it’s the improved understanding of the system you’re building and is invaluable for making things better in the long term.