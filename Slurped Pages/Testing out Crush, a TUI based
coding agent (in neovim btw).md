---
link: https://grahamhelton.com/blog/crushing-it
site: Graham Helton
excerpt: |-
  Using Charm's new AI coding
  agent to build an open graph image generator for this site.
twitter: https://twitter.com/@grahamhelton3
slurped: 2025-12-17T09:36
title: |-
  Testing out Crush, a TUI based
  coding agent (in neovim btw)
---

[Charm](https://github.com/charmbracelet) released an AI coding agent that works in your terminal called [Crush](https://github.com/charmbracelet/crush). I was eager to give it a try as I dread using the experience of using Cursor’s vscode-centric UI.

  
I quite literally do not understand charm’s business model.

![](https://grahamhelton.com/assets/images/Pasted%20image%2020250802185531.png)

One of the items on my `todo.txt` for this site implementing [opengraph](https://ogp.me/) images metadata. This metadata is relatively straightforward to add by adding HTML properties such as: `meta property="og:title" content ="title_here"/>`.

Unchanging static images are also a simple tag: `<meta property="og:image" content="path/to/png" />`. The less straightforward part is dynamically generating an image per page.

This is a relatively good candidate for experimenting with an AI tool such as Crush as:

1. I’ve done this before using tools like Hugo so I know some of the pitfalls
2. Its low risk. If it breaks it’s not a huge deal.
3. I already have all the metadata needed stored in the yaml frontmatter of each post. For example, each markdown file I write starts with something like:

```
---
title: "a working KVM solution"
date: 2025-08-01
tags: ["office", "productivity"]
description: "The KVM solution I landed on that (for once) isn't terrible"
type: note
<snip>
```

A few months ago, inspired by [this wonderful site](https://usgraphics.com/), I made a mock up of an announcement document in the style of a old school press release for Low Orbit Security. I used that as a base for what the og-image should look like.

![Press release mock up](https://grahamhelton.com/assets/images/Pasted%20image%2020250803160311.png)

The tool I ended up building is a bit hacky, but works well. My goal was to build a tool that would:

1. Generate an HTML page using the same CSS as this site.
2. Format it using an old school press release inspired format.
3. Convert the page to a png
4. Save it as a png.

The output ended up looking like this:

![og-image for a post tagged with as “redteam”](https://grahamhelton.com/assets/images/Pasted%20image%2020250803161042.png)

I’m familiar and generally a fan of Charm so it’s no surprise that Crush is pretty much exactly what I expected, a TUI coding agent that has the charm look and feel. I was able to finish the feature relatively quickly and integrate it into the static site generator build script with minimal hiccups. It looks and feels familiar if you’ve used Claude Code, but just with the familiar charm polish.

The first real task I used Crush for was to edit my neovim config to open up a Cursor style side pane using `<leader>ai`. This worked ok-ish (using gemini flash) but it allowed me to understand how to navigate using Crush which was relatively intuitive. There were some visual bugs associated with this approach, especially once I started using crush from within a neovim window.

Having a dedicated pane was useful so I could either keep an eye on things it was doing (allowing or denying changes as necessary) or being able to quickly close it when I was working on something manually.

![lazyvim with crush in a separate pane](https://grahamhelton.com/assets/images/Pasted%20image%2020250802185609.png)

The diff UI was a bit jarring at first (possibly because of the [tokyodark](https://github.com/tiagovla/tokyodark.nvim) colorscheme) but if you’re sticking to prompts that making fairly precise edits it’s not bad. I generally think it’s hard to read git-diffs anywhere if they’re more than a few dozen lines.

![Crush requesting to make an edit](https://grahamhelton.com/assets/images/Pasted%20image%2020250802190028.png)

I particularly enjoyed Crush keeping a list of changed files and, most importantly for reasons I discuss below, the model/cost.

![Note the model and associated cost](https://grahamhelton.com/assets/images/Pasted%20image%2020250802190051.png)

The `ctrl-p` options are also a really nice addition. Specifically the quick switching between models and the summary option were ones I’d use regularly.

![Crush Model selection menu](https://grahamhelton.com/assets/images/Pasted%20image%2020250802191054.png)

![Crush Summarizing a session](https://grahamhelton.com/assets/images/Pasted%20image%2020250802190542.png)

## Thoughts

Besides the TUI UX, the most important part of Crush is it’s model agnostic approach, allowing you to pick and choose which models you’re ~~sending your data to~~ using. I would love to test this out using models hosted locally, something not possible using tools like Claude Code.

Despite enjoying the experience for writing code, I won’t be using it to do so until I have a ~~datacenter~~ selfhosted GPU setup simply due to cost. To fully implement this relatively straightforward feature it cost $23.04 using mostly Sonnet 4 (not the thinking model) for the heavy lifting and Gemini flash for quick edits and doc updates.

This took me about 45 minutes to implement whereas it would have taken me a few hours over a couple of days to implement from scratch. A nice boost in efficiency, but the point of this site is personal enjoyment not absolute efficiency.

  
If it were about efficiency I would have just made a hugo theme…

When it comes to pricing companies like Cursor have two distinct advantages:

1. **Optimizations**: The Cursor team has clearly put a massive amount of time and effort into optimizing what is sent to the model for processing via complex caching mechanism
2. **Economies of scale**: The economies of scale advantage companies like Cursor are able to capitalize on to ensure incredibly low API pricing on premium models by partnering with companies like Anthropic is borderline monopolistic. For about the same price I paid in API fees to add this single og-image feature using a tool I prefer, I could instead get pay slightly less per month to get an absurd amount of prompts using larger models like `sonnet-4-thinking` if I simply use cursor.

From looking at Cursor output you can see I racked up $50.13 in API fees but the cost is still $0.

![Cursor cost dashboard](https://grahamhelton.com/assets/images/Pasted%20image%2020250802192440.png)

With that being said, I will still be experimenting with Crush for smaller tasks that don’t require intensive (read: expensive) models. Using smaller models to answer questions about a fairly simple codebase or make quick changes to a homelab is a great use case for me.