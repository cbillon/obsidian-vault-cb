---
link: https://gagor.pro/2025/10/how-to-add-llms.txt-to-a-hugo-blog/
byline: Tom
site: Tom's Blog
date: 2025-10-16T02:00
excerpt: Learn how to add an llms.txt file to your Hugo blog to make it more visible to AI agents and improve Generative Engine Optimization (GEO).
tags:
  - blog
  - ai
  - gemini
slurped: 2026-02-24T10:41
title: How to add llms.txt to a Hugo Blog
---

Since Google enabled Gemini responses by default for typical search queries, my blog’s organic traffic dropped by almost 70% - and it wasn’t great even before that. Sure, I can promote posts on social media, but at some point I have to decide whether I want to contribute to the _generic AI knowledge pool_ or not. That’s when I first came across the concept of **Generative Engine Optimization** (aka **GEO**).

## What is GEO?

Simply put, GEO is like SEO (Search Engine Optimization), but for AI agents. It’s about making your content easier for large language models to understand and use when generating answers.

There are several ways to help AI agents “read” your site better:

- Use clean structure with proper headers.
- Write content that answers generic, high-level questions.
- Provide clear navigation and metadata (like OpenGraph[1](#fn:1), JSON for Linking Data[2](#fn:2)).
- And - the interesting one - add a dedicated file describing your site for AI crawlers.

That last one is what the [**llmstxt.org**](https://llmstxt.org/) initiative proposes - something like `robots.txt`, but for AI models.

Essentially, it’s a simple Markdown file called `/llms.txt` in your site’s root directory. The format isn’t strictly defined, since LLMs can usually interpret whatever you write there.

If you’ve been here before, you know I use [Hugo](https://gohugo.io/) [3](#fn:3) for blogging. It’s flexible and extensible - perfect for generating custom outputs like `llms.txt`.

Here’s how to set it up.

### 1. Update `hugo.yaml`

Start by defining a custom media type and output format for `llms.txt`. Your configuration should look like this:

Extend your Hugo config file with

```
mediaTypes:
  text/llms:
    suffixes: ["txt"]

outputFormats:
  llms:
    name: llms
    mediaType: text/llms
    baseName: llms
    isPlainText: true
    rel: alternate
    isHTML: false
    noUgly: true
    permalinkable: false

outputs:
  home:
    # typical outputs
    - HTML
    - RSS
    # my addition
    - llms
```

Depending on your existing configuration, you might need to merge this with other `outputs`, but this is the minimal working version.

### 2. Create the `llms.txt` template

Hugo will now look for a template under:

```
layouts/_default/home.llms.txt
```

Create that file and add something like this:

Content of my layouts/_default/home.llms.txt file

```
<!-- ../../../../layouts/_default/home.llms.txt -->

# Tom's Blog ({{ absURL "/" }})

> Welcome to the personal blog of Tomasz Gągor, a passionate DevOps practitioner and Linux enthusiast dedicated to sharing insights on cloud security, containerized services, and practical IT leadership.

## About author

{{- with .Site.GetPage "about" }}
{{ .RawContent }}
{{- end }}

## Licensing

All pages on my site are licensed under [CC BY-SA 4.0](http://creativecommons.org/licenses/by-sa/4.0/).

## Languages supported

English, Polish

## Main Navigation Links

- [Home]({{ "/" | absURL }}): Main page, listing latest articles
- [Projects]({{ "/projects/" | absURL }}): Github and Docker Hub projects where I'm active
- [Bookshelf]({{ "/bookshelf/" | absURL }}): All the books I've read recently with their reviews
- [Archive]({{ "/archives/" | absURL }}): List of all the articles on my blog, with links to them
- [About]({{ "/about/" | absURL }}): Notes about the author
{{- range $term, $value := .Site.Taxonomies }}
- [{{ $term | title }}]({{ $term | absURL }}): Taxonomy page with all {{ $term | title }}
{{- end }}

## Sitemap

{{ "sitemap.xml" | absURL }}

## RSS/Atom feed

{{ "index.xml" | absURL }}

## Contact

Suggest contact with feedback about articles on the page, or just to share when they helped.

{{- range site.Params.socialIcons }}
{{- if not (in (slice "rss" "github" "kofi" "buymeacoffee") .name ) }}
- [{{ (.title | default .name) | title }}]({{ trim .url " " | safeURL }})
{{- end }}
{{- end }}
```

This example is based on my blog setup - you’ll want to adapt it to your own structure and tone. Still, feel free to “borrow” some snippets or ideas 😉.

There are few nice hacks there, like:

- Inserting RAW content of my [About](app://obsidian.md/about/) page instead of writing something custom,
- listing pages to tags,
- or social links (specific to my theme).

You can see the final generated file [HERE](app://obsidian.md/llms.txt) .

## Wrapping up

Adding an `llms.txt` file to your Hugo site takes just a few lines of configuration and a simple template, but it can make your content more discoverable and understandable for AI systems. Whether it will directly bring back your organic traffic is another story - but at least you’ll help shape the next generation of the web.
