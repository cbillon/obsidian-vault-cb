---
link: https://blog.tjll.net/the-best-thing-about-docker/
byline: Tyler Langlois
site: Tyblog
excerpt: I love Docker, but not for the typical reasons.
slurped: 2026-06-03T09:49
title: The Best Thing About Docker is not Docker
tags:
  - docker
---

### [«](https://blog.tjll.net/reverse-proxy-hot-dog-eating-contest-caddy-vs-nginx/) The Best Thing About Docker is not Docker [»](https://blog.tjll.net/a-modest-license-violation-proposal/)

- 27 September, 2022
- 433 words
- 1 minute read time

[Docker](https://www.docker.com/) is the _de facto_ solution for packaging most server-side applications these days. The **technical** merits of Docker are nifty – cgroups and other mechanisms are certainly useful – but there's one particular aspect of running an application in Docker that has been unequivocally beneficial for the industry: concretely defined inputs and outputs.

I was reading the [Gitea installation docs](https://docs.gitea.io/en-us/install-with-docker/) the other day when I realized _why_ I gravitate immediately to the "Installation with Docker" section _even when Docker may not be my deployment strategy_. I do so because a file like `docker-compose.yml` is a manifest that defines in certain terms how a program interacts with its environment in a concise, standard way.

When I run an application in an environment like my homelab, there's a constant list of things that I, as an operator, need to know:

- Where does the application persist data? For a container to hang on to anything it writes to disk, that needs to be expressed as a volume.
- What network services does it provide? Port forwarding rules list out each listener for me to handle either at my reverse proxy or other network layers.
- What environment variables (or, commonly, secrets) does this application expect? Most containers don't inherit _everything_ from the environment, so this list of variable names provides a convenient list of what the application needs.

Part of why each of those bullets is useful is that, without those various settings, a container is – almost by definition – an inert unit of compute and nothing more. It can only rely on persistent storage or inbound traffic _if you explicitly configure the container to do so_. Is that Gitea container leaving other files scattered around or leaving a port listener undocumented? Probably not, because otherwise the userbase would find out about broken or buggy containers pretty quickly. The container image can't receive traffic on a port that the developers have failed to document as part of a container configuration.

Frankly, this is why my gut feeling is that the kubernetes "revolution" that is eating DevOps feels marginally less significant to me than Docker itself. Sure, k8s is a nice abstraction layer over your hypervisors or container runtime, but the real meat – the juicy, high-fructose center – of what Docker brought to us was _the application manifest_. Do you feel a really gut-wrenching pull to `docker run` rather than invoking an `./app` statically-compiled binary based primarily on the runtime? I'll probably end up putting the `./app` in a cgroup'd systemd `Unit` _anyway_, but I absolutely want the _instructions to run it_ to be consistent.

---

---