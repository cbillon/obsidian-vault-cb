---
link: https://hister.org/
site: Hister
excerpt: Hister is a private, self hosted search engine for the pages you visit
  and the files you keep. Search from the web, terminal, command line, or MCP.
slurped: 2026-09-18T11:42
title: Hister | Your Own Search Engine
---

## Self hosted

Run it on your own machine or server

## Privacy focused

No telemetry, no external requests

## Preserve Knowledge.  
Find It Again.

Hister stores extracted document content with the search index and displays it as a readable preview alongside search results.

[Download Hister](https://github.com/asciimoo/hister/releases/latest) [Try the live demo](https://demo.hister.org/)

## Search the content you chose to index

Look beyond bookmarks and filenames. Hister indexes the full content of the pages and files you choose, then keeps that searchable knowledge on a server you control.

### Search the actual content

Look beyond titles and URLs to the words inside every indexed document.

### Narrow with precision

Use fields, phrases, wildcards, negation, priorities, and your own aliases.

### Read it in context

Open a clean stored preview beside the results without losing your search.

[Explore the query language](https://hister.org/docs/query-language)

## A private memory without the busywork

Save newly visited pages with the browser extension, watch local folders, import your history, or crawl a site.

Hister extracts the parts that matter and indexes their full text on the server you choose.

Search from the web, terminal, command line, or let an AI assistant retrieve it through MCP.

Browser extensions can index pages as they are visited. File watchers, history imports, and crawlers add other sources to the same index.

## Preserve Knowledge.  
Keep Control.

The index, stored page content, and rules remain on the Hister server you configure. The server has no telemetry and does not require a cloud service.

[Read the privacy model](https://hister.org/docs/intro#privacy)

### No telemetry

The server does not phone home or report what you search.

### No mandatory cloud

A complete personal setup can run on one local machine.

### Your chosen server

Clients send indexed content only to the Hister server you configure.

### Auditable software

The source is public and licensed as free software under AGPLv3.

Optional semantic search sends text to the embeddings endpoint you configure. Browser extensions may retrieve page favicons. You choose whether and where these connections run.

## One index. Many ways back in.

Hister indexes visited pages, watched files, imported browser history, and crawled websites. The index is available through web, terminal, CLI, HTTP API, and MCP interfaces.

### Index automatically

Browser extensions can index visited pages automatically. File watching, history imports, and the crawler add other sources.

- Browser extensions
- Local file watching
- History import
- Website crawler

### Search precisely

Full text search supports field filters, quoted phrases, wildcards, negation, date ranges, and query aliases.

- Field filters
- Quoted phrases
- Wildcards and negation
- Query aliases

### Extract content

Content extractors handle structured data from supported formats and websites. Semantic search is optional.

- Content extractors
- Semantic search
- Language aware indexes
- Readable previews

### Apply index rules

Skip and priority rules control indexing and ranking. Versioning can retain earlier document content.

- Skip rules
- Priority rules
- Version tracking
- Sensitive content checks

### Use multiple interfaces

The same index is available through the web interface, terminal client, CLI, HTTP API, and MCP server.

- Web interface
- Terminal interface
- HTTP API and CLI
- MCP server

### Choose a deployment

A single binary can run locally. Shared servers support user scoped access with SQLite or PostgreSQL.

- No config quickstart
- SQLite or PostgreSQL
- Multiple users
- Docker and Nix

### Additional capabilities

Hister also supports multiple crawler backends, language specific indexes, content versioning, ownership rules, and configurable extractors.

[Explore docs](https://hister.org/docs)