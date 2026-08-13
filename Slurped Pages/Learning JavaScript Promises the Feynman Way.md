---
link: https://www.jakeworth.com/posts/learning-javascript-promises-the-feynman-way/
byline: Jake Worth
site: Jake Worth
date: 2025-08-25T18:08
excerpt: Want to learn a tricky topic and sharpen your learning skills at the same time? In this post, I&rsquo;ll use the Feynman Learning Technique— a method of learning complex things by explaining them simply— with a sprinkle of LLM magic, to deepen my understanding of JavaScript promises.
tags:
  - javascript
slurped: 2026-06-28T15:29
title: Learning JavaScript Promises the Feynman Way
---

Want to learn a tricky topic and sharpen your learning skills at the same time? In this post, I’ll use the Feynman Learning Technique— a method of learning complex things by explaining them simply— with a sprinkle of LLM magic, to deepen my understanding of JavaScript promises.

### What is the Feynman Learning Technique?

The Feynman Learning Technique is a learning method defined by Nobel laureate physicist and bongo drummer Richard Feynman. My perception is that it’s been pieced together from things that Dr. Feynman said over time.

Feynman’s technique comprises four steps:

1. Select a concept and map your knowledge
2. Teach it to someone who understands it less than you
3. Review and refine
4. Test and archive

In part one, we start with a blank page and write or draw everything we know about the subject, without editing.

In part two, we think about explaining it to someone who understands it less than us, such as a 12-year-old. If you can’t explain it to someone with no context, there’s room to expand your understanding.

In part three, we review and refine, finding weak spots in our understanding, concepts we’re regurgitating, and areas where we got stuck. We note these and dive deeper into them.

In part four, we test our understanding by teaching it to someone, with no notes. Then, we archive it for future review.

A disclaimer: I’m going to say some things about promises in this post that are wrong! That’s the point. By exposing my ignorance, I hope to better understand an important concept. Also, I promise to lightly edit what I write.

### Feynman Learning Technique Part One: Mapping My Knowledge

Into the mind-mapping! I’m going to write about this for twenty minutes, which will be more time than needed. Here we go!

---

🗺️ **My Mind Map 8/24/25: JavaScript Promises**

Promises are part of the ECMA JavaScript specification, so they’re a standard to aspire to, not a feature of a single software product. They aren’t new, perhaps a decade old, so while not always understood by engineers, they aren’t cutting-edge technology.

Promises let us do asynchronous things in JavaScript, because JavaScript, unlike many popular scripting languages, is single-threaded.

What does this mean? Consider this code:

```
const user = api.getUser(userId);

if(user) {
  // do something 
}
```

If you’re more familiar with multithreaded languages like Ruby, your mental model might be that each of these lines is run “in order”, top to bottom, and `user` is the result of our network request. In JavaScript, this isn’t true!

Let’s assume that `api.getUser` is asynchronous– it makes a network request, the API receives it and retrieves something from a database, returns it, and we process that information. If we don’t wait for `api.getUser`, `user` is not going to be what we think, an object of data, but rather, a Promise. Our `if` statement is going to have some problems, and in most contexts, we never get a second chance to see the received value.

JavaScript can’t wait for something asynchronous— it proceeds to the next line. This is why promises exist.

There are two promise syntaxes: `async` / `await` and `.then`. Here are both versions:

```
const onLoad = async (userId) => {
  const user = await api.getUser(userId);
  if (user) showModal(`Welcome back, ${user.name}`);
};

const onLoad = (userId) => {
  return api.getUser(userId).then(user => {
    if (user) showModal(`Welcome back, ${user.name}`);
  });
};
```

These two functions behave identically. This is an area where a lot of engineers get stuck when preferring one style over the other. Some feel that `async` / `await` is more readable, a subjective argument I happen to disagree with. Others find it more modern. You don’t want to use both in the same function, or the same code file, or even the same codebase. Be consistent and boring.

There’s a constructor: `new Promise`, which lets you define a promise and what happens when it succeeds, “resolves”, or fails, “rejects”.

Promises can be chained when you want them to happen in a sequential order.

```
const onLoad = (userId) =>
  api.getUser(userId)
    .then(u => getPosts(u.id))
    .then(posts => showModal(`You have made ${posts.length} posts`));
```

Like many collections, the Promise API has some cool functions like `Promise.all()`, which waits until all the promises passed to it complete, or `Promise.any()`, which waits for any one to complete, before moving on.

Promises trip some engineers up. There’s an industry of OSS libraries like Tanstack Query that add a layer of abstraction to asynchronous data fetching, because rolling yourself isn’t easy and gets repetitive fast.

