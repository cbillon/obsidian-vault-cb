---
link: https://gagor.pro/2025/11/create-markdown-presentations-with-marp/
byline: Tom
site: Tom's Blog
date: 2025-11-06T01:00
excerpt: Learn how to create beautiful slide decks with Markdown and Marp, a simple and powerful tool for presentations.
tags:
  - markdown
slurped: 2026-02-24T10:36
title: Create Markdown presentations with Marp
---

I’m a big fan of Markdown. I use it on this blog, in technical documentation, and pretty much whenever I can. I love the simplicity of writing without the need for complex, excessive syntax like XML or HTML. Markdown is also my go-to format when working with AI agents like ChatGPT.

One thing I always missed was the ability to create notes and present them effectively. Sure, I could show formatted text generated from Markdown, but those presentations never looked great. Without proper slide separation, people could see what was coming next, which ruined the top-down flow of a presentation.

It happened again recently. I had my notes and a plan for a presentation. I started crafting slides in PowerPoint. I found the company’s template, and 15 minutes later, I had the first and last slides ready. 🤣

Choosing from different themes, colors, and layouts can be overwhelming when all I need is a few phrases to guide me and something for the audience to focus on. I already have those in my notes, but moving them into a presentation is tedious. Since I have to do this occasionally, I wanted to simplify the process.

That’s when I found Marp[1](#fn:1). It’s open-source, works on Mac and Linux (my main platforms), and is super easy to use.

If you have notes on a specific topic, you can ask any AI agent to summarize them in Markdown. With headers and bullet points, your slides are already halfway done.

You start with a `frontmatter` like this:

```
---
marp: true
theme: default
---

<!-- Then add the first slide: -->

# **My Title**

*Title of slide*

Author

---

# Another title
...
```

It uses “line separators” (`---`) to divide slides. The rest is pure Markdown, so you can use the usual syntax.

Now, how do you make this convenient? Not being able to see the slides as you write wouldn’t make the process much easier. I usually code in VS Code, and [there’s a plugin](https://marketplace.visualstudio.com/items?itemName=marp-team.marp-vscode) that works well with Markdown Preview. Just hit `ctrl/cmd` + `shift` + `m`, and you can see your slides update in real time as you write, edit, or rearrange content.

For me, this is the most natural way to work-with text, like with code. I can easily keep it in a code repository, revert changes, compare versions, and so on.

## Hacks

Of course, there are [well-documented features](https://marpit.marp.app/markdown) , but some things require a bit of time to figure out.

### Multi-column split

The first hack is a multi-column layout. By default, there’s none, but you can easily create one with a bit of [HTML and CSS](https://github.com/orgs/marp-team/discussions/192#discussioncomment-1516155) .

Example of multi-column design

```
---
marp: true
theme: default
footer: Tomasz Gągor | gagor.pro
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
---

# **This is a multi-column slide deck**

<div class="columns">
<div>

* First column point 1
* Point 2

</div>
<div>

Second column

![Amazing chart](../images/cover.webp)

</div>
</div>
```

Here’s how it looks:

![Example multi-column slide deck](app://obsidian.md/slides/multi-column.webp)

### Convert PDF to images with proper page numbers

A final presentation in PDF format is usually good enough, but if I want to add it to my blog, I prefer to use images (PNG/Webp). This can be done with ImageMagick.

I have two additional requirements:

- Double-digit numbers (prefixed with zero),
- Starting with `1` as the first slide.

All of this is possible with one command:

Converting PDF slide decks to PNG images

```
convert slide-decks.pdf -scene 1 slide-%02d.png
```

  

### Inverting the color scheme of slide decks

This simple trick can “intensify” the impact when switching slide decks. Just invert the color scheme for the first, last, or any other important slide to visually “wake up” and grab the audience’s attention.

It’s as simple as adding this comment:

Inverting color scheme for a single slide

Check out these examples for the result:

[![Examples of slide decks with inverted colors](app://obsidian.md/2025/11/create-markdown-presentations-with-marp/slides/invert-1_hu_a7fa985244fa7a4.png)](app://obsidian.md/2025/11/create-markdown-presentations-with-marp/slides/invert-1.png "Examples of slide decks with inverted colors")

[![Examples of slide decks with inverted colors](app://obsidian.md/2025/11/create-markdown-presentations-with-marp/slides/invert-2_hu_48bef4619850bfc3.png)](app://obsidian.md/2025/11/create-markdown-presentations-with-marp/slides/invert-2.png "Examples of slide decks with inverted colors")
