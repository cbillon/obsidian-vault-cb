---
link: https://jrsinclair.com/articles/2019/what-i-wish-someone-had-explained-about-functional-programming/
byline: Written by James Sinclair  28th October 2019
excerpt: It’s hard learning functional programming on your own. But it ought not
  to be. You don’t need a PhD to understand functional programming. The concepts
  are abstract, yes. But that doesn’t make them incomprehensible. It shouldn’t
  be this difficult. This is the first in a four-part series on things I wish
  someone had explained to me about functional programming.
slurped: 2024-12-14T09:04
title: Things I wish someone had explained about functional programming
---

It’s hard learning functional programming on your own. At least, that was my experience. And it ought not to be. You don’t need a PhD to understand functional programming. And I _have_ a PhD. Yet I still struggled. The concepts are abstract, yes. But that doesn’t make them incomprehensible. It shouldn’t be this difficult.

I found it difficult to learn functional programming for two reasons:

1. I made some faulty assumptions about the learning process; and
2. I lacked someone to explain the broader concepts and how they fit together.

In this article, I want to talk briefly about the faulty assumptions. That is, the unconscious biases and expectations we bring to learning functional programming. We all have them, no matter what background we come from.

In subsequent articles, I want to talk about some broad concepts. That is, some ‘big picture’ ideas from functional programming. I felt stuck for a long time because I didn’t have a clear idea of how the pieces fit together. I didn’t know the correct terminology. And without that, my web searches proved fruitless.

But first, assumptions.

## Wrong assumptions

When I started, I thought this would be like learning another programming language. I’d learned new languages before. You spend most of the time learning new syntax. And that’s easy. It’s a slight variation on existing concepts: loops, variables, conditionals, and so on. Then you learn a handful of new concepts baked into the idioms of the language. And usually, this is where the most value lies. The new concepts add to your problem-solving tool kit, and you can apply them back to other languages. It takes time, but it’s not all that difficult. And there’s real value there. I assumed that learning functional programming would be the same. It’s not.

Learning functional programming is different. It’s more than some extra concepts that slot in with what you already know. No, functional programming is a whole new way to think about programming. If you come from an OOP or imperative background, it turns a lot of received wisdom on its head. Things that you thought were bad ideas turn out to be good ideas. Things that you thought were convenient and clever turn out to be problematic. Or plain impossible.

At first, functional programming feels like a lot of restrictions:

- No loops;
- No mutating data;
- No impure functions.

Tools we formerly wielded with skill and dexterity no longer work. So we learn new tools, like recursion and list comprehensions and composition. And these solve most immediate problems. It’s even intoxicating at first. It’s fun to write concise, powerful code. It’s exciting to condense entire algorithms down to a couple of lines of Haskell. There is a reason why functional programmers are so enthusiastic about it.

But it’s not long before things get complicated. We start with a bunch of simple functions. Easy. But the types don’t all line up. And we need to generate some side effects too. And handle errors. And manage state. And how do you debug this? It gets tricky, quick.

Functional programming has idioms and tools to solve all these challenges. But learning them is hard if nobody shows you where to look. Sometimes, solving a simple problem requires learning a complicated concept. And if you come from an imperative background, it also requires unlearning a lot of habits. These habits might have afforded you a lucrative career as a software developer. But suddenly, you’re a beginner again. You’re fumbling around with ‘basic’ stuff. So the process is not just difficult, it’s humbling.

That’s okay though. It’s good to be humbled once in a while. And functional programming is fascinating _because_ it’s so different. There’s so much to learn. So many ways to write more stable, reliable code. So many elegant abstractions that turn spaghetti code into poetry. It’s fun! It’s worth the effort to learn functional programming.

I do want to warn people though. It can be a hard road. I know nothing I say will prevent people making faulty assumptions. We all do it, whether we intend to or not. But I still want to call this out. Perhaps it might make someone feel better when they realise it’s not just them.

So, let’s assume you’re an experienced developer attempting to learn functional programming. Here are my recommendations:

1. Accept that it will be hard. And it’s not because you are stupid. It’s because functional programming is really different to what you might be used to. It can be difficult to wrap your head around. But that’s the fun of it. If you can, try to enjoy the challenge.
2. Recognise that the shallows are nice, but the waters get deep quickly. The fundamentals of functional programming are really not complicated. Avoiding side effects, immutable data, and composition covers most of it. But all the consequences and implications of that get complex, fast. So anticipate that sometimes, solving a ‘simple’ problem will involve learning complex things.
3. Get in-person help, if you can. Online learning materials for functional programming are good, but they could be better. I find there’s a lot of black holes and dead ends. I’ll explain why in later articles. But this means that the fastest way to learn is to get a person to point you in the right direction. I recognise that this isn’t possible for everyone. But if you can, do try.

As I mentioned, online learning materials for functional programming could be better. In particular, I think we can do a better job of working from the concrete to the abstract. I’d like to see more resources that start with solving particular problems. And then, after demonstrating a few examples, work up to common patterns. Current resources tend to do it the other way around. They start with the common patterns first. I’ve attempted to avoid that in my writing. But, that said, at some point we also need to show how things fit together—give people the big picture. That’s what I wish someone had explained for me.

## What I wish someone had explained

When I was learning, three related concepts confused me. I tried to figure them out on my own. But, because I didn’t understand what I didn’t know, I’d end up searching for the wrong thing. The three concepts were:

1. [Algebraic structures](https://jrsinclair.com/articles/2019/algebraic-structures-what-i-wish-someone-had-explained-about-functional-programming/);
2. [Type classes](https://jrsinclair.com/articles/2019/type-classes-what-i-wish-someone-had-explained-about-functional-programming/); and
3. [Algebraic data types](https://jrsinclair.com/articles/2019/algebraic-data-types-what-i-wish-someone-had-explained-about-functional-programming/).

At different times, I thought all three were the same thing. But each idea is different. And some of the differences are subtle. I’ll try to explain each one in turn—one article each (and will link them up here as I go). If you’re an expert, forgive me if my understanding isn’t 100% accurate. I’m doing my best.

---

Enormous thanks to [Jethro Larson](https://twitter.com/jethrolarson), [Joel McCracken](https://twitter.com/joelmccracken) and [Kurt Milam](https://twitter.com/kurtmilam) for reviewing an earlier draft of this entire series. I really appreciate the feedback and suggestions.