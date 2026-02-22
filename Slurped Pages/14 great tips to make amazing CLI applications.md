---
link: https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3
byline: Manuel Odendahl
site: DEV Community
date: 2022-05-11T03:18
excerpt: I find command-line tools an incredibly way to multiply the impact of the code they produce.   Did...
twitter: https://twitter.com/@mnmlmnl
tags:
  - tui
  - cli
slurped: 2026-01-29T09:13
title: 14 great tips to make amazing CLI applications
---

I find command-line tools an incredibly way to multiply the impact of the code they produce.

Did you write a nice library to wrap an API and connect it to your DB model? Then provide a CLI tool that allows anybody to query the API, interact with the DB model, run the operations that you provided, discover all the available functionality and read the API documentation. You can now easily write scripts that exercise different functionality of your code while developing, create pre-commit scripts, create CICD scripts.

[![bubble tea demo](https://camo.githubusercontent.com/3e1285f61fa4de4bfa8efffb8ea55a2e62503515c9d9bdcde650c8a523eb2651/68747470733a2f2f73747566662e636861726d2e73682f627562626c657465612f627562626c657465612d6578616d706c652e6769663f30)](https://camo.githubusercontent.com/3e1285f61fa4de4bfa8efffb8ea55a2e62503515c9d9bdcde650c8a523eb2651/68747470733a2f2f73747566662e636861726d2e73682f627562626c657465612f627562626c657465612d6578616d706c652e6769663f30)

Here are a collection of tips I gathered over the years writing CLI tools.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#use-a-commandline-parsing-framework)Use a command-line parsing framework

While it is fun to parse `argv` by hand, I go straight for full-featured command-line parsing frameworks like [cobra](https://github.com/spf13/cobra) in go, [clap](https://github.com/clap-rs/clap) in rust,  
[click](https://click.palletsprojects.com/en/8.1.x/) in python, [php-cli](https://github.com/adhocore/php-cli) in PHP.

Not only are these well established, they often allow you to use subcommands, generate autocompletion script for multiple shells, handle pretty-printing of CLI options and link up to configuration frameworks (see below).

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#distribute-the-tool-as-a-single-file-with-no-dependencies)Distribute the tool as a single file with no dependencies

To ensure widespread adoption of your CLI tool, it must be easily installable. No one wants to install 80 node modules or god forbid, setup a python virtualenv, just to use your tool.

- Golang and Rust are perfect languages for this, as they allow you to easily compile static binaries for multiple platforms. You can also experiment with dumping world images in Common Lisp.
- Another option is to create a docker image that can be run with a single docker call, although this often makes networking and file IO more complicated.
- A compromise is to make it an easily installable package through your languages package manager (pip, npm, what have you)

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#make-documentation-part-of-the-tool-itself)Make documentation part of the tool itself

Instead of distributing the documentation as a separate wiki, make it part of your tool. Don't spit out cryptic auto-generated argument lists, but leverage your CLI framework and TUI library to provide beautiful documentation.

It should be possible to use the CLI tool to:

- lookup all the information about verbs and arguments (see the next point about examples)
- get information about the tool itself and its principles of operation
- discover the functionality provided by the tool (see the section about a building a grammar)

While I have never done so, some tools even provide full tutorials as part of their feature set!

As complex the git command line can be, it provides great documentation entries for each verb.

Consider using a pretty library to render markdown straight on the command line, for example [glamour](https://github.com/charmbracelet/glamour) (golang) or [rich](https://github.com/Textualize/rich) (python).

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#add-examples-to-your-documentation)Add examples to your documentation

The easiest way to help people get started with your tool is to provide numerous examples. This often gives a more intuitive description of arguments and how they interact. Use it as an opportunity to show certain patterns and give some background information about why a certain verb exists. Use it as an opportunity to also show complex and advanced ways of using your CLI, especially if you provide filtering and templating functionality (see below).

`git`, `aws`, `kubectl` all provide examples that I have found extremely useful.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#make-it-pretty)Make it pretty

Who doesn't like a pretty colorful CLI tool. There are a lot of terminal TUI and style management libraries out there. Don't forget to make your tool compatible to colorless terminals and IO pipes however.

[![imtui example](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fwillmcgugan%2Frich%2Fraw%2Fmaster%2Fimgs%2Ffeatures.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fgithub.com%2Fwillmcgugan%2Frich%2Fraw%2Fmaster%2Fimgs%2Ffeatures.png)

I had a great time with all the [charmbracelet](https://github.com/charmbracelet) libraries, like [bubbletea](https://github.com/charmbracelet/bubbletea) for building TUIs and [lipbgloss](https://github.com/charmbracelet/lipgloss) for styling. [rich](https://github.com/Textualize/rich) is a similar, awesome python library.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#make-it-interactive)Make it interactive

For some applications (not all), I found implementing an interactive version of the CLI tool useful. Often, I have done so when controlling hardware, where keeping context is important, and much easier in a long running application.

This can be as easy as wrapping a simple stdin/stdout loop with [rlwrap](https://github.com/hanslub42/rlwrap), all the way to using full featured TUI libraries like [bubbletea](https://github.com/charmbracelet/bubbletea) (golang), [textual](https://github.com/Textualize/textual) (python) or [imtui](https://github.com/ggerganov/imtui) (c++).

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#use-asciinema-to-record-short-tutorials)Use asciinema to record short tutorials

A great feature of command line applications is that recording a terminal session is very lightweight, and can easily be transformed into a gif or rendered into a webpage.

Tools like [asciinema](https://asciinema.org/) are great to show real-life workflows.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#carefully-craft-a-cli-grammar)Carefully craft a CLI grammar

This is what I consistently find the most difficult when building a CLI tool: designing a nice grammar. In the best of worlds, verbs and subcommands are "intuitive": if a user wants to do something, it should be easy for them to guess which command to use.

This means having a `set` command if you have a `get` command, or a proper subcommand for each resource that can be manipulated (for example `files`, `hosts`, `drives` ).

A confusing grammar would be using `get --update` to set a variable, or having a verb `ls-files` for listing files, and a `hosts list` subcommand and verb for listing hosts.

I find that I often have to rewrite the CLI verb structure a couple of times before I understand what works well for the problem I want to solve. Don't try to get it right the first time, in fact avoiding being too "frameworky" or clever the first time around. Liberally add the verbs and flags you find useful as you are building out your tool, and then do a second pass where you don't mind restarting the verbs and flags from scratch. Chances are that you build internal helper functions that can be reused.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#add-structured-input-and-output)Add structured input and output

CLIs are not just for humans, they are also extremely useful for machines. To make it easier to pipe and convert the data your tools generate, think about outputting structured data alongside human-readable data.

I have generic wrappers that generate:

- tabulated, human-readable output that is nicely formatted (and colored)
- different pre-configured human readable output options (for example `--wide`, `--pretty`, `--oneline`)
- CSV and TSV formats for easy unix tool / spreadsheet consumption
- structured JSON output (this is usually the easiest, as the objects manipulated by the CLI are often serializable out of the box)
- when applicable, dumping data out as a SQLite database is extremely useful

Similarly, make it easy to read structured input, not just from the CLI, but also from files. This makes it much easier to rerun similar commands or share big batches of data amongst team members.

For good example of these concepts, look at `kubectl` , used to managed kubernetes resources.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#add-filtering-and-aggregation-options)Add filtering and aggregation options

In conjunction with the previous concept of adding structured IO, consider adding filtering options. For example, you could give a user the option to only output 3 attributes as tabular data, or to aggregate counts by a certain other attribute.

As an alternative to implementing your own filtering and aggregating, which is not always easy, consider adding examples where you filter and process data by using tools like `jq`, `yq` or `xsv` in your CLI documentation.

[![](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fasciinema.org%2Fa%2Fcpazej888znujgm04ewzsv0mk.png)](https://asciinema.org/a/cpazej888znujgm04ewzsv0mk)

Don't forget to give copious examples (the more useful and applicable to the real-world, the better), as these are often time-consuming to come up as a user, especially when you just need the tool to get something done. Think of each useful example you give as a little nugget of pleasure that someone will experience in the future.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#add-templating-options)Add templating options

If you want to go even a step further beyond structured output, consider giving the user the option to provide output templates. Tools like `kubectl` or `aws` allow you to pass your own go-template inspired templates, making it possible to create exactly the output needed.

```


kubectl get no -o go-template='{{range .items}}{{if .spec.unschedulable}}{{.metadata.name}} {{.spec.externalID}}{{"\n"}}{{end}}{{end}}'


```

 

I always found writing templates for CLIs quite time-consuming, but extremely useful once I figured it out. So don't skimp on examples!

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#make-it-easy-to-extend-your-cli)Make it easy to extend your CLI

Make it easy to add new verbs / extend the CLI tool. A common pattern to do so is to lookup if a binary called `yourtool-XXX` exists when an unknown verb `XXX` is used, and pass control to that binary. This is how `gh` (GitHub client) and `git` operate.

Even simple shell script verbs provide great extension capability for users. For even bigger impact, provide a [simple wrapper framework](https://github.com/cli/go-gh) like `gh` does, that allows extensions written in go to easily call the main tool itself.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#make-it-easy-to-configure-your-cli)Make it easy to configure your CLI

Chances are that you will want to provide a lot of default values and configuration options in your CLI (credentials, URL endpoints, output settings).

Make it easy to configure these by providing:

- environment variables. This has the added benefit of making your tool easy to integrate in CICD, and provide security by avoiding listing credentials in your command line. It also allows for easy customization per directory or project, using a tool like [direnv](https://direnv.net/).
- configuration files. A few command line option libraries provide the option to use a configuration file alongside the command line parser (for example [viper](https://github.com/spf13/viper) in golang). Otherwise, nothing is easier to use a JSON, YAML, or TOML parser to read in defaults and overrides.
- provide verbs to query and update your config file. A lot of modern CLI tools provide verbs to interact with their own configuration. Good examples are `git config`, which provides a rich verb to query, remove, update, set the git configuration. Dumping and querying the current information (alongside version, platform, loaded environments and config files) makes it easy to debug issues.
- provide global and local configuration files. Besides providing a global configuration file, allow users to provide local configuration files. Make it easy to chain configuration files, so that you can have a global config, followed by a project-wide config (checked into git), followed by a user specific override config (not checked into git).

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#provide-autocompletion-integration)Provide autocompletion integration

Nothing is nicer for the user than just pressing tab and getting a nice autocompletion. Shells these days provide really nice UIs for autocompletion, and a rich framework to leverage these.

Besides using the builtin autocompletion generation by CLI argument parsing frameworks (see above), think about providing dynamic autocompletion options. I have used this in the past to dynamically query connected devices when providing a hardware control tool. Autocompletion would show a list of connected widgets by ID.

## [](https://dev.to/wesen/14-great-tips-to-make-amazing-cli-applications-3gp3#conclusion)Conclusion

CLIs and TUIs are incredibly effective tool for the developer. They are usually easy to write, extremely useful for the author themselves as ways to exercise their code while developing, and a great way to "self-document" and extend the functionality of the application our library.

With a bit of thought and styling, CLIs can be turned into game changers.

What about you? Do you write CLI tools? If so, what tricks have you been using? Are you into TUIs and interactivity?