---

### Feynman Learning Technique Part 2: Teaching it to Someone

Here’s how I’d explain this to someone who understands it less than me:

> “JavaScript is an important programming language. It has a quirk, though: it can’t wait for things. So, we invented a tool called Promises that lets JavaScript code wait.”

### Feynman Learning Technique Part 3: Review and Refine

Here are a few points from my summary that I’d like to consider and refine. My sources are Wikipedia and MDN.

> Me: “Promises are part of the ECMA JavaScript specification…”

The term ‘promise’ was proposed in 1976 by Daniel P. Friedman and David Wise. It’s been in JavaScript as of ECMAScript 2015, and via the keywords `async` and `await` since ECMAScript 2017.

> Me: “Promises let us do asynchronous things in JavaScript…”

This is directionally correct. I think that JavaScript’s non-blocking concurrency model better describes this behavior.

> Me: “What does this mean?”

Pretty accurate? I’m not sure presenting it from the perspective of a Rubyist is useful, but it helped me.

> Me: “These two functions behave exactly the same…”

I’m on my soapbox here. And, trying to demystify promises for my fellow engineers.

> Me: “There’s a constructor…”

This is missing a lot of what makes Promises great. Constructing, resolving, and rejecting are things that I haven’t done a lot. Functions like `.all` are cool, but I’ve seen them overused and so I’m wary about them.

> Me: “Promises trip some engineers up.”

They do! I think that abstracting them is good for the world.

### Feynman Learning Technique Part 3 Bonus: LLM Review

As an experiment, I gave my summary to Chat-GPT 5 and asked it to identify incorrect and incomplete points. Here’s what it said:

---

🗺️ **Chat-GPT 5 Evaluates My Mind Map**

- **Promises don’t make JS “able to do async.”** Async I/O comes from the host (DOM/Fetch/Node). Promises are a _representation_ of an eventual result and a scheduling mechanism (microtasks), not threads.
    
- **Single-threaded nuance.** The main JS thread is single-threaded, but the platform can use background threads (Web Workers, Node’s libuv threadpool). Promises give ordered callbacks on the main event loop.
    
- Mixing `await` and `.then` in one function is _allowed_; prefer one style per block for readability, but it’s not a hard rule.
    
- **Error handling is missing.** Use `try/catch` around `await`, or `.catch(...)`. Remember `async` functions return a Promise and rejected Promises propagate unless handled.
    
- **Promise constructor caveat.** `new Promise((resolve, reject) => { ... })` runs the executor **synchronously**. You rarely need it; avoid the “Promise constructor anti-pattern” (wrapping an API that already returns a Promise).
    
- **Parallel vs sequential.** `await`ing in a row is sequential. For independent work, kick them off first, then `await` together:
    
    ```
    const [user, posts] = await Promise.all([getUser(id), getPosts(id)]);
    ```
    
- **Microtasks matter.** `.then`/`catch`/`finally` callbacks run in the **microtask** queue after the current call stack, before macrotasks like `setTimeout`.
    
- **Top-level await.** In ES modules you can `await` at the top level—worth knowing for modern apps/build tools.
    
- **Libraries (e.g., TanStack Query) do more than “wrap Promises.”** They add caching, deduping, background refetch, retries, pagination, and synchronization with UI lifecycles—useful ergonomics beyond basic Promise flow.
    
- **Style guidance tweak.** Consistency is good, but it’s fine for a codebase to use both styles where appropriate (e.g., `.then` for small pipelines, `await` in async functions). The “never in the same file” stance is too strict.
    

---

Chat-GPT, coming in hot! I’m glad I asked for the LLM review. And, I’m also glad that I did my own first. As Dr. Feynman once said: “The first principle is that you must not fool yourself and you are the easiest person to fool.” It’s vital to develop your own internal BS detector.

### Feynman Learning Technique Part 4: Test and Archive

To test my understanding, I’ll be doing a lightning talk on Promises static methods (the `.all` and `.any` functions I mentioned) at the next [Maine JS Meetup](https://www.meetup.com/mainejs/events/307703166/) on September 9, 2025, in Portland, Maine. We’ll see what I learned and how I can communicate it.

Once I’m finished, I hope to review this post and my talk in the future and see how my concept of Promises has evolved.

### Conclusion

Here are the four steps of the Feynman Learning Technique once again.

1. Select a concept and map your knowledge
2. Teach it someone who understands it less than you
3. Review and refine
4. Test and archive

How could you apply this to something you’d like to understand better? Let me know if you try.