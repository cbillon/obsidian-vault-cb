---
link: https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/
site: slecache
excerpt: The first time I heard about `just`, I wondered what its added value was compared to the alias command. Then, after a second reminder, it clicked!
twitter: https://twitter.com/@slecache
slurped: 2025-05-17T09:53
title: Just - a runner for tasks or project-specific commands
tags:
  - alias
  - tips
  - command
---

Looking for an efficient way to execute **project-specific commands** without any hassle? Look no further! Introducing `just`, a powerful task runner that simplifies your development workflow. Whether you need **shortcuts to frequently used commands** or want to manage project-specific tasks, `just` is here to make your life easier. In this article, we’ll explore the features of `just` and its usefulness in **enhancing your productivity**. Get ready to discover how `just` can help you efficiently handle your projects with ease.

## 💡 Introduction [#](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#%F0%9F%92%A1-introduction)

The first time I heard about [just](https://just.systems/)[[1]](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#fn1), I wondered what its added value was compared to the `alias` command. Then, after a second reminder[[2]](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#fn2), it clicked!

This kind of tool allows you to create **shortcuts to tasks that are specific to a project**. Please note that this tool is not intended to replace a project lifecycle management tool like Maven, Gradle, or NPM. Instead, it is designed to quickly access commands that are regularly executed on a project.

Furthermore, if like me, you use multiple computers, you have to maintain `alias` on all of these machines. With `just`, all the shortcuts are committed to the project. There’s no need to copy the `alias` from one machine to another anymore.

So, if you find yourself typing the same commands over and over again, I recommend taking a look at this type of tool, as it will increase your productivity.

## 🧑‍🎓 Solutions [#](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#%F0%9F%A7%91%E2%80%8D%F0%9F%8E%93-solutions)

After understanding the concept of task runner and being convinced of its usefulness, I looked for the tool that best suited my needs.

I didn’t conduct an in-depth analysis, but here are the 2 solutions I tried.

### Just [#](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#just)

Following Guillaume’s recommendation, I first tried [just](https://github.com/casey/just). This command-line tool is multi-OS (MS Windows, GNU/Linux & Mac OS).

The configuration of tasks, called “recipes”, is done in a `.justfile` file. Its format is inspired by `make`.

I used this `justfile` to evaluate the tool:

```

alias cs := chuck-search

default:
  just --list

# display hello world
hello:
    echo "Hello world!"

# display a random Chuck Norris fact
@chuck:
    curl -s https://api.chucknorris.io/jokes/random|jq -Cr .value

# search a Chuck Norris fact
chuck-search QUERY:
    curl -s https://api.chucknorris.io/jokes/search?query=|jq -Cr .result[0].value

# list categories of Chuck Norris facts
chuck-categories:
    curl -s https://api.chucknorris.io/jokes/categories|jq -Cr .[]

# display all given arguments
args *ARGS:
    node -e "console.log(process.argv)" 
```

- The `cs` alias is a shortcut for the `chuck-search` recipe.
- The default task lists all recipes when executing the `just` command without arguments.
- A `@` before the name of a command hides the executed commands.
- There are 5 commands:
    - `hello` displays the classic “Hello world!”
    - `chuck` returns a random Chuck Norris fact
    - `chuck-search` returns a Chuck Norris fact containing the keyword passed as a parameter
    - `chuck-categories` lists the categories of Chuck Norris facts
    - `args` displays all passed arguments using Node.js

Therefore, if I want to display a joke about “computers”, I simply type `just cs computer` and I get this response:

```
curl -s https://api.chucknorris.io/jokes/search?query=computer|jq -Cr .result[0].value
Chuck Norris doesn't use a computer because a computer does everything slower than Chuck Norris.
```

I didn’t need to configure commands with environment variables, but `just` can load them from a `.env` file.

If you want to learn more about the capabilities of `just`, I recommend reading the [cheat sheet](https://cheatography.com/linux-china/cheat-sheets/justfile/).

### Maid [#](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#maid)

Being used to writing everything in Markdown files, including all useful commands, I became interested in the tool [Maid](https://github.com/egoist/maid).

This tool is written in Node.js and is therefore multi-OS (MS Windows, GNU/Linux, Mac OS). Its principle is very similar to `just`, with the difference that tasks are declared in a Markdown file named `maidfile.md`.

Here is the `maidfile.md` file that allowed me to test `maid`:

````

# Runbook / How to

## hello

It displays a hello world.

```bash
echo "Hello world!"
```

## chuck

Display a random Chuck Norris fact.

![Chuck Norris](https://api.chucknorris.io/img/chucknorris_logo_coloured_small@2x.png)

```bash
curl -s https://api.chucknorris.io/jokes/random|jq -Cr .value
```

## chuck:search

Search a Chuck Norris fact.

```bash
curl -s https://api.chucknorris.io/jokes/search?query=$1|jq -Cr .result[0].value
```

## chuck:categories

List categories of Chuck Norris facts.

```bash
curl -s https://api.chucknorris.io/jokes/categories|jq -Cr .[]
```

## args

Display all given arguments.

```js
console.log(process.argv)
```
````

You can list all tasks with the `maid help` command. So, if I want to display a joke about “computers”, I simply type `maid chuck:search computer` and I get this response:

```
[00:35:36] Starting 'chuck:search'...
Chuck Norris doesn't use a computer because a computer does everything slower than Chuck Norris.
[00:35:36] Finished 'chuck:search' after 318 ms...
```

## 👍 Conclusion [#](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#%F0%9F%91%8D-conclusion)

After testing these two tools, which one is better suited to my use cases?

I really like the idea of being able to write my tasks in Markdown files. In fact, I’ve been doing that for a long time already. It allows me to better document my tasks.

However, `just` stands out for its offered features. Here are the tested features that I needed:

|Feature|Just|Maid|
|---|---|---|
|Arguments|Defined in the recipe|Appended at the end of the command|
|Alias|✔️|✖️|
|Accessible from subdirectories|✔️|✖️|
|Multi-line description|✖️|✔️|
|Task chaining|✔️|[Syntax in the description](https://github.com/egoist/maid#run-tasks-beforeafter-a-task) or [hooks](https://github.com/egoist/maid#task-hooks)|
|Quiet mode|Defined in the recipe|CLI parameter (`--quiet`)|

Except for multi-line description, `just` supports all the features I needed.

Do the examples of tasks used not seem realistic to you? Here’s a real example of a `.justfile` on one of my projects:

```

alias gst := git-st
alias gcd := git-ci-dt

# List all files to commit with last update date
@git-st:
    git status -s | while read mode file; do echo "${mode}" $(stat -c %y "${file}") "${file}"; done|sort -k1,4

# Commit at a given COMMIT_DATE with a COMMENT
@git-ci-dt COMMIT_DATE COMMENT:
    GIT_COMMITTER_DATE="{{COMMIT_DATE}}" git commit --date="{{COMMIT_DATE}}" -m "{{COMMENT}}"
```

The 2 commands are:

- `gst` (`git-st`) lists all modified files with their last update date.
- `gcd` (`git-ci-dt`) creates a git commit at a given date with a specific comment.

In conclusion, `just` has joined my arsenal of tools to increase my productivity. Once again, I recommend exploring this category of tools, which are **task runners** or **project-specific command executors**.

---

1. Guillaume Laforge shared his [feedback](https://glaforge.dev/posts/2023/06/07/just-a-handy-command-line-tool/). [↩︎](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#fnref1)
    
2. [Listen to LCC 297 podcast (in French)](https://lescastcodeurs.com/2023/06/12/lcc-297-lockless-design/). [↩︎](https://slecache.com/posts/just-a-runner-for-tasks-or-project-specific-commands/#fnref2)