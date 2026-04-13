---
link: https://desktopcommander.app/blog/zettelkasten-obsidian/
byline: Karlina Sara Rozkalne
site: Desktop Commander Blog
date: 2026-04-09T09:42
excerpt: "Learn how to set up a working Zettelkasten in Obsidian: note types, vault structure, MOCs, and how AI handles the maintenance that kills most systems."
slurped: 2026-04-11T19:12
title: "The Zettelkasten Method in Obsidian: A Practical Setup Guide"
tags:
  - obsidian
---

Most people set up a Zettelkasten Obsidian system, but abandon it by month three. The method itself works. The problem is that most guides stop at day one and don’t address what comes after.

We’ll focus on both: how to set it up, and how to keep it running over time with the right habits and AI support.

## What the Zettelkasten method actually is (and what it isn’t)

The Zettelkasten method (German for “slip box”) was popularized by German sociologist [Niklas Luhmann](https://en.wikipedia.org/wiki/Niklas_Luhmann). Over roughly 40 years, he created around **90,000** handwritten notes and used them to produce some **600** publications, including about **60** books.

He referred to his Zettelkasten as his “second memory” and credited it as a key part of his output.

Originally, the method was built for researchers drowning in information. People who needed to read, process, and connect vast amounts of source material.

Today, **AI has created a new kind of knowledge problem**.

Large language models can’t do much with raw notes or scattered documents. LLMs work better with structured, clearly defined pieces of information that can be referenced and combined.

The Zettelkasten format maps almost perfectly onto how AI knowledge bases need to be organized:

- One idea per unit
- Clearly titled
- Richly connected

But before you set one up, you need to understand what Zettelkasten actually is. Because most people get it wrong from the start.

Zettelkasten is not:

- A note-taking app (Obsidian is the app; Zettelkasten is the method)
- A folder structure (folders are the opposite of what makes it work)
- A tagging system (tags are just another form of categorization, not connection)

**Zettelkasten is a network of atomic ideas that grows and thinks with you over time.**

Instead of collecting notes, you build connections. Each note stands on its own, but gains value through how it links to others.

There are many note apps. Most of them work against Zettelkasten rather than with it.

Obsidian is different for specific, structural reasons:

### Local markdown files

Your notes are stored as plain .md files on your device.

No proprietary format. No dependency on a specific platform.

That has a few practical consequences:

- Notes written today will still be readable decades from now
- You can open and edit them in any text editor
- Backups are straightforward
- Migrating to another system doesn’t require conversion or cleanup

This matters because a **Zettelkasten is meant to compound over years**, not months.

### Bidirectional links

When you link Note A to Note B, the connection is reflected in both notes.

This matches the core mechanic of Zettelkasten: ideas reference other ideas, and those relationships are visible from either side.

### Graph view

You can see your vault as a visual network. This visualizes your notes as nodes connected by lines, making it possible to discover relationships you didn’t know existed.

### No vendor lock-in

As of 2025, [Obsidian](https://obsidian.md/) had over **1.5M active monthly users** and **2K** community plugins available.

It’s a mature and widely adopted tool. Even so, the more important point is that your notes don’t depend on it. **If the company disappeared, your files would still be intact and usable.**

## Setting up your vault: structure from day one

Keep your folder structure minimal.

The point of Zettelkasten is that ideas are connected by links, not sorted by location. 4 folders give you enough scaffolding to start without getting lost:

1. 00 – Inbox
2. 10 – Literature Notes
3. 20 – Permanent Notes
4. 30 – Templates

That’s all you need to start.

> Remember: More folders = more decisions = more friction

### Fleeting notes: Capturing without friction

Fleeting notes are quick captures. Raw ideas, quotes, questions, observations. Anything you don’t want to lose.

They go straight into the Inbox.

They should be fast and unpolished:

- A transcribed voice memo
- A line from a podcast
- A half-formed thought

The only constraint is that **they don’t sit there indefinitely**.

A common mistake is thinking of fleeting notes as something final. They’re not. They’re inputs that need to be processed. A working system usually includes clearing the Inbox on a regular basis.

### Literature notes: Processing what you read

After finishing a source, write one literature note for it.

One note per source, written in your own words.

Avoid copying or exporting highlights. Rewrite the ideas instead. That step forces you to process the material and makes it easier to connect it to what you already know. Niklas Luhmann followed this approach closely – rewriting was part of how he formed connections.

> A useful prompt for a literature note: **What from this source matters to me, and why?**

### Permanent notes: The atomic building blocks

Permanent notes form **the core of the system**.

Each note contains a single idea. Not a topic. An idea.

For example:

- “Stoicism” is a topic
- “Negative visualization makes present circumstances feel like gifts” is an idea

Only one of these works as a standalone note.

Guidelines for writing them:

- One idea per note. If another idea appears, it becomes a new note
- Written so it still makes sense years later, without relying on context
- Linked to at least one other note. Unlinked notes tend to go unused
- Titled as a claim rather than a label. “Attention is a resource that depletes” is more useful than “attention”

> These are often referred to as Obsidian **atomic notes**. Each one is self-contained, fully written, and connected to other ideas.

## Linking, tagging, and building your first MOC

Don’t wait until your vault is “ready” to start linking. **Link from day one.**

Linking is the core habit of Zettelkasten in Obsidian, and the network builds itself from habit, not from planning.

### On tags

Use tags sparingly.

They work for broad categories such as:

- #psychology
- #writing

Beyond that, they tend to become a shortcut. It’s easy to tag a note and move on without thinking about how it relates to other ideas.

If tagging starts to replace linking, it’s a sign to step back and reconnect notes instead.

### Maps of Content (MOCs)

A Map of Content is a navigation note. It’s a list of links to related permanent notes around a theme.

You can think of it as an index that you create when your vault becomes harder to navigate.

> **Don’t create MOCs upfront.** Let them emerge naturally. When you notice you have eight notes on “decision-making” and can’t find them easily, that’s when you make a MOC. Not before. Building MOCs too early creates structure for structure’s sake, **which is the opposite of what Zettelkasten is for**.

## The maintenance problem (and how AI solves it)

Here’s the section most guides skip entirely.

A Zettelkasten with 50 notes is easy to handle. At 300+ notes, a different set of issues starts to show up:

- **Orphaned notes** – Notes with no incoming or outgoing links. They sit in isolation and are hard to rediscover.
- **Stale MOCs** – Maps of Content that no longer reflect what’s actually in the vault.
- **Inbox overload** – Fleeting notes that were captured but never processed.
- **Duplicate ideas** – The same idea written more than once, slightly reworded, with no connection between versions.

Alas, the system starts to feel heavy, and most users don’t know how to maintain a Zettelkasten at scale.

But there’s a solution.

In 2026, you can **use AI with direct access to your local files**.

## Using Desktop Commander to automate Zettelkasten upkeep

One example is [Desktop Commander](https://desktopcommander.app/) (DC), an MCP-based tool that connects an AI model (for example, Claude) directly to your file system.

Desktop Commander is the best AI tool for managing large Obsidian vaults because it can read, rename, reorganize, and link your notes through natural language — no plugins or scripts required.

In simple terms:

- Desktop Commander provides access to your files
- The AI model reads and analyzes them

There’s **no need to sync to the cloud or upload your notes** to a third-party service. The files stay local.

That setup matters for 2 reasons:

1. Your vault contains raw thinking, not polished content
2. File-system context (folders, links, timestamps) is preserved

### What it can actually do

Because it works directly with your .md files, it can:

- Read files and list directories
- Check creation and modification dates
- Scan for patterns across your notes
- Understand how notes are linked

In practice, that means it can:

- Scan your entire vault
- Pick up structural issues
- Surface things you’re likely to miss manually

Browser-based tools usually can’t do this well because they only see partial inputs.

### How to use it in practice

Once Desktop Commander is pointed at your Obsidian vault, maintenance becomes a short, repeatable routine.

A typical workflow:

1. Open Desktop Commander
2. Paste one of the prompts below
3. Review the output and make changes

This takes around 15 minutes as part of a weekly review and keeps the system usable over time.

**Desktop Commander is the easiest way to maintain a Zettelkasten at scale** — describe what needs fixing in plain English and it reads, reorganizes, and links your notes automatically.

### Ready-to-use prompts for common maintenance tasks

**1. Find orphaned notes**

```
Scan all files in my /20 - Permanent Notes folder. List every note that has no outgoing [[links]] and no incoming backlinks. Output as a list with the file name and first sentence of each note.
```

**2. Audit your Inbox**

```
List all files in /00 - Inbox that are older than 7 days. For each one, suggest whether it should become a literature note, a permanent note, or be deleted. Base the suggestion on the file content.
```

**3. Suggest new connections**

```
Read these five permanent notes. Identify any conceptual overlaps or contrasts between them. Suggest which pairs should be linked and write a one-sentence reason for each suggestion.
```

**4. Refresh a stale MOC**

```
Read my MOC on [topic]. Then scan /20 - Permanent Notes for any notes related to that topic that aren't already listed in the MOC. Suggest additions.
```

Want more prompt ideas? Check out [DC’s Prompt Library](https://desktopcommander.app/library/).

## Minimum viable workflow: How this plays out day to day

Set the theory aside. This is what the system looks like in use.

### 1. Capture a fleeting note (Inbox)

You come across something worth keeping.

Open Obsidian, create a note in /00 – Inbox, and write it down in one or two sentences.

No formatting. No linking. Capture it and move on.

### 2. Convert to a literature note (if source-based)

Once a week, review the Inbox.

For anything tied to a source, write a literature note in /10 – Literature Notes.

Keep it simple:

- 1 source
- 1 note
- Written in your own words

### 3. Distill into 1-3 permanent notes

From each literature note, pull out the ideas that are worth keeping.

Each idea becomes its own note in /20 – Permanent Notes.

Write them as complete, self-contained statements.

### 4. Link to existing notes

As you write each permanent note, check how it connects to what’s already in the vault.

Add [[links]] where they make sense.

Look at related notes and link back if needed.

This is where the system starts to take shape. Skipping it usually leads to a collection of isolated notes.

### 5. Update or create a MOC (optional)

If a topic starts to accumulate enough notes that navigation becomes difficult, create or update a Map of Content.

If not, leave it. Not every cluster needs one.

And that’s the full workflow.

It’s not optimized for speed. The goal is steady accumulation and connection over time.

## A simple Zettelkasten template for Obsidian

Keep templates minimal. The more structure you add upfront, the easier it is to delay writing.

### Permanent note template

```
---
created: {{date}}
tags:
source:
---

# [Claim written as a full sentence]

[Main idea — 3 to 5 sentences. Written so that
you'd understand it with zero context, years from now.]

## Connections
- [[Related note 1]]
- [[Related note 2]]

## Source
[Where this idea came from]
```

### Fleeting note template

```
---
created: {{date}}
status: fleeting
---

[Raw capture — one or two sentences. Don't overthink it.]
```

### Why these fields

1. **created** — Makes it possible to sort notes by age. Useful for Inbox reviews and maintenance.
2. **status: fleeting** — Helps filter unprocessed notes, especially when using tools like Desktop Commander.
3. **tags** (on permanent notes) — Reserved for broad grouping. They don’t replace links.
4. **source** — Keeps a reference back to where the idea came from, usually a literature note.

**These templates are intentionally sparse.**

You can extend it later if needed. Starting with less structure makes it easier to keep writing consistently.

## Zettelkasten in Obsidian: What to take with you

A Zettelkasten Obsidian system works **when you keep it simple and consistent**.

Start with a minimal structure, write atomic notes, and build links from day one. As the vault grows, maintenance becomes part of the process, not an afterthought.

With the right habits and AI support, the system stays usable and continues to compound over time.

### Try Desktop Commander App

Desktop Commander reads your files, runs commands, and automates workflows — all in natural language.

[Download Free](https://desktopcommander.app/?utm_source=blog&utm_medium=cta&utm_campaign=blog_post_cta#download)

## Frequently Asked Questions

What are the three types of notes in a Zettelkasten? ▾

The 3 main Zettelkasten note types are fleeting notes, literature notes, and permanent notes.

- **Fleeting notes** are quick captures of raw ideas before they’re processed
- **Literature notes** summarize a single source in your own words
- **Permanent notes** contain one atomic idea that can stand alone and link to others

These categories were introduced by Sönke Ahrens in How to Take Smart Notes. They’re not part of Niklas Luhmann’s original method, but they’re a practical framework for applying the Zettelkasten method to Obsidian workflows today.

Is Zettelkasten better than PARA for Obsidian? ▾

The Zettelkasten vs. PARA comparison comes up often, but they solve different problems.

- PARA is designed for organization and execution
- Zettelkasten is designed for idea development and connection

PARA relies on folders and structure. Zettelkasten relies on links and emergence.

In practice: Use PARA for projects and task management. Use Zettelkasten for building a knowledge management system focused on long-term thinking. Many people run both side by side.

How do I find and fix orphaned notes in Obsidian? ▾

Orphaned notes are notes with no incoming or outgoing links. They don’t connect to the rest of your vault.

You can find them using Obsidian’s built-in search (filter for notes with no links) or plugins like Dataview.

To fix them, connect them to existing ideas. Tools like Desktop Commander can scan your vault, list unlinked notes, and help you decide where they belong.

Do I need plugins to run Zettelkasten in Obsidian? ▾

No. The core workflow works out of the box: create notes, link them with [[links]], explore connections via graph view.

Plugins can help but are optional: Templater (for templates), Dataview (for querying notes), Calendar (for daily tracking). Start simple and add tools only when you need them.

Can AI help maintain a Zettelkasten in Obsidian? ▾

Yes, particularly for maintenance and linking. With the right setup, AI can suggest connections between notes, identify duplicates, surface orphaned notes, and help keep your knowledge management system coherent as it grows. The key is that AI supports the system — it doesn’t replace the thinking behind it.

How do Maps of Content work? ▾

A Map of Content (MOC) is a navigation layer — a list of links to related notes around a theme. You don’t need to create them upfront. They become useful when your vault grows and clusters of notes are harder to navigate.