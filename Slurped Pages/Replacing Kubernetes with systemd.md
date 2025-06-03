---
link: https://blog.yaakov.online/replacing-kubernetes-with-systemd/
byline: Yaakov
site: Yaakov's Blog
date: 2024-02-04T12:15
excerpt: In this post I go through the journey of overusing Kubernetes and how systemd can actually do most of what I use it for.
twitter: https://twitter.com/@yaakov_h
slurped: 2025-05-28T18:47
title: Replacing Kubernetes with systemd
tags:
  - kubernetes
  - systemd
  - docker
  - podman
---

Yes, I'm fully aware those are two separate things, but hear me out here for a moment.

Back in 2018 I was hearing a lot of stuff from all angles and all sorts of friends and influences about Kubernetes, and from what I heard it seemed like a pretty promising piece of kit to use. At the time, I actually went out and bought a NUC to act as a little hypervisor so that I could play around with a small cluster at home.

Funnily enough, my blog post on this was [six years ago to the very day](https://blog.yaakov.online/learning-kubernetes-at-home/).

The main lesson that I learned is that although Kubernetes is made up of all sorts of pieces and web services and sidecars and webhooks, basically acts as a giant `while` loop as follows:

```
while (true)
{
    var current_state = GetCurrentState();
    var desired_state = GetDesiredState();
    var diff = CalculateDiff(current_state, desired_state);
    ApplyDiff(diff);
}
```

If I said there should be a Pod here, and there wasn't, Kubernetes would create it. If I said there should be 3 replicas and there were 4, Kubernetes would get rid of one.

This actually extended out in really cool ways, such as with [cert-manager](https://cert-manager.io/?ref=blog.yaakov.online). If I said there should be a valid TLS certificate for some domain, and told Kubernetes how it could request one, then if the certificate was missing or expiring, Kubernetes would go out and get a new certificate and install it in the web server automagically.

But as the memes go, what I was using Kubernetes for was fun to experiment with, but total overkill.

Whilst most problems I encountered did provide a legitimate learning experience, it turns out that Kubernetes, particularly on a NUC, is not bedroom-friendly. Kubernetes chews through a lot of resources, and `while (true)` loops tend to chew through a lot of CPU. This made my computers run constantly, run hot, keep the fan running, and made it hotter and noisier and harder to sleep.

Even in the cloud, this effect gets felt in different ways. My personal experience on Azure Kubernetes Service was that I immediately lose a massive chunk of RAM to their Kubernetes implementation, and it uses about 7-10% idle CPU on worker nodes. Even with single-instance [Microk8s](https://microk8s.io/?ref=blog.yaakov.online) on a small VPS I had an idle CPU load hovering around 12% on a 2x vCPU x86_64 box, and [K3S](http://k3s.io/?ref=blog.yaakov.online) which is supposed to be leaner is at about 6% constant CPU consumption on a 2x vCPU Ampere A1 machine.

(No guesses as to which cloud provider that latter one is running on.)

I even tried running Kubernetes on a Raspberry Pi but I couldn't actually find an implementation that would happily run without kicking up heat/fans and that would actually leave enough CPU behind for my workloads.

Yet the thing that kept bringing me back was the automation. Particularly with GitOps and [Flux](https://www.weave.works/oss/flux/?ref=blog.yaakov.online), making changes was a breeze. With the container image automation and recent addition of [webhooks in Flux v2](https://fluxcd.io/flux/guides/webhook-receivers/?ref=blog.yaakov.online), all I had to do was push a new container image and within seconds my servers had pulled the new container image and were running the new version of the application in production.

Every so often I would stick my head back out of Kubernetes and look at the rest of the world and see if there is something else that can do the same container automation and just like Noah's dove, I would come back empty-handed.

The only solutions I could find were "just recreate the whole container with all of the original command line arguments" as though I have the patience to manage that or remember each flag, or "here is some magic goop that works if you give it full control of `docker.sock`" which I never really liked the idea of.

I have even been sorely tempted to build my own thing but surely, _surely_, there is something out there already that does this, right?

Well, recently, I came across [Podman auto-updating](https://docs.podman.io/en/latest/markdown/podman-auto-update.1.html?ref=blog.yaakov.online). The simplest way to explain [Podman](https://podman.io/?ref=blog.yaakov.online) is an alternative Docker CLI (yes I know that is oversimplified), but it has one particular feature that caught my eye.

Once you create a container, Podman can automatically [generate a systemd service](https://docs.podman.io/en/latest/markdown/podman-generate-systemd.1.html?ref=blog.yaakov.online) file to start and stop that container. Starting the service creates (or replaces) the container, and stopping the service removes the container. So that already takes care of my "manage each original flag" problem.

But the cherry on top if that if you tag your containers with `io.containers.autoupdate`, then once a day on a timer or on-demand when you `podman auto-update`, it will check for a new image and if there is one it will recreate the container for you with the new image.

[This article from Fedora Magazine](https://fedoramagazine.org/auto-updating-podman-containers-with-systemd/?ref=blog.yaakov.online) basically gave me 99% of the magic sauce. There were only two more components I needed to make this work:

1. Run `systemctl --user enable mycontainer.service` to make the container start up automatically, whenever I log in.
2. Run `loginctl enable-linger` so that I "log in" when the server starts up.

These three components - Podman, systemd, and user lingering, now give me 99% of the benefit I was getting from Kubernetes with vastly reduced complexity and none of the CPU/memory hits associated with it.

I've migrated a full set of services from one VPS to a new one with half the vCPUs and RAM. It's only been running for a handful of hours so far, but I can see that it is running significantly lighter, snappier, and with a lower compute cost to boot, which gives me higher service density and more bang for my buck.

Of course, as my luck would have it, Podman integration with systemd appears to be deprecated already and they're now talking about defining containers in "Quadlet" files, whatever those are. I guess that will be something to learn some other time.