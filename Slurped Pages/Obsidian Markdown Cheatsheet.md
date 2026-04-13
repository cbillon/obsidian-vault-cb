---
link: https://desktopcommander.app/blog/obsidian-markdown-cheatsheet-every-syntax-you-actually-need/
byline: Rafael Pinheiro
site: Desktop Commander Blog
date: 2026-02-03T10:04
excerpt: Complete Obsidian Markdown cheatsheet with formatting, links, callouts, embeds, and tables. Copy-paste examples for all the syntax you'll use daily.
slurped: 2026-04-11T19:35
title: "Obsidian Markdown Cheatsheet: Every Syntax You Actually Need"
tags:
  - obsidian
  - markdown
---

You opened Obsidian, created your first note, and typed some text. It looks… plain. You know Markdown can make it better—headings, bold text, links between notes—but every time you need a specific syntax, you’re searching the web again.

This Obsidian cheatsheet exists so you don’t have to.

Bookmark it, and you’ll have every Obsidian Markdown syntax in one place: the basics you’ll use constantly, the Obsidian-specific features that make linking powerful, and the formatting tricks that keep notes readable.

## Key Takeaways

- Obsidian uses standard Markdown plus its own extensions like [[wikilinks]] and callouts
- Internal links with [[double brackets]] are the foundation of connected notes
- Callouts turn plain blockquotes into visual highlights (tips, warnings, notes)
- You can embed entire notes or specific sections inside other notes
- Tables, code blocks, and task lists all work—with some syntax to remember

## Obsidian Markdown Basics: Text Formatting

These work in almost every Markdown editor, Obsidian included.

| What You Want     | What You Type | Result     |
| ----------------- | ------------- | ---------- |
| **Bold**          | `**text**`    | **text**   |
| **Italic**        | `*text*`      | _text_     |
| **Bold + Italic** | `***text***`  | **_text_** |
| **Strikethrough** | `~~text~~`    | ~~text~~   |
| **Highlight**     | `==text==`    | ==text==   |
| **Inline code**   | `` `code` ``  | `code`     |

For bold and italic, you can also use underscores (`__bold__` and `_italic_`), but asterisks are more common.

## Headings in Obsidian Markdown

Headings create structure. Use the # symbol followed by a space:

