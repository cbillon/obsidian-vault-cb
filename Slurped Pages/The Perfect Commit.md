---
link: https://simonwillison.net/2022/Oct/29/the-perfect-commit/
byline: Simon Willison
site: Simon Willison’s Weblog
excerpt: For the last few years I’ve been trying to center my work around creating what I consider to be the Perfect Commit. This is a single commit that contains all …
twitter: https://twitter.com/@simonw
slurped: 2025-12-19T19:21
title: The Perfect Commit
tags:
  - git
---

29th October 2022

For the last few years I’ve been trying to center my work around creating what I consider to be the _Perfect Commit_. This is a single commit that contains all of the following:

- The **implementation**: a single, focused change
- **Tests** that demonstrate the implementation works
- Updated **documentation** reflecting the change
- A link to an **issue thread** providing further context

Our job as software engineers generally isn’t to write new software from scratch: we spend the majority of our time adding features and fixing bugs in existing software.

The commit is our principle unit of work. It deserves to be treated thoughtfully and with care.

_**Update 26th November 2022**: My 25 minute talk [Massively increase your productivity on personal projects with comprehensive documentation and automated tests](https://simonwillison.net/2022/Nov/26/productivity/) describes this approach to software development in detail._

- [Implementation](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#implementation)
- [Tests](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#tests)
- [Documentation](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#documentation)
- [A link to an issue](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#link-to-an-issue)
- [An issue is more valuable than a commit message](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#issue-over-commit-message)
- [Not every commit needs to be “perfect”](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#not-all-perfect)
- [Write scrappy commits in a branch](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#scrappy-branches)
- [Some examples](https://simonwillison.net/2022/Oct/29/the-perfect-commit/#examples)

#### Implementation

Each commit should change a single thing.

The definition of “thing” here is left deliberately vague!

The goal is have something that can be easily reviewed, and that can be clearly understood in the future when revisited using tools like `git blame` or [git bisect](https://til.simonwillison.net/git/git-bisect).

I like to keep my commit history linear, as I find that makes it much easier to comprehend later. This further reinforces the value of each commit being a single, focused change.

Atomic commits are also much easier to cleanly revert if something goes wrong—or to cherry-pick into other branches.

For things like web applications that can be deployed to production, a commit should be a unit that can be deployed. Aiming to keep the main branch in a deployable state is a good rule of thumb for deciding if a commit is a sensible atomic change or not.

#### Tests

The ultimate goal of tests is to _increase_ your productivity. If your testing practices are slowing you down, you should consider ways to improve them.

In the longer term, this productivity improvement comes from gaining the freedom to make changes and stay confident that your change hasn’t broken something else.

But tests can help increase productivity in the immediate short term as well.

How do you know when the change you have made is finished and ready to commit? It’s ready when the new tests pass.

I find this reduces the time I spend second-guessing myself and questioning whether I’ve done enough and thought through all of the edge cases.

Without tests, there’s a very strong possibility that your change will have broken some other, potentially unrelated feature. Your commit could be held up by hours of tedious manual testing. Or you could YOLO it and learn that you broke something important later!

Writing tests becomes far less time consuming if you already have good testing practices in place.

Adding a new test to a project with a lot of existing tests is easy: you can often find an existing test that has 90% of the pattern you need already worked out for you.

If your project has no tests at all, adding a test for your change will be a lot more work.

This is why I start every single one of my projects with a passing test. It doesn’t matter what this test is—`assert 1 + 1 == 2` is fine! The key thing is to get a testing framework in place, such that you can run a command (for me that’s usually `pytest`) to execute the test suite—and you have an obvious place to add new tests in the future.

I use [these cookiecutter templates](https://simonwillison.net/2021/Aug/28/dynamic-github-repository-templates/) for almost all of my new projects. They configure a testing framework with a single passing test and GitHub Actions workflows to exercise it all from the very start.

I’m not a huge advocate of test-first development, where tests are written before the code itself. What I care about is tests-included development, where the final commit bundles the tests and the implementation together. I wrote more about my approach to testing in [How to cheat at unit tests with pytest and Black](https://simonwillison.net/2020/Feb/11/cheating-at-unit-tests-pytest-black/).

#### Documentation

If your project defines APIs that are meant to be used outside of your project, they need to be documented. In my work these projects are usually one of the following:

- Python APIs (modules, functions and classes) that provide code designed to be imported into other projects.
- Web APIs—usually JSON over HTTP these days—that provide functionality to be consumed by other applications.
- Command line interface tools, such as those implemented using [Click](https://click.palletsprojects.com/) or [Typer](https://typer.tiangolo.com/) or [argparse](https://docs.python.org/3/library/argparse.html).

It is critical that this documentation **must live in the same repository as the code itself**.

This is important for a number of reasons.

Documentation is only valuable **if people trust it**. People will only trust it if they know that it is kept up to date.

If your docs live in a separate wiki somewhere it’s easy for them to get out of date—but more importantly it’s hard for anyone to quickly confirm if the documentation is being updated in sync with the code or not.

Documentation should be **versioned**. People need to be able to find the docs for the specific version of your software that they are using. Keeping it in the same repository as the code gives you synchronized versioning for free.

Documentation changes should be **reviewed** in the same way as your code. If they live in the same repository you can catch changes that need to be reflected in the documentation as part of your code review process.

And ideally, documentation should be **tested**. I wrote about my approach to doing this using [Documentation unit tests](https://simonwillison.net/2018/Jul/28/documentation-unit-tests/). Executing example code in the documentation using a testing framework is a great idea too.

As with tests, writing documentation from scratch is much more work than incrementally modifying existing documentation.

Many of my commits include documentation that is just a sentence or two. This doesn’t take very long to write, but it adds up to something very comprehensive over time.

How about end-user facing documentation? I’m still figuring that out myself. I created my [shot-scraper tool](https://simonwillison.net/2022/Mar/10/shot-scraper/) to help automate the process of keeping screenshots up-to-date, but I’ve not yet found personal habits and styles for end-user documentation that I’m confident in.

#### A link to an issue

Every perfect commit should include a link to an issue thread that accompanies that change.

Sometimes I’ll even open an issue seconds before writing the commit message, just to give myself something I can link to from the commit itself!

The reason I like issue threads is that they provide effectively unlimited space for commentary and background for the change that is being made.

Most of my issue threads are me talking to myself—sometimes with dozens of issue comments, all written by me.

Things that can go in an issue thread include:

- **Background**: the reason for the change. I try to include this in the opening comment.
- **State of play** before the change. I’ll often link to the current version of the code and documentation. This is great for if I return to an open issue a few days later, as it saves me from having to repeat that initial research.
- **Links to things**. So many links! Inspiration for the change, relevant documentation, conversations on Slack or Discord, clues found on StackOverflow.
- **Code snippets** illustrating potential designs and false-starts. Use ` ```python ... ``` ` blocks to get syntax highlighting in your issue comments.
- **Decisions**. What did you consider? What did you decide? As programmers we make hundreds of tiny decisions a day. Write them down! Then you’ll never find yourself relitigating them in the future having forgotten your original reasoning.
- **Screenshots**. What it looked like before, what it looked like after. Animated screenshots are even better! I use [LICEcap](https://www.cockos.com/licecap/) to generate quick GIF screen captures or QuickTime to capture videos—both of which can be dropped straight into a GitHub issue comment.
- **Prototypes**. I’ll often paste a few lines of code copied from a Python console session. Sometimes I’ll even paste in a block of HTML and CSS, or add a screenshot of a UI prototype.

After I’ve closed my issues I like to add one last comment that links to the updated documentation and ideally a live demo of the new feature.

#### An issue is more valuable than a commit message

I went through a several year phase of writing essays in my commit messages, trying to capture as much of the background context and thinking as possible.

My commit messages grew a lot shorter when I started bundling the updated documentation in the commit—since often much of the material I’d previously included in the commit message was now in that documentation instead.

As I extended my practice of writing issue threads, I found that they were a better place for most of this context than the commit messages themselves. They supported embedded media, were more discoverable and I could continue to extend them even after the commit had landed.

Today many of my commit messages are a single line summary and a link to an issue!

The biggest benefit of lengthy commit messages is that they are guaranteed to survive for as long as the repository itself. If you’re going to use issue threads in the way I describe here it is critical that you consider their long term archival value.

I expect this to be controversial! I’m advocating for abandoning one of the core ideas of Git here—that each repository should incorporate a full, decentralized record of its history that is copied in its entirety when someone clones a repo.

I understand that philosophy. All I’ll say here is that my own experience has been that dropping that requirement has resulted in a net increase in my overall productivity. Other people may reach a different conclusion.

If this offends you too much, you’re welcome to construct an _even more perfect commit_ that incorporates background information and additional context in an extended commit message as well.

One of the reasons I like GitHub Issues is that it includes a comprehensive API, which can be used to extract all of that data. I use my [github-to-sqlite tool](https://github.com/dogsheep/github-to-sqlite) to maintain an ongoing archive of my issues and issue comments as a SQLite database file.

#### Not every commit needs to be “perfect”

I find that the vast majority of my work fits into this pattern, but there are exceptions.

Typo fix for some documentation or a comment? Just ship it, it’s fine.

Bug fix that doesn’t deserve documentation? Still bundle the implementation and the test plus a link to an issue, but no need to update the docs—especially if they already describe the expected bug-free behaviour.

Generally though, I find that aiming for implementation, tests, documentation and an issue link covers almost all of my work. It’s a really good default model.

#### Write scrappy commits in a branch

If I’m writing more exploratory or experimental code it often doesn’t make sense to work in this strict way. For those instances I’ll usually work in a branch, where I can ship “WIP” commit messages and failing tests with abandon. I’ll then squash-merge them into a single perfect commit (sometimes via a self-closed GitHub pull request) to keep my main branch as tidy as possible.

#### Some examples

Here are some examples of my commits that follow this pattern:

- [Upgrade Docker images to Python 3.11](https://github.com/simonw/datasette/commit/9676b2deb07cff20247ba91dad3e84a4ab0b00d1) for [datasette #1853](https://github.com/simonw/datasette/issues/1853)—a pretty tiny change, but still includes tests, docs and an issue link.
- [sqlite-utils schema now takes optional tables](https://github.com/simonw/sqlite-utils/commit/ab8d4aad0c42f905640981f6f24bc1e37205ae62) for [sqlite-utils #299](https://github.com/simonw/sqlite-utils/issues/299)
- [shot-scraper html command](https://github.com/simonw/shot-scraper/commit/5048e21a1ca5accedfeca6ac25a16a38dc240b81) for [shot-scraper #96](https://github.com/simonw/shot-scraper/issues/96)
- [s3-credentials put-objects command](https://github.com/simonw/s3-credentials/commit/c7bb7268c4a124349bb511f7ec3ee3f28f9581ad) for [s3-credentials #68](https://github.com/simonw/s3-credentials/issues/68)
- [Initial implementation](https://github.com/simonw/datasette-gunicorn/commit/0d561d7a94f76079b1eb7779b3e944c163d2539e) for [datasette-gunicorn #1](https://github.com/simonw/datasette-gunicorn/issues/1)—this was the first commit to this repository, but I still bundled the tests, docs, implementation and a link to an issue.