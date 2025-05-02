---
link: https://github.com/fiatjaf/awesome-jq
byline: fiatjaf
site: GitHub
excerpt: A curated list of awesome jq tools and resources. Contribute to fiatjaf/awesome-jq development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2025-02-20T15:54
title: "GitHub - fiatjaf/awesome-jq: A curated list of awesome jq tools and resources."
tags:
  - jq
  - tools
---

A curated list of awesome things built with the JSON processor and _turing-complete functional language_ **jq**.

- [Awesome jq](https://github.com/fiatjaf/awesome-jq#awesome-jq)
    - [Implementations](https://github.com/fiatjaf/awesome-jq#implementations)
    - [Tools](https://github.com/fiatjaf/awesome-jq#tools)
    - [Documentation](https://github.com/fiatjaf/awesome-jq#documentation)
    - [Use Cases](https://github.com/fiatjaf/awesome-jq#use-cases)
    - [Libraries and tools for jq itself](https://github.com/fiatjaf/awesome-jq#libraries-and-tools-for-jq-itself)
    - [External libraries](https://github.com/fiatjaf/awesome-jq#external-libraries)
    - [Podcasts and presentations](https://github.com/fiatjaf/awesome-jq#podcasts-and-presentations)
    - [Contribute](https://github.com/fiatjaf/awesome-jq#contribute)
    - [License](https://github.com/fiatjaf/awesome-jq#license)

---

## Implementations

[](https://github.com/fiatjaf/awesome-jq#implementations)

_Standalone implementations of the jq language._

- [jq](https://jqlang.github.io/jq/) ([github](https://github.com/jqlang/jq)) – The original jq command-line JSON processor.
- [gojq](https://github.com/itchyny/gojq) – A jq implementation in Go.
- [jaq](https://lib.rs/crates/jaq) – A jq implementation in Rust that misses some small features but is often more correct than the original.
- [query-json (`q`)](https://github.com/davesnx/query-json) – query-json is a faster, simpler and more portable implementation of the jq language in Reason.
- [xq](https://github.com/MiSawa/xq) – Pure rust implementation of jq
- [jq.js](https://github.com/mwh/jqjs) – Pure Javascript implementation of jq
- [jqjq](https://github.com/wader/jqjq) – jq implementation of jq

## Tools

[](https://github.com/fiatjaf/awesome-jq#tools)

_jq-based JSON visualizers and explorers_.

### Command-line

[](https://github.com/fiatjaf/awesome-jq#command-line)

- [faq](https://github.com/jzelinskie/faq) – CLI program that processes BSON, Bencode, JSON, TOML, XML, YAML using **libjq**.
- [jiq](https://github.com/fiatjaf/jiq) – A visual command-line interactive JSON explorer with jq filters.
- `echo '' | fzf --print-query --preview "cat *.json | jq {q}"` – An [fzf](https://github.com/junegunn/fzf) hack that turns it into an interactive jq explorer.
- [jqq](https://github.com/jcsalterego/jqq/) – A visual command-line interactive jq explorer written in Ruby.
- [yq](https://github.com/kislyuk/yq) (and `xq`) – jq wrapper for YAML and XML documents.
- [yiq](https://github.com/zoetrope/yiq) – Like `jiq`, but using `yq` instead, so it supports YAML documents.
- [ijq](https://github.com/fiatjaf/ijq) – jq REPL with automatic variable assignment and global statements support.
- [jqsh](https://github.com/bmatsuo/jqsh) – An interactive wrapper written in Go.
- [jq-zsh-plugin](https://github.com/reegnz/jq-zsh-plugin) – zsh line editor for constructing jq queries interactively.
- [fq](https://github.com/wader/fq) – jq for binary formats
- [jq-fish-plugin](https://github.com/jihchi/jq-fish-plugin) – Inspired by [jq-zsh-plugin](https://github.com/reegnz/jq-zsh-plugin), interactively build jq expressions in fish shell.
- [jqp](https://github.com/noahgorstein/jqp) – a TUI playground for exploring jq.
- [jnv](https://github.com/ynqa/jnv) – interactive JSON filter using jq with navigation and autocompletion.
- [jqunit](https://github.com/mrwilson/jqunit) – A test framework for JQ, written in Rust, on top of libjq.
- [play](https://github.com/paololazzari/play) – A TUI playground to experiment with your favorite programs, such as grep, sed, awk, jq and yq.

### Web

[](https://github.com/fiatjaf/awesome-jq#web)

- [query-json playground](https://query-json.netlify.app/) – Web playground that uses `query-json` compiled to JavaScript.
- jiq-web ([github](https://github.com/fiatjaf/jiq-web)) – `jiq`, but in a web page, uses `jq-web`.
- [jq play](https://jqplay.org/) ([github](https://github.com/jingweno/jqplay)) – A playground for jq with sharing capabilities.
- [jq kung fu](https://www.jqkungfu.com/) – A jq playground in WebAssembly powered by the original jq compiled with _emscripten_.
- jq-finder ([github](https://github.com/fiatjaf/jq-finder)) – A multipanel, Finder-like, JSON explorer with jq filters instead of paths, uses `jq-web`.
- [jqaas](https://github.com/captn3m0/jqaas) – jq as a service, an open HTTP endpoint that executes jq queries.
- [jqp](https://github.com/sighrobot/jqp) – A free serverless proxy for filtering JSON and CSV data using jq.
- [jqterm](https://jqterm.com/) ([github](https://github.com/remy/jqterm)) – Online playground - "jq as a service"

### Desktop

[](https://github.com/fiatjaf/awesome-jq#desktop)

- [jqi](https://nire0510.github.io/jqi/) ([github](https://github.com/nire0510/jqi)) – The almighty jq processor wrapped in a graphical UI, for Mac OSX.
- [jqview](https://github.com/fiatjaf/jqview) – A jq JSON explorer with a minimalist native GUI.

### Extensions

[](https://github.com/fiatjaf/awesome-jq#extensions)

- [bro/q](https://github.com/zalando-incubator/bro-q) – A Chrome Extension for JSON formatting and jq filtering.
- [virtual-json-viewer](https://github.com/paolosimone/virtual-json-viewer) – A JSON Chrome/Firefox Extension with virtual DOM, full-text search and jq filtering.
- [atom-jq](https://github.com/sanack/atom-jq) – Interactive jq playground inside the Atom editor.
- [jq-mode](https://github.com/ljos/jq-mode) – A jq mode for Emacs.
- [vscode-jq](https://github.com/andricDu/vscode-jq) – A jq extension for VS Code.
- [vscode-jq-playground](https://github.com/davidnussio/vscode-jq-playground) – A jq playground notebook extension for VS Code.
- [vim-jqplay](https://github.com/bfrg/vim-jqplay) – Interactive jq playground inside Vim.
- [jq-playground.nvim](https://github.com/yochem/jq-playground.nvim) – Interactive jq playground inside Nvim, written in Lua.
- `:%!jq '.'` is a Vim command that formats JSON in-place with jq (beware of any other tricks you might be thinking of).
- [jq-lsp](https://github.com/wader/jq-lsp) – jq language server. Works with VSCode, neovim and Emacs. Has syntax and scope checking, goto defintion, completion and hover documentation.
- [vscode-jq](https://github.com/wader/vscode-jq) – VSCode jq extension that uses [jq-lsp](https://github.com/wader/jq-lsp). Has syntax highlight, snippets and everything jq-lsp provides.
- [bat syntax highlighting](https://github.com/jqlang/jq/wiki/bat-language-syntax) – Syntax file to use bat to syntax highlight jq files

## Documentation

[](https://github.com/fiatjaf/awesome-jq#documentation)

_Readings about jq_.

### Core documentation

[](https://github.com/fiatjaf/awesome-jq#core-documentation)

- [Manual](https://jqlang.github.io/jq/manual/) – jq manual (development version).
- [FAQ](https://github.com/jqlang/jq/wiki/FAQ) – jq FAQ.
- [Cookbook](https://github.com/jqlang/jq/wiki/Cookbook) – jq cookbook.
- [Advanced Topics](https://github.com/jqlang/jq/wiki/Advanced-Topics) – jq advanced topics.

### Good small specific tutorials

[](https://github.com/fiatjaf/awesome-jq#good-small-specific-tutorials)

- [Bash that JSON (with jq)](http://blog.librato.com/posts/jq-json).
- [JSON on the command line with jq](https://shapeshed.com/jq-json/).
- [Reshaping JSON with jq](https://programminghistorian.org/en/lessons/json-and-jq).
- [jq is sed for JSON](https://robots.thoughtbot.com/jq-is-sed-for-json).
- [Mastering jq: part 1](https://codefaster.substack.com/p/mastering-jq-part-1-59c)
- [An Introduction to JQ](https://earthly.dev/blog/jq-select/)
- [Articles exploring and using jq for data tasks](https://qmacro.org/tags/jq/)

### Code examples

[](https://github.com/fiatjaf/awesome-jq#code-examples)

- [jq at Rosetta Code](http://rosettacode.org/wiki/Category:Jq) – Dozens of algorithms written in jq .
- [Builtins](https://github.com/jqlang/jq/blob/master/src/builtin.jq) – jq builtins coded in _jq_ itself, not C.
- [Collection of jq recipes](https://remysharp.com/drafts/jq-recipes)
- [Collection of interactive jq examples](https://ishan.page/blog/2023-11-06-jq-by-example/) – Dozens of interactive jq examples (and explanations) in the browser.
- [Collection of jq oneliners](https://nntrn.github.io/jq-recipes/)

### Documentation browsers

[](https://github.com/fiatjaf/awesome-jq#documentation-browsers)

- [jq dash docset](https://github.com/wader/jq-dash-docset)

## Use Cases

[](https://github.com/fiatjaf/awesome-jq#use-cases)

_Apps using jq in the wild_.

- [liteJQ](https://github.com/Florents-Tselai/liteJQ) – jq SQLite extension.
- [pgJQ](https://github.com/Florents-Tselai/pgJQ) – jq Postgres extension.
- [jqmd](https://github.com/bashup/jqmd) – A "literate devops" tool that allows embedding jq code, shell scripts, YAML, and JSON in a markdown document and making it executable. (A bit like R markdown or IPython notebooks, except with shell/jq/YAML/JSON, and as a CLI scripting tool rather than a GUI.)
- [sc](https://github.com/annacrombie/sc) – A lightweight [SoundCloud](https://soundcloud.com/) client, with a composable api, powered by jq.
- [jc](https://github.com/kellyjonbrazil/jc) – CLI tool that converts the output of popular command-line programs and filetypes to JSON so they can be piped to jq.
- [jqt](https://fadado.github.io/jqt/index.html) ([github](https://github.com/fadado/jqt)) – A web template engine that uses jq as expression language.
- [datasette-jq](https://github.com/simonw/datasette-jq) – A plugin that enables jq queries on JSON columns on [datasette](https://datasette.readthedocs.io/) deployments.
- [jtool](https://github.com/fadado/jtool) – jq-based JSON tools for a modern shell.
- [just-dashboard](https://kantord.github.io/just-dashboard/) – A serverless app for implementing JSON-powered dashboards with JSON or YAML files (and jq filters as strings) serving as the only source of configuration.
- [bf.jq](https://github.com/MakeNowJust/bf.jq) – A Brainfuck interpreter written in jq.
- [jq-voronoi](https://github.com/hosuaby/jq-voronoi) – Implementation of Fortune’s algorithm to calculate Voronoi diagram on jq.

## Libraries and tools for jq itself

[](https://github.com/fiatjaf/awesome-jq#libraries-and-tools-for-jq-itself)

_Incrementing jq capabilities_.

- [jqnpm](https://github.com/jqnpm/jqnpm) – A jq package manager that installs modules from GitHub and runs jq scripts.
- [JBOL](https://github.com/fadado/JBOL) – A collection of utility modules for jq (math, prelude, set, string etc.).
- [bigint](https://github.com/joelpurra/jq-bigint), [array](https://github.com/joelpurra/jq-disarray), [string](https://github.com/joelpurra/jq-stress) and [other libraries](https://github.com/joelpurra?utf8=%E2%9C%93&tab=repositories&q=jq) – jq libraries from the author of jqnpm.
- [jq-jsonpointer](https://github.com/nichtich/jq-jsonpointer) – jq module implementing JSON Pointer (RFC 6901)
- [tree-sitter-jq](https://github.com/nverno/tree-sitter-jq) – Tree sitter grammar implementation for Jq language
- [json5.jq](https://github.com/wader/json5.jq) JSON5 implementation for jq

---

## External libraries

[](https://github.com/fiatjaf/awesome-jq#external-libraries)

_Using jq from other languages_.

- [gojq](https://github.com/itchyny/gojq) – A full jq implementation in Go, usable as a library.
- [jq-web](https://github.com/fiatjaf/jq-web) – jq itself compiled to JavaScript with _emscripten_. There's also an alternative at [jqdash](https://www.npmjs.com/package/jqdash).
- [jq-go](https://github.com/threatgrid/jq-go) – Golang cgo bindings for **libjq** ([jqpipe-go](https://github.com/threatgrid/jqpipe-go) is a CLI wrapper from the same people).
- [libjq-go](https://github.com/flant/libjq-go) – Golang cgo bindings for **libjq**. This one works with recent versions of jq: 1.5, 1.6+.
- [node-jq](https://github.com/sanack/node-jq) – A jq wrapper for Node.js.
- [ruby-jq](https://github.com/winebarrel/ruby-jq) – A jq wrapper for Ruby.
- [pyjq](https://github.com/doloopwhile/pyjq) – A jq wrapper for Python.
- [jq.py](https://github.com/mwilliamson/jq.py) – Another jq wrapper for Python.
- [php-ext-jq](https://github.com/kjdev/php-ext-jq) – PHP extension for jq.
- [java-jq](https://github.com/arakelian/java-jq) – A jq wrapper for Java ([jackson-jq](https://github.com/eiiches/jackson-jq) is a Jackson extension).
- [jqr](https://github.com/ropensci/jqr) – R interface to jq.
- [Ansible jq](https://github.com/moreati/jq-filter) – A jq filter for [Ansible](https://ansible.com/) configuration manager.

## Podcasts and presentations

[](https://github.com/fiatjaf/awesome-jq#podcasts-and-presentations)

- [ThePrimeTime - The BEST CLI Tool](https://www.youtube.com/watch?v=n8sOmEe2SDg)
- [Programming By Stealth](https://pbs.bartificer.net/) ([instalments](https://pbs.bartificer.net/#instalments) PBS 155 to PBS 167)

## Contribute

[](https://github.com/fiatjaf/awesome-jq#contribute)

Please contribute! Open an issue or a PR and we’ll discuss it or merge it. If you’re opening a PR, please ensure all formatting is ok (if you’re in a hurry just open an issue).

## License

[](https://github.com/fiatjaf/awesome-jq#license)

[![CC0](https://camo.githubusercontent.com/87dcb0978b064fa1825115518f85a4161f31121bedc6da8543bbb0efbf3154d8/68747470733a2f2f6c6963656e7365627574746f6e732e6e65742f702f7a65726f2f312e302f38387833312e706e67)](https://creativecommons.org/publicdomain/zero/1.0/)