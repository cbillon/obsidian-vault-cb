---
link: https://pele.run/
excerpt: Pele is a lightweight open source library that renders Mermaid diagrams
  to SVG, themed with CSS variables.
slurped: 2026-10-18T12:43
title: Pele · Mermaid diagrams as themeable SVG
---

## How it works

Write Mermaid syntax, Pele renders it to SVG. Try editing the example below.

Mermaid

```
flowchart TD  A[Mermaid text] --> B{Flowchart?}  B -- Yes --> C[Pele]  B -- No --> D[Mermaid]  C --> E([SVG])  D --> E
```

Use CSS variables to set colors and fonts. Color changes apply without re-rendering.

CSS

```
.pele {  --pele-bg: #fffcf0;  --pele-fg: #100f0f;  --pele-muted: #6f6e69;  --pele-line: #100f0f;  --pele-surface: #f2f0e5;  --pele-surface-alt: #e6e4d9;  --pele-border: #cecdc3;}
```

Pele

ThemesVariablesLightDark

## Why Pele?

Zero dependencies, 6× smaller, and up to 500× faster than Mermaid.js. Pele supports every diagram type built into Mermaid 12.1.0.

||Pele|Mermaid.js|
|---|---|---|
|Bundle size gzip|230 kB|1.6 MB|
|Render time|0.05–0.3 ms|6–82 ms|
|Direct dependencies|0|23|
|Small screens|Adapts the layout|Scales the diagram|
|Theming|CSS variables|Theme configuration|

## Pele in your app

Add Pele to your app using the [API](https://pele.run/api).

```
npm install pele
```

Pass Mermaid syntax to `render()` and insert the returned SVG into your page.

```
import { render } from 'pele';const { svg } = render(`flowchart LR  A[Mermaid text] --> B[SVG]`);document.querySelector('#diagram').innerHTML = svg;
```

## Explore

[

### Examples

Browse supported diagram types.

](https://pele.run/examples)[

### API

Render diagrams in your app.

](https://pele.run/api)[

### Theming

Set colors and fonts with CSS variables.

](https://pele.run/theming)[

### Compatibility

Compare syntax, layout, and features with Mermaid.

](https://pele.run/compatibility)

## Pele in use

Pele is [open source](https://github.com/obsidianmd/pele), developed for [Obsidian](https://obsidian.md/) to render diagrams quickly and match your theme. It is named after [Pele](https://en.wikipedia.org/wiki/Pele_$deity$), the goddess of volcanoes and fire.