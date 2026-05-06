---
link: https://grahamhelton.com/blog/obsidian-v2
site: Graham Helton
excerpt: |-
  How I structure five Obsidian
  vaults, enforce atomic notes through templated creation flows, and use
  frontmatter properties to turn notes into queryable databases.
twitter: https://twitter.com/@grahamhelton3
slurped: 2026-05-04T08:06
title: |-
  Human Memory Management And
  Obsidian V2
tags:
  - obsidian
---

In October 2023 I published [Human Memory Management: Techniques For Actionable Security Research](https://grahamhelton.com/blog/human-memory-management), a blog where I detailed exactly how I used Obsidian. It’s not my most-read post, but it’s the one I get the most questions about. Readers who spent time reading through it often wanted to replicate the system.

My obsidian workflow has changed dramatically over the years, and as such, an update is in order. Most notably, agents have become genuinely useful, which forced me to rethink how the vault is structured. and I’ve spent a lot of time brainstorming how my vault structure should evolve.

## Vaults

The traditional obsidian ethos encourages a single vault structure so that `[[wikilinks]]` can be used to navigate a vault structure and make connections. I am wary of this approach and silo different domains of my life into different vaults. The _brain_ vault I use for planning a vacation should never need to be linked to the _security_ vault I use for offensive security research.

I currently maintain 5 vaults:

- **grahamhelton.com**: The vault for publishing content to this website
- **security**: The _main_ obsidian vault I work in. This contains all the notes and knowledge I’ve collected about cybersecurity and my career.
- **brain**: This is my personal vault I use for journaling, grocery lists, and ideas
- **jia van**: This is the experimental vault I created for my [jia van](https://grahamhelton.com/blog/jia-van) project.
- **testing**: This is the _staging_ vault where I test new plugins, ideas, and workflows

This post will mostly describe the main _security_ vault since it’s the most heavily customized one.

## Plugins

Historically, I didn’t use many plugins, but AI tools like Claude Code have lowered the time it takes to build custom ones, so I now use several hyper-specific ones throughout my vaults. The `grahamhelton.com` vault has one that turns Obsidian into a half-CMS, half-CI pipeline for publishing blog posts.

![Custom blog overview](https://grahamhelton.com/assets/images/Pasted%20image%2020260415103639-800.png)

Custom blog overview

![Custom obsidian publishing actions pipline](https://grahamhelton.com/assets/images/Pasted%20image%2020260415103735-800.png)

Custom obsidian publishing actions pipline

I also use a custom Google Docs-inspired commenting plugin that I created for the sole purpose of allowing AI models to have an interface to act as _editor_ of my writing, not a _ghostwriter_. The ethos is: build hyper-custom tooling as needed.

**Note**  
I’m also somewhat paranoid about obsidian plugins getting [Jia Tan](https://en.wikipedia.org/wiki/XZ_Utils_backdoor)’d so reducing external code usage where I can somewhat helps with that.

0:000:00

Plugin for commenting and automated editor review

Additionally, I’ve borrowed some features from neovim such as a `leader` key (mapped to `,`) and [telescope](https://github.com/nvim-telescope/telescope.nvim) style fuzzy finder where `,ff` will search through note titles and `,sg` will search through file content.

![Searching for files with bgp in the title](https://grahamhelton.com/assets/images/Pasted%20image%2020260415121653-800.png)

Searching for files with _bgp_ in the title

![Searching for bgp within a file’s content](https://grahamhelton.com/assets/images/Pasted%20image%2020260415122012-800.png)

Searching for _bgp_ within a file’s content

## Attachments

Dealing with attachments is annoying in most software. Obsidian is only _slightly_ less annoying because I’ve opted out of dealing with them entirely. The first setting I change when I first create a vault is the _default location for new attachments_ to _In the folder specified below_ and then set the _attachment folder path_ to `_obsidian/attachments`.

![Attachment folder path](https://grahamhelton.com/assets/images/Pasted%20image%2020260415113327-800.png)

Attachment folder path

When any new attachment gets added to the vault, it ends up in that folder. Obsidian really _just works_ at pulling any images, videos, or pdfs from that folder. The only time I have to think about attachments is when sharing files, but I typically send a PDF anyway.

## Folders

Folder structure is a controversial topic in the personal knowledge management world. I’ve tried all of the methods like [PARA](https://fortelabs.com/blog/para/) and [Zettelkasten](https://obsidian.rocks/getting-started-with-zettelkasten-in-obsidian/) and none of them worked well for me. I’ll discuss how I use folders in more detail in a moment, but my vaults all contain a `_obsidian` folder that houses _metadata_ about the vault such as attachments, templates, etc.

![My “metadata” folder](https://grahamhelton.com/assets/images/Pasted%20image%2020260415114158-800.png)

My “metadata” folder

**Note**  
This is different than the hidden `.obsidian` folder created upon vault initialization that houses vault configuration settings like appearance and plugin settings.

## The Security Vault

Here is the canonical and mostly useless graph view picture to convince you I’m a _real_ note taker with _real_ insights into how to do _real_ knowledge management.

![A fancy and useless graph](https://grahamhelton.com/assets/images/Pasted%20image%2020260415115518-800.png)

A fancy and useless graph

My _security_ vault has evolved since the original [Human Memory Management](https://grahamhelton.com/blog/human-memory-management) post to the point of being unrecognizable. So here’s how the vault works today.

The _security_ vault is organized into 8(ish) top-level directories.

- __obsidian_: Metadata about the vault, see above for more details.
- _1_TECHNOLOGY_: Any notes about a specific technology. IE: `kubernetes`
- _2_TECHNIQUE_: Any notes on a technique that can be executed upon a technology. IE: `BGP`
- _3_RESEARCH_: This is a catch all for any notes that are things I’m looking into or want to dig into more in the future. Often times I’ll encounter something when working and say “thats weird” and create a new research note with screenshots or output I can use to pick back up on in the future. I honestly don’t use this as often anymore and typically throw these breadcrumbs of research ideas into [Linear](https://linear.app/)
- _4_TOOL_: Any notes describing what a tool is or how to use it goes here. IE: `tcpdump` or `nmap`.
- _5_CONCEPT_: This is the largest directory. Anything that is a security _concept_ lives in here. For example: `Cryptography`
- _6_PROMPTS_: A new addition, this is my prompt library where I store useful prompts for AI systems.
- _99_OTHER_: Anything that doesn’t fit neatly into another directory gets stuffed into this one.

![Obsidian Folder Structure](https://grahamhelton.com/assets/images/Pasted%20image%2020260415115909-800.png)

Obsidian Folder Structure

This entire structure feeds into the note creation process of the vault which is the largest change I’ve made.

## Frontmatter

[Obsidian properties](https://obsidian.md/help/properties) are the key to my setup. Every note follows a very similar YAML frontmatter schema, allowing me to add structured data to each note. Following a familiar structure for every note has powerful implications for automation, organization, and visualization, especially when using [Obsidian Bases](https://obsidian.md/help/bases).

The frontmatter schema contains the following properties

```
---
title: knowledge management
date_created: 2026-04-15
description: "The systematic process of capturing, organizing, storing, sharing, and applying information"
type: concept
domain:
  - Career
reference:
  - "[[writing]]"
  - "[[obsidian]]"
---
```

The specific fields matter less than picking them and sticking to them. The value of frontmatter comes from using them consistently (make sure you template them out!)

Properties also unlock [Obsidian Bases](https://obsidian.md/help/bases), a relatively new feature that turns your notes into databases.

Here are two examples of how I use bases:

1. A base from my _brain_ vault that functions as my cookbook with each _card_ being a recipe note.
2. A base from my _security_ vault that filters for tools, their description, and notes they reference.

![Cookbook base where each card is a markdown file](https://grahamhelton.com/assets/images/Pasted%20image%2020260415142044-800.png)

Cookbook base where each card is a markdown file

![Tool description base where each row is a markdown file](https://grahamhelton.com/assets/images/Pasted%20image%2020260415141725-800.png)

Tool description base where each row is a markdown file

## Note Creation

The note creation workflow is the most unorthodox part of my Obsidian vault and I only use it for the security vault. Each time a note is created in the vault, a dialogue box appears that forces me to go through a standardized flow that auto-populates the properties within a note. This flow is slightly different depending on which _type_ of note is being created. For a _concept_ note the flow looks like this:

1. Type a name for the note
2. Select what _type_ of note this is
3. Select what domain this note falls under (redteam, blueteam, career, other, etc)
4. Select any other note I want referenced

0:000:00

Creating a concept note

For a _technique_ note, the flow is the same, except instead of setting the _domain_ property it sets a _tag_ that corresponds to [MITRE ATT&CK](https://attack.mitre.org/).

0:000:00

Creating a technique and tagging it to MITRE ATT&CK

This **does** add some friction to creating a note but I’ve come to not mind it. I’ve used this system long enough that it’s become second nature, but it also has an unintended outcome. If I create a note that doesn’t fit cleanly into one of these categories, it probably doesn’t follow the atomic note taking structure I want my vault to be in. This structured note creation process enforces that I create atomic notes called something like “knowledge management” and “obsidian” and not “how to do knowledge management using obsidian”.

## Atomic notetaking v2

I detailed why atomic notetaking works for me in the original post, but I will quickly recap what I mean when I say it. Atomic notetaking means each note captures one concept, even if that means creating multiple linked notes instead of one long entry. For example, recently I’ve been taking a lot of notes on concepts, tools, and techniques related to AI.

![Graph view of my AI notes](https://grahamhelton.com/assets/images/Pasted%20image%2020260415140057-800.png)

Graph view of my AI notes

When I come across a new term that is related to AI security, instead of making a new note called _how to do a prompt injection attack_ and _how to defend against prompt injection_ I create a technique note called _prompt injection_. As I’m researching prompt injection, I might understand more about the structure, specifically there are variants of prompt injection such as _indirect prompt injection_ and there might be defenses for prompt injection such as _input validation_. Each of these get created in obsidian using the same flow as before

![Example of what an atomic note contains](https://grahamhelton.com/assets/images/Pasted%20image%2020260415140928-800.png)

Example of what an atomic note contains

The benefit here is seeing linked notes within the graph view. Just by looking at the graph view I can see _indirect prompt injection_ -> _prompt injection_ -> _input validation_. Technique -> concept -> defense.

That’s useful if I ever encounter indirect prompt injection and need to recommend a defense.

## Finding Gaps in Knowledge

When I take notes on something in obsidian, I’m heavily using the `[[wikilinks]]` syntax to create a _stub_ of a new note. Surrounding a word or concept with `[[double brackets]]` doesn’t _create_ the note in the vault, but it does add it as a grey node in the graph view. If I begin to see a large amount of grey’d out nodes around a note, that means there is a lot of gaps in my knowledge in that area.

In this example, you can see several gaps in my knowledge of docker storage mechanisms.

![Gaps in knowledge in docker storage methods](https://grahamhelton.com/assets/images/Pasted%20image%2020260415224541-800.png)

Gaps in knowledge in docker storage methods

Another useful way to find gaps in your knowledge is to identify what notes have many links connecting to it. If many notes mention something you’re not familiar with, you should probably spend some time learning it. A great example of this is my `TLS` note with dozens of notes mentioning it.

  
You can never know too much about TLS…

![TLS note with many incoming and outgoing links](https://grahamhelton.com/assets/images/Pasted%20image%2020260428030204-800.png)

TLS note with many incoming and outgoing links

## Using AI

There are a few areas where AI tools really shine in obsidian. First, you can spin up entirely custom extensions in minutes. AI tools like Claude Code genuinely excel at working in a structured environment like an obsidian vault. A well structured `claude.md` file explaining how to navigate the vault still seems borderline magical.

Aside from generic vault admin work like mass-updating frontmatter to conform to a different schema or generating descriptions/summaries of tools to store in frontmatter, I’m still experimenting with other areas agents can be helpful within my vaults. ## Agents

AI agents live and breathe markdown files which means there are many options for using them when working in obsidian. Despite that reality, my usage of agents within my vault is relatively _focused_. The goal of my knowledge base is actually _not_ to store as much information as possible. It is not advantageous in any way to have thousands of unreviewed markdown file sitting on disk created by an agent.

I could easily get claude code to churn through tokens filling out my notes on `TLS` or `github actions`. I could also google it or, more likely, prompt an LLM with the exact information I need at the time I need it.

There are other reasons the vast majority of the data in my vaults are written by me and by hand.

1. Writing notes by hand helps me synthesize and make sense of the information
2. Writing notes by hand allows me to make connections (via `[[wikilinks]]`) to ideas or concepts that might not technically have anything to do with what I’m making notes on.
3. As mentioned before, it allows me to map out areas where there are gaps in my knowledge

Truthfully, the speed AI tools are evolving makes it difficult to pin down a single workflow for using agents in my vault. I expect to continue integrating AI tools into my obsidian vault and have a more cohesive workflow nailed down at some point soon. When I do, I will document it in part 3 of this series.

## Moving Forward

Obsidian is becoming less of a note taking application and more of an IDE for my thoughts and writing. IDEs like VSCode have a very specific job that includes making the structural work of coding easier so you can spend your attention on the hard part. Linters, jump-to-definition, fuzzy finders, and custom extensions removes friction from everything that isn’t writing the code (once you learn how to use them of course). That’s very similar to what I’ve been building inside of Obsidian with the note creation flow, bases, graph views, and custom plugins.