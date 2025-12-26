---
link: https://simonwillison.net/2023/Sep/30/cli-tools-python/
byline: Simon Willison
site: Simon Willison’s Weblog
excerpt: I build a lot of command-line tools in Python. It’s become my favorite way of quickly turning a piece of code into something I can use myself and package up …
twitter: https://twitter.com/@simonw
slurped: 2025-12-19T19:12
title: Things I’ve learned about building CLI tools in Python
tags:
  - cli
  - python
---

30th September 2023

I build a lot of command-line tools in Python. It’s become my favorite way of quickly turning a piece of code into something I can use myself and package up for other people to use too.

My biggest CLI projects are [sqlite-utils](https://sqlite-utils.datasette.io/), [LLM](https://llm.datasette.io/), [shot-scraper](https://shot-scraper.datasette.io/en/stable/) and [Datasette](https://datasette.io/)—but I have dozens of others and I build new ones at the rate of at least one a month. A fun recent example is [blip-caption](https://github.com/simonw/blip-caption), a tiny CLI wrapper around the Salesforce BLIP model that can generate usable captions for image files.

Here are some notes on what I’ve learned about designing and implementing CLI tools in Python so far.

#### Starting with a template

I build enough CLI apps that I developed my own [Cookiecutter](https://github.com/cookiecutter/cookiecutter) template for starting new ones.

That template is [simonw/click-app](https://github.com/simonw/click-app). You can create a new application from that template directly on GitHub, too—I wrote more about that in [Dynamic content for GitHub repository templates using cookiecutter and GitHub Actions](https://simonwillison.net/2021/Aug/28/dynamic-github-repository-templates/).

#### Arguments, options and conventions

Almost all of my tools are built using the [Click](https://click.palletsprojects.com/) Python library. Click encourages a specific way of designing CLI tools which I really like—I find myself annoyed at the various tools from other ecosystems that don’t stick to the conventions that Click encourages.

I’ll try to summarize those conventions here.

- Commands have arguments and options. Arguments are positional—they are strings that you pass directly to the command, like `data.db` in `datasette data.db`. Arguments can be required or optional, and you can have commands which accept an unlimited number of arguments.
- Options are, usually, optional. They are things like `--port 8000`. Options can also have a single character shortened version, such as `-p 8000`.
    - Very occasionally I’ll create an option that is required, usually because a command has so many positional arguments that forcing an option makes its usage easier to read.
- Some options are flags—they don’t take any additional parameters, they just switch something on. `shot-scraper --retina` is an example of this.
- Flags with single character shortcuts can be easily combined—`symbex -in fetch_data` is short for `symbex --imports --no-file fetch_data` [for example](https://github.com/simonw/symbex/blob/1.4/README.md#usage).
- Some options take multiple parameters. `datasette --setting sql_time_limit_ms 10000` is an example, taking both the name of the setting and the value it should be set to.
- Commands can have sub-commands, each with their own family of commands. [llm templates](https://llm.datasette.io/en/stable/templates.html) is an example of this, with `llm templates list` and `llm templates show` and [several more](https://llm.datasette.io/en/stable/help.html#llm-templates-help).
- Every command should have help text—the more detailed the better. This can be viewed by running `llm --help`—or for sub-commands, `llm templates --help`.

Click makes it absurdly easy and productive to build CLI tools that follow these conventions.

#### Consistency is everything

As CLI utilities get larger, they can end up with a growing number of commands and options.

The most important thing in designing these is _consistency_ with other existing commands and options ([example here](https://github.com/simonw/llm/issues/160#issuecomment-1682995315))—and with related tools that your user may have used before.

I often turn to GPT-4 for help with this: I’ll ask it for examples of existing CLI tools that do something similar to what I’m about to build, and see if there’s anything in their option design that I can emulate.

Since my various projects are designed to complement each other I try to stay consistent between them as well—I’ll often post an issue comment that says “similar to functionality in X”, with a copy of the `--help` output for the tool I’m about to imitate.

#### CLI interfaces are an API—version appropriately

I try to stick to [semantic versioning](https://semver.org/) for my projects, bumping the major version number on breaking changes and the minor version number for new features.

The command-line interface to a tool is absolutely part of that documented API. If someone writes a Bash script or a GitHub Actions automation that uses one of my tools, I’m cautious to avoid breaking that without bumping my major version number.

#### Include usage examples in --help

A habit I’ve formed more recently is trying to always including a working example of the command in the `--help` for that command.

I find I use this a lot for tools I’ve developed myself. All of my tools have extensive online documentation, but I like to be able to consult `--help` without opening a browser for most of their functionality.

Here’s one of my more involved examples—the help for the [sqlite-utils convert](https://sqlite-utils.datasette.io/en/stable/cli.html#converting-data-in-columns) command:

```
Usage: sqlite-utils convert [OPTIONS] DB_PATH TABLE COLUMNS... CODE

  Convert columns using Python code you supply. For example:

      sqlite-utils convert my.db mytable mycolumn \
          '"\n".join(textwrap.wrap(value, 10))' \
          --import=textwrap

  "value" is a variable with the column value to be converted.

  Use "-" for CODE to read Python code from standard input.

  The following common operations are available as recipe functions:

  r.jsonsplit(value, delimiter=',', type=<class 'str'>)

      Convert a string like a,b,c into a JSON array ["a", "b", "c"]

  r.parsedate(value, dayfirst=False, yearfirst=False, errors=None)

      Parse a date and convert it to ISO date format: yyyy-mm-dd
      
      - dayfirst=True: treat xx as the day in xx/yy/zz
      - yearfirst=True: treat xx as the year in xx/yy/zz
      - errors=r.IGNORE to ignore values that cannot be parsed
      - errors=r.SET_NULL to set values that cannot be parsed to null

  r.parsedatetime(value, dayfirst=False, yearfirst=False, errors=None)

      Parse a datetime and convert it to ISO datetime format: yyyy-mm-ddTHH:MM:SS
      
      - dayfirst=True: treat xx as the day in xx/yy/zz
      - yearfirst=True: treat xx as the year in xx/yy/zz
      - errors=r.IGNORE to ignore values that cannot be parsed
      - errors=r.SET_NULL to set values that cannot be parsed to null

  You can use these recipes like so:

      sqlite-utils convert my.db mytable mycolumn \
          'r.jsonsplit(value, delimiter=":")'

Options:
  --import TEXT                   Python modules to import
  --dry-run                       Show results of running this against first
                                  10 rows
  --multi                         Populate columns for keys in returned
                                  dictionary
  --where TEXT                    Optional where clause
  -p, --param <TEXT TEXT>...      Named :parameters for where clause
  --output TEXT                   Optional separate column to populate with
                                  the output
  --output-type [integer|float|blob|text]
                                  Column type to use for the output column
  --drop                          Drop original column afterwards
  --no-skip-false                 Don't skip falsey values
  -s, --silent                    Don't show a progress bar
  --pdb                           Open pdb debugger on first error
  -h, --help                      Show this message and exit.
```

#### Including --help in the online documentation

My larger tools tend to have extensive documentation independently of their help output. I update this documentation at the same time as the implementation and the tests, as described in [The Perfect Commit](https://simonwillison.net/2022/Oct/29/the-perfect-commit/).

I like to include the `--help` output in my documentation sites as well. This is mainly for my own purposes—having the help visible on a web page makes it much easier to review it and spot anything that needs updating.

Here are some example pages from my documentation that list `--help` output:

- [sqlite-utils CLI reference](https://sqlite-utils.datasette.io/en/stable/cli-reference.html)
- [LLM CLI reference](https://llm.datasette.io/en/stable/help.html)
- [Datasette CLI reference](https://docs.datasette.io/en/stable/cli-reference.html)
- `shot-scraper` embeds help output on the relevant pages, e.g. [shot-scraper shot --help](https://shot-scraper.datasette.io/en/stable/screenshots.html#shot-scraper-shot-help)
- [s3-credentials command help](https://s3-credentials.readthedocs.io/en/stable/help.html)

All of these pages are maintained automatically using [Cog](https://github.com/nedbat/cog). I described the pattern I use for this in [Using cog to update --help in a Markdown README file](https://til.simonwillison.net/python/cog-to-update-help-in-readme), or you can [view source](https://github.com/simonw/datasette/blob/1.0a7/docs/cli-reference.rst?plain=1) on the Datasette CLI reference for a more involved example.