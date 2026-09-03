---
link: https://lalitm.com/post/changing-devtools-is-cheap-owning-them-isnt/
byline: Lalit Maganti
site: Lalit Maganti
date: 2026-08-09T15:53
excerpt: AI makes devtools cheap to customize, but personalized forks still
  demand ongoing attention and ownership.
slurped: 2026-08-24T08:27
title: Changing Devtools Is Cheap. Owning Them Isn’t.
---

In [Devtools must be open source](https://blog.exe.dev/devtools-must-be-open-source)[1](#fn:1) ([via](https://news.ycombinator.com/item?id=49156111)), David Crawshaw makes the case that, because of https://lalitm.com/post/github-alternatives/coding agents, we’re now in an era where devtools will be personalized by individual users. Specifically, agents’ ability to jump into new codebases and build whatever we want means we’ll be hacking on the source of the devtools we use day to day (even those without extension APIs) adding features and automatically rebasing our patches across releases.

The argument is seductive, especially to a reader who thinks of themselves as a maker or tinkerer: after all, the idea that you can hyper-tune everything you use sounds like a utopia; it means things can work _exactly_ how you want them to.

But I’d argue that Crawshaw underappreciates the ongoing cost when he writes

> “Both the upfront fixed costs and the ongoing costs of personalizing software have disappeared.”

While AI has made the upfront cost of _changing_ software a lot lower, properly personalizing software still requires your attention. And attention in the AI age is scarcer than ever.

Having maintained an open-source devtool designed to be modified and forked for nine years now[2](#fn:2), I can say that most users don’t _want_ to customize their devtools. They want someone else to make the tool reliable and coherent, so they can focus on the problems they opened it to solve. They reach for source modification only as a last resort, when a change is critical to their workflow and no other route works.[3](#fn:3)

This is not to say that this sort of personalization won’t become more common: I absolutely think it will. I just think it will take the form of strong core systems with well-defined boundaries and extension points.

### Personalization still needs a person[#](#personalization-still-needs-a-person)

As a thought experiment, imagine an open-source diff viewer with no extension API. You find most diffs noisy, so you ask an agent to add a “focus mode” that collapses imports, generated files, and other changes you consider mechanical. It works well and becomes part of your normal workflow.[4](#fn:4)

At first, life is good: everything works, and you’ve solved your problem. Then upstream releases a new version that refactors the code you changed. As Crawshaw suggests, you’re clever, so you’ve set up a bot to automatically rebase your changes onto each update. It resolves any merge conflicts and moves your code to the right place.

But now suppose a few months pass and upstream makes a more substantial change: it adds syntax-aware move detection. If a function moves between files, the viewer now shows it as a move instead of one large deletion and addition. The agent muddles through, rebases your focus-mode patch, and gets everything compiling without any merge conflicts.

But now what should focus mode do if the function has mostly moved but also contains a few meaningful edits? Does it hide the whole block as a mechanical move? Does it show only the edited lines without any surrounding context? Or does it show the whole function?

There isn’t an obviously correct answer; it depends on what _you_ want to see in the diff. So what, are you going to interrupt your day to make this decision?

There’s a central paradox here: if you’re okay with “let the agent decide”, then you’ve delegated your authority to the agent. For small choices, that may be perfectly adequate. But if you want the tool to work exactly how you want, you need to inspect and direct those choices. Do you really want to have opinions about the design of a devtool you use forever?

The key is _attention_. Any one personalized tool might be unlikely to fail on a given day, but if you do this to every devtool you use, you multiply the number of tools that can unexpectedly demand your attention.[5](#fn:5) Worse, those failures are unpredictable: a tool might work for months and then break at the exact moment you urgently need it. Most engineers want to use devtools to accomplish a task; they don’t want their attention diverted to designing and repairing them.

### Shared tools need a shared reality[#](#shared-tools-need-a-shared-reality)

All of the above applies to small teams as well. You can share the attention cost, but at the end of the day, the team still has to ask, “How much time do we want to spend on tools versus doing the actual work we’re meant to be doing?”

I also want to look beyond Crawshaw’s post and consider how this would work in larger companies: what happens when many teams independently personalize the same shared devtool?

I’ve seen this firsthand: another big tech company makes extensive use of Perfetto, and has hit this exact problem. Different teams in that company decided to fork Perfetto and add ad hoc changes for their local needs. Now one of the engineers there is fighting to consolidate them because of how painful it is when every team means something different by “Perfetto”.

Imagine the same pattern with a company-wide bug tracker. Do you want every team to use a version with subtly different meanings for status, priority, assignment, and resolution? What happens when a bug moves between teams? Different layouts and personal filters are harmless; the problem begins when personalization changes the shared semantics or workflow.

When a devtool mediates work between people, it also forms part of their common language. Teaching, auditing, reproducing investigations, and verifying that people are talking about the same thing all depend on a shared baseline.

### Upstream gets more malleable too[#](#upstream-gets-more-malleable-too)

We should also not compare pre-AI upstream development with post-AI forks. Maintainers can use the same agents to investigate reports, brainstorm ideas, and prototype new features. I can certainly attest to how useful AI has been for both implementing small feature requests from users and prototyping larger ones to determine feasibility.

In my opinion, upstream maintainers can, and should, spend the time saved on implementation making their tools more adaptable: implementing broadly useful features, adding configuration knobs where they make sense, and creating extension points for recurring needs. AI lowers the cost of doing all of this, including deciding where customization makes sense, adding more elaborate tests on creative uses of your tools and verifying backwards compatibility as these interfaces evolve.

Upstream has a natural advantage here: any work done there benefits _everyone_, while a change to your personal fork benefits only you. By relying on upstream, the attention required to build good software shifts from people who _don’t_ want to spend it to maintainers who have chosen to care.

### The building-block economy[#](#the-building-block-economy)

In my opinion, there’s an alternative view that is much more likely to come true, one described well in Mitchell Hashimoto’s article on the [building-block economy](https://mitchellh.com/writing/building-block-economy).

Concretely, it accepts the same premise: agents can write lots of code and build niche applications, tools, integrations, forks, and so on. But instead of assuming that forks will become the norm, Hashimoto argues that high-quality, well-documented building blocks will power this world.

I tend to agree: agents are very good at composing high-quality components. If maintainers provide those components alongside a focused application, makers can build specialized artifacts on top while accepting the costs. This model also creates an easy feedback loop for ideas to flow upstream because the product was designed to be extended.

I see signs that the world is already heading in this direction. For example, [`bb`](https://www.sawyerhood.com/blog/an-agentic-ide-that-builds-itself) is a very interesting agentic IDE that I’ve been playing around with recently. It has a very nice experience that lets users add substantial new product surfaces through self-modification. But the key is that those features are plugins built around a maintained core and extension system, not changes made by forking the project directly.

`bb` is also only a few weeks old at the time of writing, so we cannot draw any firm conclusions from it, but it’s an interesting sign of the future, IMO.

### Wrapping up[#](#wrapping-up)

I care deeply about both the world of devtools and open source, so this is something I feel very passionate about. Having been immersed in this world for almost a decade now, I think the future of well-built tools with thoughtful design and well-designed extension points is bright.

Sure, there will always be folks who want to fork and make ad hoc changes. These are the same people who already maintain custom builds of their window manager or terminal emulator, carrying a stack of patches to get everything exactly how they want it.[6](#fn:6) For them, the tinkering is part of the enjoyment and craft.

But I think most users just want to get their work done with devtools, and we owe it to them to give them a strong, dependable experience instead of asking them to take on the burden of maintaining the product themselves.