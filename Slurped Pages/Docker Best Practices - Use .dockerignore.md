---
link: https://gagor.pro/2022/09/docker-best-practices-use-.dockerignore/
byline: Tom
site: Tom's Blog
date: 2022-09-11T02:00
excerpt: Learn best practices for writing Dockerfiles by using .dockerignore to optimize build times and exclude unnecessary files from the build context.
tags:
  - docker
  - best_pratices
slurped: 2026-05-27T09:20
title: Docker Best Practices - Use .dockerignore
---

People often complain, that building Docker image takes a long time. “I just added a single `jar` package” they say… Really?

They often don’t remember that whole “build context”[1](#fn:1) is uploaded to Docker daemon during build, which often means they’re not only adding “single jar”, but also all sources, test results and whatever they have in working directory.

Solution is simple - to use `.dockerignore` file[2](#fn:2). Syntax is similar to `.gitignore`. It excludes what shouldn’t be uploaded to Docker daemon.

Take a look at an example file below:

```
src
tests
target/*.xml

# exclude temp files
*/temp*
*/*/temp*
temp?

# exclude things you won't need anyway
*~
.DS_Store
*.old
.vscode
```