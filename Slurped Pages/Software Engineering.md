---
link: https://jo-m.ch/software_engineering/
site: jo-m.ch
excerpt: "Software Engineering Principles # Start simple and iterate, you won’t
  get it right the first time anyways. Make it fail gracefully. There can never
  be enough logging, debug statements, asserts. Measure before you optimize.
  Make it hard to do the wrong thing. Ugly hacks keep the world spinning.
  Limitations are as important as features. Magic is bad. Hyrums Law is very
  real and needs to be actively worked against. Specifications are important. If
  someone wants you to build something, it needs to be specified. Documents #
  Design and Decision # Should contain:"
slurped: 2026-04-19T15:27
title: Software Engineering
---

## Principles [#](#principles)

- Start simple and iterate, you won’t get it right the first time anyways.
- Make it fail gracefully.
- There can never be enough logging, debug statements, asserts.
- Measure before you optimize.
- Make it hard to do the wrong thing.
- Ugly hacks keep the world spinning.
- Limitations are as important as features.
- Magic is bad.
- [Hyrums Law](https://www.hyrumslaw.com/) is very real and needs to be actively worked against.
- Specifications are important. If someone wants you to build something, it needs to be specified.

## Documents [#](#documents)

### Design and Decision [#](#design-and-decision)

Should contain:

- Author, reviewers, date
- Intro, abstract
- Goals and scope
- Proposed solution/decision
- Implementation steps

Can contain:

- Problem analysis
- Context
- Non-goals
- Existing solution
- Alternative solutions considered
- Cost
- Evaluation
- Resources
- Open questions, to do, notes
- Future enhancements

### Testing/Experimentation Plan [#](#testingexperimentation-plan)

Write before the experiment:

- Author, reviewers, date
- Intro, abstract
- Goals and desired outcome
- Data to collect
- Equipment and prerequisites needed
- Procedure

Write during and after the experiment:

- Execution log
- Deviations from procedure
- Results, analysis
- Discussion

**Keep in mind, data analysis and discussion will take time as well - usually more than the test itself.**

### Inline Documentation [#](#inline-documentation)

Inline code documentation on classes/methods/data needs the following info:

- Cardinality
- Input/output examples
- Run time, O(…)
- Data storage, ownership
- Life cycle
- Blocking vs. async
- Naming

### Pull requests and Git Workflow [#](#pull-requests-and-git-workflow)

- As small as possible
- Squash-merge only
- First submit - first merge
- Never merge a PR manually on your own machine
- Only the person who submitted may merge

## Various [#](#various)

### Equivalent Dockerignore/Gitignore [#](#equivalent-dockerignoregitignore)

`.gitignore`

```
# dependencies
node_modules
.pnp
.pnp.js

# misc
.vscode
.DS_Store
```

`.dockerignore`

```
# dependencies
**/node_modules
**/.pnp
**/.pnp.js

# misc
**/.vscode
**/.DS_Store
**/Dockerfile
**/.dockerignore
**/docker-compose.yml
**/README.md
```

### Creating a private fork on Github [#](#creating-a-private-fork-on-github)

```
git clone git@github.com:org/repo.git
cd repo
git remote rm origin
git remote add origin git@github.com:private-org/repo.git
git remote add upstream git@github.com:org/repo.git
git push --all
git push --tags
git fetch upstream
git checkout -b upstream-master upstream/master
git checkout master
git checkout -b my-master
git push --set-upstream origin my-master
```

Updating from upstream:

```
git checkout upstream-master
git pull
git checkout -b master origin/master # first time only
git checkout master
git merge upstream-master
git checkout my-master
git merge upstream-master
git push --all
```

### Ruin somebodys day (in C) [#](#ruin-somebodys-day-in-c)

Sneak into his headers:

```
# define struct union
# define else
```

### Python Webservers [#](#python-webservers)

**WSGI:** Like GCI for Python. PEP 333/3333. Implementation:

```
# application side
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/plain')])
    yield b'Hello, World!\n'
```

Many application frameworks support this interface, e.g. Flask, Django.

**ASGI:** Like WSGI, but for async servers and apps.

```
# run: `uvicorn example:HelloWorld`
class HelloWorld:
    def __init__(self, scope):
        pass

    async def __call__(self, receive, send):
        await send({
            'type': 'http.response.start',
            'status': 200,
            'headers': [
                [b'content-type', b'text/plain'],
            ]
        })
        await send({
            'type': 'http.response.body',
            'body': b'Hello, world!',
        })
```

> ASGI should help enable an ecosystem of Python web frameworks that are highly competitive against Node and Go in terms of achieving high throughput in IO-bound contexts. It also provides support for HTTP/2 and WebSockets, which cannot be handled by WSGI.

**Projects:**

- FastAPI - API framework built on top of Starlette
- Starlette - ASGI framework/toolkit
- Gunicorn - WSGI HTTP server. Pre-fork worker, like Ruby Unicorn. Has Sync and Async workers.
- Uvicorn - ASGI HTTP server. “Uvicorn currently supports HTTP/1.1 and WebSockets. Support for HTTP/2 is planned.”

### Lon/Lat [#](#lonlat)

- Latitude: [-90, 90]°, +=North, -=South
- Longitude: [-180, 180]°, -=West, +=East

Google Maps uses (Lat, Lon).

Zurich is at Lat 47 North, Lon 8 East.