---
link: https://simonwillison.net/series/open-source-process/
byline: Simon Willison
site: Simon Willison’s Weblog
excerpt: Articles about the process I use for developing my open source projects.
slurped: 2025-12-19T19:14
title: "Simon Willison: My open source process"
tags:
---

Articles about the process I use for developing my open source projects.

[Atom feed](https://simonwillison.net/series/open-source-process.atom "Atom feed")

### [Documentation unit tests](https://simonwillison.net/2018/Jul/28/documentation-unit-tests/)

_Or: Test-driven documentation._

[... [1,521 words](https://simonwillison.net/2018/Jul/28/documentation-unit-tests/)]

### [How to cheat at unit tests with pytest and Black](https://simonwillison.net/2020/Feb/11/cheating-at-unit-tests-pytest-black/)

I’ve been making a lot of progress on [Datasette Cloud](https://simonwillison.net/tags/datasettecloud/) this week. As an application that provides private hosted Datasette instances (initially targeted at data journalists and newsrooms) the majority of the code I’ve written deals with permissions: allowing people to form teams, invite team members, promote and demote team administrators and suchlike.

[... [933 words](https://simonwillison.net/2020/Feb/11/cheating-at-unit-tests-pytest-black/)]

### [Open source projects: consider running office hours](https://simonwillison.net/2021/Feb/19/office-hours/)

Back in December I decided to try something new for my [Datasette](https://datasette.io/) open source project: [Datasette Office Hours](https://calendly.com/swillison/datasette-office-hours). The idea is simple: anyone can book a 25 minute conversation with me on a Friday to talk about the project. I’m interested in talking to people who are using Datasette, or who are considering using it, or who just want to have a chat.

[... [786 words](https://simonwillison.net/2021/Feb/19/office-hours/)]

### [How to build, test and publish an open source Python library](https://simonwillison.net/2021/Nov/4/publish-open-source-python-library/)

At [PyGotham](https://2021.pygotham.tv/talks/how-to-build-test-and-publish-an-open-source-python-library/) this year I presented a ten minute workshop on how to package up a new open source Python library and publish it to [the Python Package Index](https://pypi.org/). Here is [the video](https://www.youtube.com/watch?v=VMnLXynUqys) and accompanying notes, which should make sense even without watching the talk.

[... [2,055 words](https://simonwillison.net/2021/Nov/4/publish-open-source-python-library/)]

### [How I build a feature](https://simonwillison.net/2022/Jan/12/how-i-build-a-feature/)

I’m maintaining [a lot of different projects](https://github.com/simonw/simonw/blob/main/releases.md) at the moment. I thought it would be useful to describe the process I use for adding a new feature to one of them, using the new [sqlite-utils create-database](https://sqlite-utils.datasette.io/en/stable/cli.html#cli-create-database) command as an example.

[... [2,850 words](https://simonwillison.net/2022/Jan/12/how-i-build-a-feature/)]

### [Writing better release notes](https://simonwillison.net/2022/Jan/31/release-notes/)

Release notes are an important part of the open source process. I’ve been thinking about these a lot recently, and I’ve assembled some thoughts on how to do a better job with them.

[... [918 words](https://simonwillison.net/2022/Jan/31/release-notes/)]

### [Software engineering practices](https://simonwillison.net/2022/Oct/1/software-engineering-practices/)

Gergely Orosz [started a Twitter conversation](https://twitter.com/GergelyOrosz/status/1576161504260657152) asking about recommended “software engineering practices” for development teams.

[... [1,557 words](https://simonwillison.net/2022/Oct/1/software-engineering-practices/)]

### [The Perfect Commit](https://simonwillison.net/2022/Oct/29/the-perfect-commit/)

For the last few years I’ve been trying to center my work around creating what I consider to be the _Perfect Commit_. This is a single commit that contains all of the following:

[... [2,061 words](https://simonwillison.net/2022/Oct/29/the-perfect-commit/)]

### [Coping strategies for the serial project hoarder](https://simonwillison.net/2022/Nov/26/productivity/)

[![Visit Coping strategies for the serial project hoarder](https://static.simonwillison.net/static/2022/djangocon-productivity/productivity.001.jpeg)](https://simonwillison.net/2022/Nov/26/productivity/)

I gave a talk at DjangoCon US 2022 in San Diego last month about productivity on personal projects, titled “Massively increase your productivity on personal projects with comprehensive documentation and automated tests”.

[... [3,865 words](https://simonwillison.net/2022/Nov/26/productivity/)]

### [Things I’ve learned about building CLI tools in Python](https://simonwillison.net/2023/Sep/30/cli-tools-python/)

I build a lot of command-line tools in Python. It’s become my favorite way of quickly turning a piece of code into something I can use myself and package up for other people to use too.

[... [1,235 words](https://simonwillison.net/2023/Sep/30/cli-tools-python/)]

### [Publish Python packages to PyPI with a python-lib cookiecutter template and GitHub Actions](https://simonwillison.net/2024/Jan/16/python-lib-pypi/)

[![Visit Publish Python packages to PyPI with a python-lib cookiecutter template and GitHub Actions](https://static.simonwillison.net/static/2024/template-repo-create.jpg)](https://simonwillison.net/2024/Jan/16/python-lib-pypi/)

I use [cookiecutter](https://github.com/cookiecutter/cookiecutter) to start almost all of my Python projects. It helps me quickly generate a skeleton of a project with my preferred directory structure and configured tools.

[... [686 words](https://simonwillison.net/2024/Jan/16/python-lib-pypi/)]

### [A selfish personal argument for releasing code as Open Source](https://simonwillison.net/2025/Jan/24/selfish-open-source/)

[![Visit A selfish personal argument for releasing code as Open Source](https://static.simonwillison.net/static/2025/E_236_Podcast_Title.jpg)](https://simonwillison.net/2025/Jan/24/selfish-open-source/)

I’m the guest for the most recent episode of the Real Python podcast with Christopher Bailey, talking about [Using LLMs for Python Development](https://realpython.com/podcasts/rpp/236/). We covered a _lot_ of other topics as well—most notably my relationship with Open Source development over the years.

[... [464 words](https://simonwillison.net/2025/Jan/24/selfish-open-source/)]

### [Your job is to deliver code you have proven to work](https://simonwillison.net/2025/Dec/18/code-proven-to-work/)

In all of the debates about the value of AI-assistance in software development there’s one depressing anecdote that I keep on seeing: the junior engineer, empowered by some class of LLM tool, who deposits giant, untested PRs on their coworkers—or open source maintainers—and expects the “code review” process to handle the rest.

[... [840 words](https://simonwillison.net/2025/Dec/18/code-proven-to-work/)]