---
link: https://gagor.pro/2022/09/dockerfile-writing-best-practices/
byline: Tom
site: Tom's Blog
date: 2022-09-10T02:00
excerpt: Learn best practices for writing Dockerfiles to create efficient, secure, and maintainable Docker images, based on real-world experience and insights.
tags:
  - docker
  - dockerfile
  - best_pratices
slurped: 2026-05-27T09:21
title: Dockerfile writing best practices
---
docker
I’ve been thinking for a long time about writing set of articles on the topic of: “Dockerfile writing best practices”.

As it’s often my daily job to prepare best in class containers, that are later used by thousands of company’s applications, I have quite good insights on the topic. Some experience and knowledge gathered is often against intuition and building it took me a while. I want to share it, with a hope that feedback I get will allow me to excel on the topic even further.

Initially I was thinking about writing one big article, connect all the dots there and make it great… I even started it. But I failed by it’s scale, so I changed strategy and now I plan to release them as series of smaller articles, that should be easier to deliver and maintain.

## Topics I want to cover

1. [Use .dockerignore](https://gagor.pro/2022/09/docker-best-practices-use-.dockerignore/)
2. [Follow “Filesystem Hierarchy Standard”](https://gagor.pro/2024/03/best-practices-for-writing-dockerfiles-follow-filesystem-hierarchy-standard/)
3. Don’t leave packages you don’t need in images
4. [Use VOLUME for all mutable, temporary files locations](https://gagor.pro/2022/09/docker-best-practices-use-volume-for-temporary-and-mutable-files/)
5. Don’t rely on Docker official images
6. Don’t run applications as a root
7. Use multi-stage builds
8. Use Label Schema/OCI Image Label Spcification
9. Image security, how to scan them and when
10. Build cache - to use it or not?
11. And more…

I will curate list of links to dedicated articles on this page as they will be arriving.