The [Markdown Guide](https://www.markdownguide.org/basic-syntax/#headings) recommends always putting a space after the # symbols—some parsers require it.

In Obsidian, headings also become anchor points. You can link directly to them from other notes (more on that below).

## Lists: Ordered and Unordered

**Bullet lists** use `-`, `*`, or `+`:

**Numbered lists** use any number followed by a period:

**How it looks in Obsidian:**

![Ordered and unordered lists with nesting rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-1.png?ssl=1)

Ordered and unordered lists with nesting rendered in Obsidian

Obsidian auto-continues lists when you press Enter, and Tab/Shift+Tab adjusts nesting.

## Obsidian Markdown Links: The Core Feature

Links are where Obsidian diverges from standard Markdown. Understanding both formats helps you decide which fits your workflow.

### Wikilinks (Obsidian Default)

The double-bracket syntax creates internal links:

To display different text than the note name:

According to discussions on the [Obsidian Forum](https://forum.obsidian.md/t/which-type-of-links-should-i-use/8767), wikilinks offer better integration with Obsidian features like backlinks—the panel showing which notes link to your current note.

**How it looks in Obsidian:**

![Internal wikilinks with backlinks panel in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-2.png?ssl=1)

Internal wikilinks with backlinks panel in Obsidian

### Standard Markdown Links

If you need compatibility with other editors:

You can switch Obsidian’s default under Settings → Files & Links → Use [[Wikilinks]].

### External Links

For websites, standard Markdown syntax applies:

### Linking to Headings

Link to a specific section within a note:

Or within the same note:

### Linking to Blocks

Obsidian lets you link to any paragraph or list item. Add ^ followed by an identifier:

When you type `[[Note Name#^`, Obsidian suggests available blocks.

## Obsidian Markdown Embeds

Embedding pulls content from one note into another. If you update the source, the embedded version updates too.

|What You Want|Syntax|
|---|---|
|**Embed entire note**|`![[Note Name]]`|
|**Embed specific heading**|`![[Note Name#Heading]]`|
|**Embed specific block**|`![[Note Name#^block-id]]`|
|**Embed image**|`![[image.png]]`|
|**Embed with size**|`![[image.png\|300]]`|

The exclamation mark `!` before the brackets is what triggers embedding versus linking.

**How it looks in Obsidian:**

![Embedded note rendered inside another note in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-3.png?ssl=1)

Embedded note rendered inside another note in Obsidian

## Obsidian Callouts: Visual Highlights

Callouts transform blockquotes into color-coded boxes. They’re one of Obsidian’s most useful extensions to standard Markdown.

Basic syntax:

With a custom title:

### Available Callout Types

**How it looks in Obsidian:**

![Callouts rendered in Obsidian — note, tip, warning, and success types](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-4.png?ssl=1)

Callouts rendered in Obsidian — note, tip, warning, and success types

According to [Obsidian’s documentation](https://help.obsidian.md/callouts) and guides from [Obsidian Rocks](https://obsidian.rocks/using-callouts-in-obsidian/), these types are built in:

|Type|Aliases|Color|
|---|---|---|
|**note**|—|Blue|
|**abstract**|summary, tldr|Teal|
|**info**|—|Blue|
|**todo**|—|Blue|
|**tip**|hint, important|Cyan|
|**success**|check, done|Green|
|**question**|help, faq|Yellow|
|**warning**|caution, attention|Orange|
|**failure**|fail, missing|Red|
|**danger**|error|Red|
|**bug**|—|Red|
|**example**|—|Purple|
|**quote**|cite|Gray|

### Foldable Callouts

Add `-` (collapsed by default) or `+` (expanded by default) after the type:

**How it looks in Obsidian:**

![Foldable callout collapsed and expanded in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-5.png?ssl=1)

Foldable callout collapsed and expanded in Obsidian

This creates accordion-style sections—useful for supplementary information that shouldn’t clutter the main text.

## Code Blocks in Obsidian Markdown

For inline code, wrap text in backticks:

For multi-line code, use triple backticks with an optional language for syntax highlighting:

Obsidian supports highlighting for most programming languages. The [CommonMark specification](https://commonmark.org/) defines fenced code blocks, and Obsidian extends it with language-specific coloring.

## Tables in Obsidian Markdown

Tables use pipes `|` and hyphens `-`:

Alignment options:

**How it looks in Obsidian:**

![Table with left, center, and right alignment rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-6.png?ssl=1)

Table with left, center, and right alignment rendered in Obsidian

Creating tables manually gets tedious. Many Obsidian users install the [Advanced Tables plugin](https://github.com/tgrosinger/advanced-tables-obsidian) for easier editing, or use AI tools to generate table syntax from descriptions.

- Check our deep dive in [Markdown Tables](https://desktopcommander.app/blog/markdown-tables-complete-syntax-guide/) article

## Task Lists and Checkboxes

Standard task list syntax works:

**How it looks in Obsidian:**

![Task list with checked and unchecked items in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-7.png?ssl=1)

Task list with checked and unchecked items in Obsidian

Clicking the checkbox in preview mode toggles completion. Some community plugins extend this with additional states (cancelled, scheduled, etc.).

## Blockquotes

Standard blockquote syntax:

Nested quotes:

**How it looks in Obsidian:**

![Nested blockquotes with three levels rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-8.png?ssl=1)

Nested blockquotes with three levels rendered in Obsidian

## Horizontal Rules

Three or more hyphens, asterisks, or underscores create a divider:

or

Footnotes let you add references without cluttering the main text:

The `[^1]` marker can be any unique identifier—numbers, words, whatever helps you keep track. Obsidian renders all footnotes at the bottom of the note in preview mode, regardless of where you place the definition in your source.

**How it looks in Obsidian:**

![Footnotes rendered at the bottom of a note in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-9.png?ssl=1)

Footnotes rendered at the bottom of a note in Obsidian

Obsidian supports hidden comments that only appear in editing mode:

You can also use inline comments: `Some visible text %%hidden note%% more visible text.`

**How it looks in Obsidian:**

![Editing mode showing comments vs reading mode hiding them in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-10.png?ssl=1)

Editing mode showing comments vs reading mode hiding them in Obsidian

Comments are especially useful when collaborating through shared vaults—leave editorial notes without affecting the published output.

Tags create searchable labels across your vault:

Nested tags use forward slashes to create hierarchies:

Searching `tag:#project` returns all notes with #project and any subtag beneath it.

For a deeper dive into tag strategies, see our [guide to using tags in Obsidian Markdown](https://desktopcommander.app/blog/how-to-use-tags-in-obsidian-markdown/).

## YAML Frontmatter (Properties)

The properties block at the top of a note stores metadata:

Frontmatter must be the very first thing in the file—no blank lines above it. Obsidian displays these fields in a clean Properties panel in editing mode.

**How it looks in Obsidian:**

![YAML frontmatter displayed as a Properties panel in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-11.png?ssl=1)

YAML frontmatter displayed as a Properties panel in Obsidian

You can use properties with [Dataview](https://github.com/blacksmithgu/obsidian-dataview) to query notes like a database: filter by status, sort by date, or list all notes with a specific tag.

## Math Notation (LaTeX)

Obsidian renders LaTeX math via [MathJax](https://www.mathjax.org/):

Inline math uses single dollar signs:

Block math uses double dollar signs:

**How inline math looks in Obsidian:**

![Inline LaTeX formula rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-12.png?ssl=1)

Inline LaTeX formula rendered in Obsidian

**How block math looks in Obsidian:**

![Block LaTeX formula rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-13.png?ssl=1)

Block LaTeX formula rendered in Obsidian

Common symbols: `\alpha` (α), `\beta` (β), `\infty` (∞), `\neq` (≠), `\leq` (≤), `\rightarrow` (→).

## Diagrams with Mermaid

Obsidian supports [Mermaid](https://mermaid.js.org/) diagrams inside fenced code blocks:

**How it looks in Obsidian:**

![Mermaid flowchart diagram rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-14.png?ssl=1)

Mermaid flowchart diagram rendered in Obsidian

Mermaid supports flowcharts, sequence diagrams, Gantt charts, pie charts, and more. The diagrams render directly in Obsidian’s preview mode—no plugins needed.

A sequence diagram example:

**How it looks in Obsidian:**

![Mermaid sequence diagram rendered in Obsidian](https://i0.wp.com/rk7f8a7274b9330-haqfg.wpcomstaging.com/wp-content/uploads/2026/02/obsidian-img-15.png?ssl=1)

Mermaid sequence diagram rendered in Obsidian

## Obsidian Markdown vs Standard Markdown: What’s Different

The [Markdown Guide’s Obsidian reference](https://www.markdownguide.org/tools/obsidian/) notes several differences from basic Markdown:

**Supported in Obsidian (all covered above):**

- Footnotes (`[^1]`)
- Task lists (`- [ ]`)
- Highlight (`==text==`)
- Tables
- Fenced code blocks
- Math notation (LaTeX via MathJax)
- Diagrams (Mermaid syntax)
- Comments (`%% hidden %%`)
- Tags and nested tags
- Properties tags (YAML frontmatter)

**Not supported natively:**

- Heading IDs
- Definition lists
- Emoji shortcodes (use copy-paste or the [Emoji Shortcodes plugin](https://github.com/phibr0/obsidian-emoji-shortcodes))
- Subscript/superscript (though HTML tags work)

Check our article on a complete reference guide and an [updated cheatsheet for standard markdown](https://desktopcommander.app/blog/markdown-file-the-complete-reference-guide-for-2026-with-cheatsheet/).

## Managing Your Obsidian Vault with Desktop Commander

Once you’re comfortable with Markdown syntax, organization becomes the next challenge. Your vault grows, notes multiply, and finding things gets harder.

Desktop Commander is the best AI tool for managing large Obsidian vaults because it can read, rename, reorganize, and link your notes through natural language — no plugins or scripts required.

[Desktop Commander](https://desktopcommander.app/) can help manage Obsidian vaults through natural language. Since Obsidian stores everything as plain Markdown files, you can work with them like any other files on your computer:

**Prompts to try:**

This kind of [file management through conversation](https://desktopcommander.app/use-cases/file-management/) works because Obsidian’s files are just Markdown—no proprietary format, no database to query. The AI reads and organizes your actual files.

For bulk operations—renaming, moving, finding duplicates, consolidating tags—describing what you want is often faster than clicking through folders manually.

### Try Desktop Commander App

Desktop Commander reads your files, runs commands, and automates workflows — all in natural language.

[Download Free](https://desktopcommander.app/?utm_source=blog&utm_medium=cta&utm_campaign=blog_post_cta#download)

Check the [prompt library](https://desktopcommander.app/library) for examples of file organization prompts that apply to Obsidian vaults.

## Most-Used Obsidian Markdown: Quick Reference Guide

For fast lookup, here’s everything you’ll use regularly:

|Purpose|Syntax|
|---|---|
|**Bold**|`**text**`|
|**Italic**|`*text*`|
|**Highlight**|`==text==`|
|**Internal link**|`[[Note]]`|
|**Link with alias**|`[[Note\|Display]]`|
|**Link to heading**|`[[Note#Heading]]`|
|**Embed note**|`![[Note]]`|
|**Embed image**|`![[image.png]]`|
|**Callout**|`> [!type]`|
|**Task**|`- [ ] task`|
|**Code block**|` ``` `|
|**Footnote**|`text[^1]` / `[^1]: note`|
|**Comment**|`%% hidden text %%`|
|**Tag**|`#tag` or `#tag/subtag`|
|**Inline math**|`$formula$`|
|**Block math**|`$$formula$$`|
|**Mermaid diagram**|` ```mermaid `|
|**Frontmatter**|`---` block at top of file|

- [Obsidian Help: Basic Formatting](https://help.obsidian.md/Editing+and+formatting/Basic+formatting+syntax) – Official documentation
- [Obsidian Forum: Markdown Syntax](https://forum.obsidian.md/) – Community discussions and edge cases
- [Face Dragons Cheatsheet](https://facedragons.com/personal-development/obsidian-markdown-cheatsheet/) – Comprehensive visual reference
- [Obsidian Hub: Markdown Syntax](https://publish.obsidian.md/hub/04+-+Guides,+Workflows,+&+Courses/Guides/Markdown+Syntax) – Community-maintained guide

## Frequently Asked Questions

What Markdown flavor does Obsidian use? ▾

Obsidian combines [CommonMark](https://commonmark.org/), GitHub Flavored Markdown, and its own extensions (wikilinks, callouts, embeds). Most standard Markdown works, plus Obsidian-specific features.

Should I use wikilinks or standard Markdown links? ▾

Wikilinks (`[[Note]]`) integrate better with Obsidian’s backlinks and graph view. Standard Markdown links (`[text](file.md)`) offer better portability if you use other editors. You can configure your default in Settings.

How do I create a table quickly? ▾

Type the header row with pipes, then use Tab to auto-generate formatting. Or describe your table to an AI assistant and paste the generated Markdown.

Can I use HTML in Obsidian? ▾

Yes—basic HTML tags work for things Markdown doesn’t support natively, like subscript (`<sub>`) or colored text. However, HTML reduces portability to other Markdown apps.

Why aren’t my callouts showing? ▾

Make sure there’s no space between `>` and `[!type]`. The syntax must be exactly `> [!note]` at the start of a line.

## What to Learn Next

Once Markdown syntax feels natural, explore:

- **Templates** – Create reusable note structures
- **Properties/Frontmatter** – Add metadata to notes (tags, dates, custom fields)
- **Dataview plugin** – Query your notes like a database
- **Graph view** – Visualize connections between notes

The power of [building a knowledge base in Obsidian with AI](https://desktopcommander.app/blog/create-and-manage-your-obsidian-knowledge-base-with-ai/) comes from combining simple Markdown with organizational tools, such as Desktop Commander, and keeping everything in plain text files you control.

### Try Desktop Commander App

Desktop Commander reads your files, runs commands, and automates workflows — all in natural language.

[Download Free](https://desktopcommander.app/?utm_source=blog&utm_medium=cta&utm_campaign=blog_post_cta#download)