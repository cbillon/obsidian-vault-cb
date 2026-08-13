---
tags:
  - documents
---
[Docu browser](https://github.com/linuxrebel/DocuBrowser)

**DocuBrowse turns a messy pile of documents into something you can actually search.** Point it at your files — PDFs, ebooks, Word docs, notes, whatever — and it builds a smart index that understands not just keywords, but meaning. Ask for "that contract about the lease renewal" and find it even if those exact words never appear. Click any result for an instant AI summary before you even open the file. PII aware and works with multiple document directories.

DocuBrowse runs entirely on your own machine using local AI models — no internet connection required, no accounts, no API keys, and no per-query costs eating into a token budget. **Your data. Your AI.**

Under the hood: SQLite FTS5 keyword search plus AI-powered semantic similarity and synopsis generation (Ollama + nomic-embed-text + dolphin3). Supports multiple document and source code types.