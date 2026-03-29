---
link: https://www.reco.ai/blog/we-rewrote-jsonata-with-ai
byline: ABOUT THE AUTHOR
excerpt: One engineer used AI to rewrite JSONata as a pure-Go library called
  gnata. Seven hours, $400 in tokens, 1,000x speedup, and $500K/year off our
  cloud bill.
slurped: 2026-03-28T09:46
title: We Rewrote JSONata with AI in a Day, Saved $500K/Year
---

A few weeks ago, Cloudflare published “[How we rebuilt Next.js with AI in one week.](https://blog.cloudflare.com/vinext/)” One engineer and an AI model reimplemented the Next.js API surface on Vite. Cost about $1,100 in tokens.

‍

The implementation details didn’t interest me that much (I don’t work on frontend frameworks), but the methodology did. They took the existing Next.js spec and test suite, then pointed AI at it and had it implement code until every test passed. Midway through reading, I realized we had the exact same problem - only in our case, it was with our JSON transformation pipeline.

‍

Long story short, we took the same approach and ran with it. The result is [gnata](https://github.com/RecoLabs/gnata) — a pure-Go implementation of JSONata 2.x. Seven hours, $400 in tokens, a 1,000x speedup on common expressions, and the start of a chain of optimizations that ended up saving us $500K/year.

‍

## An expensive language boundary

‍

At [Reco](https://www.reco.ai/), we have a policy engine that evaluates [JSONata](https://jsonata.org/) expressions against every message in our data pipeline - billions of events, on thousands of distinct expressions. JSONata is a query and transformation language for JSON (think jq with lambda functions), which makes it ideal for enabling our researchers to write detection rules without having to directly interact with the codebase.

‍

The reference implementation is JavaScript, whereas our pipeline is in Go. So for years we’ve been running a fleet of jsonata-js pods on Kubernetes - Node.js processes that our Go services call over RPC. That meant that for every event (and expression) we had to serialize, send over the network, evaluate, serialize the result, and finally send it back.

‍

This was costing us ~$300K/year in compute, and the number kept growing as more customers and detection rules were added. For example, one of our larger clusters had scaled out to well over 200 replicas just for JSONata expressions, which resulted in some unexpected Kubernetes troubles (like reaching IP allocation limits).

‍

In some respects, the RPC latency overhead was actually worse than the pure dollar cost. An RPC round-trip is ~150 microseconds before any evaluation even starts. For a simple field lookup like user.email = "admin@co.com" something that should take nanoseconds - we’re paying microseconds just for crossing a language boundary. At our scale, those microseconds stack up quickly.

‍

We’d tried a few things over the years - optimizing expressions, output caching, and even embedding V8 directly into Go (to avoid the network hop). They did their part, but it was mostly just incremental improvements. The closest we got was a local evaluator we built using [GJSON](https://github.com/tidwall/gjson) that handled simple expressions directly on raw bytes. It was fast for what it covered, but anything complex had to fall back to jsonata-js. We were patching around the problem, but the root cause remained unsolved.

‍

## Building gnata

‍

During the weekend I built out a plan (using AI) separated into ‘waves’. The approach was the same as Cloudflare’s vinext rewrite: port the official jsonata-js test suite to Go, then implement the evaluator until every test passes. The following day, I pressed play. The plan was straightforward - build out the full JSONata 2.x spec in Go, with a focus on performant streaming and some extra features sprinkled on (localized caching, WASM support, metrics, and fallthrough capabilities back to the jsonata-js RPC).

‍

A few iterations and some 7 hours later - 13,000 lines of Go with 1,778 passing test cases.

Total token cost: $400.

![Cursor AI dashboard for the gnata session](https://cdn.prod.website-files.com/644fc991ce69ff211edbeb95/69c3886f48e6486091300e85_blog-gnata-cursor-stats.jpg)

I shared the numbers internally and someone asked about the ROI. Production cost for jsonata-js in the previous month was about $25K - now it was 0. That conversation ended up being pretty short.

‍

## Two-tier evaluation

‍

gnata has a two-tier evaluation architecture. At compile time, each expression is analyzed and classified.

‍

The **fast path** handles simple expressions - field lookups, comparisons, and a set of 21 built-in functions applied to pure paths (things like $exists(a.b) or $lowercase(name)). These are evaluated directly against the raw JSON bytes without ever fully parsing the document. For something like account.status = "active" you get 0 heap allocations.

‍

Everything else goes through the **full path** - a complete parser and evaluator with full JSONata 2.x semantics. This does parse the JSON, but only the subtrees it actually needs, not the entire document.

‍

On top of this there’s a streaming layer (the StreamEvaluator) designed for our specific workload: evaluate N compiled expressions against each event, where events are structurally similar.

1. All field paths from all expressions are merged into a single scan. The number of expressions doesn’t matter - raw event bytes are only read once.
2. After warm-up, the hot path is lock-free. Evaluation plans are computed once per event schema and cached immutably, so reads are a single atomic load with no synchronization.
3. Memory is bounded. The cache has a configurable capacity and evicts the oldest entries when full.

The fast-path design took a lot of inspiration from the existing local evaluator. For simple expressions it was excellent — gnata won’t be faster on an apples-to-apples comparison for those. But the schema-aware caching and batch evaluation is where the real gains come from.

![](https://cdn.prod.website-files.com/644fc991ce69ff211edbeb95/69c38a1c16493e9cfd75333a_fomr.PNG)

Correctness: 1,778 test cases from the official jsonata-js test suite + 2,107 integration tests in the production wrapper.

‍

The speedup on simple lookups is mostly from eliminating the RPC overhead entirely — gnata evaluates directly on raw bytes with no JSON parsing. Complex expressions involve full parsing and AST evaluation, so the gap narrows, but they’re still 25-90x faster than the RPC path.

‍

In practice, gnata runs as a library inside our existing Go services. The serialization and RPC overhead goes away.

![Architecture: Before and After](https://cdn.prod.website-files.com/644fc991ce69ff211edbeb95/69c38a8c120b71c89f2f9f16_blog-gnata-architecture.jpg)

## Shadow mode and the week after

‍

Building the library was day one. The rest of the week was about making sure it was actually correct.

‍

We already had mismatch detection infrastructure in the codebase - feature flags, shadow evaluation, comparison logging - built months earlier for the local evaluator. Wiring gnata into the same system was straightforward.

‍

The rollout:

- Day 1: gnata built. PR opened.
- Days 2-6: Code review, QA against real production expressions, deployed to preprod in shadow mode. gnata evaluates everything, but jsonata-js results are still used. Mismatches logged and alerted. Fixed edge cases as they came up.
- Day 7: Three consecutive days of zero mismatches. gnata promoted to primary.

By the time we promoted gnata, it had already been processing billions of events and producing identical results to jsonata-js. We also caught bugs in jsonata-js itself. Cases where the reference implementation doesn’t follow its own spec. gnata handles them correctly.

‍

A side effect we didn’t expect: gnata was one of the first large PRs where we had AI agents reviewing AI-generated code. The agents were flagging everything - real concurrency issues alongside cosmetic nitpicks - and we had to teach them which is which. That work fed into how we handle AI code review more broadly now.

‍

## Beyond gnata - the next $200K

‍

Eliminating the RPC fleet took care of $300K, but there was one more thing we wanted to tackle - batching events end-to-end in our rule engine. JSONata - being only able to do a single evaluation at a time - forces the infra around it to contort itself with workarounds in order to stay performant. For our rule engine, that meant we were spinning up tens of thousands of goroutines to maximize concurrency (with all the added resources that entailed) in what would otherwise be a straightforward pipeline of micro-batches. As you might expect, that resulted in excessive memory and high CPU contention. In other words, our rule engine was both expensive and slow.

‍

gnata has no such limitations, so we were able to replace the rule engine internals with a far simpler and more efficient implementation. The details of the refactor deserve their own blog post, but they involved just-in-time batching (based on the request coalescing pattern), short-lived caches and grouped enrichment queries (done right before evaluation). I was quite surprised myself with the throughput increase the refactor brought - and with it, a sharp drop in resource utilization.

‍

The result: another ~$18K/month off the bill - around $200K/year. Combined with gnata, that’s **$500K/year gone from the pipeline, all in under 2 weeks of work.**

‍

## No longer just vibe coding

‍

There’s an active debate over whether fully hands-off AI code belongs in production. I have some strong opinions on this (which I’ve been vocal about internally). But gnata has been a good case study for when it works well.

‍

Andrej Karpathy recently wrote that [programming is becoming unrecognizable](https://x.com/karpathy/status/2026731645169185220), and that at the top tiers, deep technical expertise is “even more of a multiplier than before because of the added leverage.” Until recently, I was rather skeptical of agentic code. February 2026, however, has been a sort of inflection point even stubborn developers like myself can’t ignore.

‍

I believe gnata is just the beginning. I suspect 2026 will be the year of surgical refactors.

‍

## Try it

‍

```
go get github.com/recolabs/gnata

expr, _ := gnata.Compile(`user.role = "admin" and user.loginCount > 100`)

json := []byte(`{
  "user": {
    "email": "admin@example.com",
    "role": "admin",
    "loginCount": 247
  }
}`)

result, _ := expr.EvalBytes(ctx, json)
fmt.Println(result) // true
```

‍

Full docs, streaming API, and WASM playground: [github.com/RecoLabs/gnata](https://github.com/RecoLabs/gnata)