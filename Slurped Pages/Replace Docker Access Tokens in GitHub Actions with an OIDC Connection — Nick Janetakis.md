---
link: https://nickjanetakis.com/blog/replace-docker-access-tokens-in-github-actions-with-an-oidc-connection
byline: Nick Janetakis
site: Nick Janetakis
date: 2026-09-22T02:00
excerpt: Avoid creating and managing long-lived tokens by setting this up. You
  can have fine grained control over the rules too.
twitter: https://twitter.com/@nickjanetakis
slurped: 2026-10-20T08:53
title: Replace Docker Access Tokens in GitHub Actions with an OIDC Connection —
  Nick Janetakis
---

Updated on September 22, 2026 in [#docker](app://obsidian.md/blog/tag/docker-tips-tricks-and-tutorials)

![replace-docker-access-tokens-in-github-actions-with-an-oidc-connection.jpg](app://obsidian.md/assets/blog/cards/replace-docker-access-tokens-in-github-actions-with-an-oidc-connection-7b1c001fb6de5ef7c51a39040d21094250077a7a7b054bbf5bbbbc93da93139b.jpg)

## Avoid creating and managing long-lived tokens by setting this up. You can have fine grained control over the rules too.

**Quick Jump:**

- - [Demo Video](#demo-video)

I put together an end to end example of this here [https://github.com/nickjj/docker-oidc-example](https://github.com/nickjj/docker-oidc-example). It covers creating a GitHub repo, Docker repo, setting up the Docker OIDC connection, building [multi-architecture Docker images](app://obsidian.md/blog/build-git-tagged-multi-arch-docker-images-with-github-actions) in a GitHub Action, publishing those images to your Docker repo and testing it all end to end.

It’ll work with public or private repos, both on GitHub or the Docker Hub.

The video below shows everything.

### [#](#demo-video) Demo Video

#### Timestamps

- 0:52 – We’ll do everything in this video
- 1:39 – Creating a repo on GitHub and Docker Hub
- 3:41 – Details and locally testing it
- 5:52 – Create the Docker OIDC connection
- 12:38 – Adding a few GitHub repo variables
- 14:35 – Going over the GitHub Action
- 16:16 – Testing it out end to end
- 18:30 – Pulling it locally as a final test
- 19:39 – Making everything open source

**Are you going to be using this Docker feature? Let me know below.**

Like you, I'm super protective of my inbox, so don't worry about getting spammed. You can expect a few emails per year (at most), and you can 1-click unsubscribe at any time. [See what else you'll get](app://obsidian.md/newsletter) too.

---

---

## Comments

---