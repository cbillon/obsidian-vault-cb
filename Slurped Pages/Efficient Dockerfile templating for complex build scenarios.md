---
link: https://gagor.pro/2025/01/efficient-dockerfile-templating-for-complex-build-scenarios/
byline: Tom
site: Tom's Blog
date: 2025-01-01T01:00
excerpt: "Why even consider templating Dockerfiles? Dockerfiles revolutionized the industry with their simplicity. Each instruction creates a new layer in the image, which is automatically cached. This process integrates well with SCM, where you &ldquo;commit&rdquo; the results of one stage and move forward with other changes. The process can be easily parameterized with ARG instructions, similar to ENV but provided during the build. This allows for creating highly flexible builds. For most users, this is more than sufficient. However, there&rsquo;s a notable exception: Docker base images."
tags:
  - docker
  - dockerfile
slurped: 2026-02-24T10:49
title: Efficient Dockerfile templating for complex build scenarios
---

## Why even consider templating Dockerfiles?

Dockerfiles revolutionized the industry with their simplicity. Each instruction creates a new layer in the image, which is automatically cached. This process integrates well with SCM, where you “commit” the results of one stage and move forward with other changes. The process can be easily parameterized with `ARG` instructions, similar to `ENV` but provided during the build. This allows for creating highly flexible builds. For most users, this is more than sufficient. However, there’s a notable exception: Docker base images.

Whether you’re working on public base images or preparing bases for your enterprise, they differ from typical app images. App images rely on bases to deploy your single app with consistent quality, stable builds, and no surprises from breaking changes in the base images. Typically, you select the runtime version you need, for example, Tomcat 10.1.34, and then use the Docker image `tomcat:10.1.34-jdk21`. If you’re brave, you might choose `tomcat:10.1-jdk21`, which provides some Tomcat upgrades, or even `tomcat:10-jdk21`. Currently, Tomcat supports three major versions: 9, 10, and 11. Your apps might use any of them and they all have to be provided with stable and secure dependencies. Check [Docker Hub page for the Tomcat image](https://hub.docker.com/_/tomcat) to see how many of tags they have to provide for people’s convenience.

From the perspective of someone responsible for base images, how can you support this? Tomcat upgrades are owned by development teams because they might need to upgrade dependent libraries or configurations. But maintaining a stable and patched base image is usually the responsibility of an infra/SRE team. Keeping “the platform” patched and stable might require updating OS packages in images, even if developers don’t pay attention to that process. If the Dev team stays on an older version of Tomcat, that’s their risk, but if you keep images with unpatched `openssl`, it becomes “a platform problem”. Directives like NIS2[1](#fn:1) make enterprises accountable for not keeping the application stack updated, but upgrades of Docker images are tricky because you might need to support:

- Different OS, like Ubuntu and Alpine,
- Different Tomcat versions,
- Different Java versions,
- Configuration differences,
- Different package names, etc.

Let’s say you support 2 different Ubuntu versions + 1 Alpine, up to 3 versions of Tomcat on 3 LTS versions of Java, resulting in `(2+1)*3*3 -> 27` combinations. Supporting up to 5 versions of Tomcat results in 45 combinations. It’s growing exponentially!

graph LR
    A(tomcat) --> T9(9)
    T9 --> J11(Java 11)
    T9 --> J17(Java 17)
    T9 --> J21(Java 21)
    J11 --> U1(Ubuntu 24.04)
    J11 --> U2(Ubuntu 22.04)
    J11 --> A1(Alpine 3.21)
    J17 --> U1
    J17 --> U2
    J17 --> A1
    J21 --> U1
    J21 --> U2
    J21 --> A1
    A --> T10(10)
    T10 --> J11(Java 11)
    T10 --> J17(Java 17)
    T10 --> J21(Java 21)
    A --> T11(11)
    T11 --> J11(Java 11)
    T11 --> J17(Java 17)
    T11 --> J21(Java 21)

In a large enterprise, you’re not only working with Tomcat. There will be various Java versions, some legacy but still required for old systems. And more: JBoss, multiple Python versions, Apache or Nginx, Redis, NoSQL DBs, PostgreSQL from version 10 to 16. All requiring support on 2-3 different operating systems. The number of combinations is overwhelming!

Some questions should already arise in your head…

## How can We do it better?

In organizations big enough, you need a central process to manage upgrades. You need to prove that you at least tried to keep everything updated. That’s why it might make sense to manage your own base images, which you can control and manage security upgrades at your own pace, [with known rules in the organization](https://gagor.pro/2024/02/best-practices-for-patching-and-deprecating-docker-images/) . We know what we have to do, but how to do it?

Standard tools provide a lot of convenience to parameterize builds, so you might start wrapping some shell scripts to use them. At some point, you will realize that without parallel builds, it would take hours to finish, so you make your scripts even more complicated. [I used Makefiles for that](https://gagor.pro/2024/02/how-i-stopped-worrying-and-loved-makefiles/) . Then you will face small differences between tools or OS versions - sometimes it’s a package name, other times it’s a file location. You have to keep those differences somewhere and start adding more directories, copying files back and forth, making it easy to make a typo or mistake. Just take a look at [official Corretto images](https://github.com/corretto/corretto-docker/tree/a2a619f2ff274d58df109c775ae79acc7b8605a2) - there are directories, with sub-directories, with sub… To manage all the quirks.

That’s when you start thinking:

> If I could just template this and that, I’d have just two files and the rest could be autogenerated.

But building such a templating system is not easy, and that’s where I reached too. I needed a tool that would allow me to easily declare what I want to build, and then execute it simply. I want to make the whole tagging process easy, and I use labels on images too, so I wanted them supported as well.

I found this [one article](https://medium.com/@bossm8/dockerfile-templating-made-easy-a261f9f2bd49) but I wasn’t happy with the tool. It was too complicated and gluing things together felt odd. I knew I could do better.

My idea was inspired by Ansible playbooks. Simple YAML, easy to write, easy to read and understand, with structure that describe itself. Ansible relies on Jinja2[2](#fn:2) and as I started writing my tool in Python, I used it too. But most of my friends, including myself, got used to GoLang templates - used for Helm charts templating and widely around Docker tools. I decided it’s time to learn GoLang (how hard could it be, right?) and switch templating language. That’s how [`template-dockerfiles`](https://github.com/tgagor/template-dockerfiles) (or [`td`](https://github.com/tgagor/template-dockerfiles) in short) tool started: .

Key features:

1. Simple YAML config defining what to build and how.
2. Dockerfiles templating when needed (GoLang templates + Sprig functions[3](#fn:3)).
3. Tags or versions templated the same way.
4. Labels (I use OCI Label Schema heavily) templated the same way.
5. Easy parallel builds when possible.
6. Easy to integrate with CI/CD, allowing to just generate templates or build and push completely - whatever you prefer.

## Why templating is better?

At some scale, copying and pasting files and scripts, despite being a simple task, can be error-prone due to the number of changes you have to perform. This increases the probability of mistakes or typos. I’ve made mistakes like that, they passed review by my teammates and went to production because there were just a lot of similar changes and it was hard to spot one outlier.

Working with templates has the benefit that you define a general rule by which templates will be generated. Over time, you might add a few exceptions or additional steps required for specific versions. When you generate images from templates, they follow the rules strictly, and if you make a small mistake in the definition, it will propagate to multiple images, causing a blast impact. We can say that either you succeed or fail spectacularly 🤣

Working with templates is more like coding. It requires a higher level of focus and by that, it’s less prone to making a mistake that would apply to just one version. On the other hand, copying and pasting the same change across multiple files or directories is a dumb task, which we’re doing on auto-pilot, and it’s much easier to make a silly mistake.

Enough talking about the reasons. Let me go through some examples.

## Templating Dockerfiles Examples

In general, my tool requires:

1. Configuration file.
2. Bunch of Docker files.

You can structure them as you want as long as the configuration is aligned with your directory structure.

### Example 1: Simple Configuration, no templating

Let start with something simple, like preparing a generic base image. You save this in your `build.yaml` file (or whatever you would provide to `--config` flag).

Define your config file: build.yaml

```
images:
  base:
    dockerfile: base/Dockerfile
    variables:
      alpine:
        - "3.18"
        - "3.19"
        - "3.20"
    args:
      BASEIMAGE: alpine:{{ .alpine }}
    tags:
      - base:{{ .tag }}-alpine{{ .alpine }}
      - base:{{ .tag }}-alpine{{ .alpine | splitList "." | first }}
      - base:alpine{{ .alpine | splitList "." | first }}
      - base:{{ .tag }}
```

  

#### Fields Description

- `images`: The root key for defining different images.
- `base`: The name of the image. Name it as
- `dockerfile`: The path to the Dockerfile or Dockerfile template (`.tpl` extension).
- `variables`: Variables to be used in the template.
- `alpine`: A variable which represents list of Alpine versions.
- `args`: Arguments that will be passed as `--build-arg`, during the image build.
- `tags`: The tags to be applied to the generated images, plus templated values.
- `{{ .tag }}` : Is a value provided to `--tag` parameter, usually a version of your image.

Let choose really basic, base image:

```
ARG BASEIMAGE
FROM ${BASEIMAGE}
CMD ["echo"]
```

Assuming you already [installed the tool](https://github.com/tgagor/template-dockerfiles?tab=readme-ov-file#installation) , let generate templates by calling:

```
td --config build.yaml --tag 1.0.0 --build
```

It’s a simple config file but there’s already a lot happening there. It will produce 5 Docker images, one per each Alpine version and will tag them according to the rules. Thanks to that we already have multiple patterns, from which your developers can choose

```
$ docker image ls

REPOSITORY   TAG                 IMAGE ID       CREATED        SIZE
base         1.0.0              bb5fa559eff4   3 months ago   12.1MB
base         1.0.0-alpine3      bb5fa559eff4   3 months ago   12.1MB
base         1.0.0-alpine3.20   bb5fa559eff4   3 months ago   12.1MB
base         1.0.0-alpine3.19   5e664aff1066   3 months ago   11.5MB
base         1.0.0-alpine3.18   87b441f4c072   3 months ago   11.5MB
```

If you would like to introduce [just released Alpine 3.21](https://alpinelinux.org/posts/Alpine-3.21.0-released.html) , it’s just one line more in the config file.

### Example 2: Simple configuration with templating

We went through simplest example, so it’s time to rise the bar. We will use `Dockerfile.tpl` now and we don’t need `args` anymore. We will also introduce Alpine 3.21, which for some reason would require to do something differently than other versions. We will also add one more tag, to simplify referring to specific Alpine’s version. Final config will be:

Define your config file: build.yaml

```
images:
  base:
    dockerfile: base/Dockerfile.tpl
    variables:
      alpine:
        - "3.18"
        - "3.19"
        - "3.20"
        - "3.21"
    tags:
      - base:{{ .tag }}-alpine{{ .alpine }}
      - base:alpine{{ .alpine }}
      - base:{{ .tag }}-alpine{{ .alpine | splitList "." | first }}
      - base:alpine{{ .alpine | splitList "." | first }}
      - base:{{ .tag }}
```

Our template is just slightly different as it have to “react” to this “breaking change” in Alpine 3.21. It would be easy to use `bash` conditions to react to different Alpine version in `RUN` directives, but it’s not easy to do the same for `CMD` - at least, not without custom entrypoint. As Go-Lang templates, provide us conditions and loops, we can do really crazy things in those templates, but please - don’t 😉.

```
FROM alpine:{{ .alpine }}

{{ if eq .alpine "3.21" }}
CMD echo this is alpine 3.21 specific
{{ else }}
CMD echo this is generic
{{ end }}
```

Build command stays the same, but we will bump a minor version[4](#fn:4):

```
td --config build.yaml --tag 1.1.0 --build
```

And the result will be:

```
$ docker image ls

REPOSITORY   TAG                 IMAGE ID       CREATED          SIZE
base         alpine3.18          c46557602bb4   30 minutes ago   11.5MB
base         v1.1.0-alpine3.18   c46557602bb4   30 minutes ago   11.5MB
base         alpine3.19          ecc2bc3541fe   30 minutes ago   11.5MB
base         v1.1.0-alpine3.19   ecc2bc3541fe   30 minutes ago   11.5MB
base         alpine3.20          0319369f1076   30 minutes ago   12.1MB
base         v1.1.0-alpine3.20   0319369f1076   30 minutes ago   12.1MB
base         alpine3             9d78f7b4ccac   30 minutes ago   12.2MB
base         alpine3.21          9d78f7b4ccac   30 minutes ago   12.2MB
base         v1.1.0              9d78f7b4ccac   30 minutes ago   12.2MB
base         v1.1.0-alpine3      9d78f7b4ccac   30 minutes ago   12.2MB
base         v1.1.0-alpine3.21   9d78f7b4ccac   30 minutes ago   12.2MB
```

Let test the result:

~/2025/01/efficient-dockerfile-templating-for-complex-build-scenarios/

```
$ docker run -ti --rm base:alpine3.18
this is generic

$ docker run -ti --rm base:alpine3.21
this is alpine 3.21 specific
```

Let check again what happen here. With quite the same configuration and a template of Dockerfile we’ve been able to generate 4 different images (1 per Alpine’s version), with 11 tags just to make the choice for people easier. Tags are generated in order, and variable’s values are also ordered so latest version of Alpine (listed as last one) is the new default.

It’s really clean and easy to follow, what happens here, which would be even more important in the next example 😉

### Example 3: Complex Configuration

I used Tomcat as an example earlier and this will be our next example. I know, that there are ready to use images, but it’s just a good, complex enough example 😄

Example config file: tomcat.yaml

```
registry: repo.local
prefix: base
maintainer: Tomasz Gągor <https://gagor.pro>

labels:
  org.opencontainers.image.vendor: My Corp
  org.opencontainers.image.licenses: GPL-3.0-only
  org.opencontainers.image.url: https://my.url
  org.opencontainers.image.documentation: https://my.url/docs
  org.opencontainers.image.title: My Corp's Docker base images
  org.opencontainers.image.description: |
    This is a longer description of what this image is capable of.
    You might be setting your own timezone or preferred language settings
    and explain it nicely here.
    Some labels will be added by the tool automatically too.

images:
  base:
    dockerfile: base/Dockerfile
    variables:
      alpine:
        - "3.20"
        - "3.21"
    args:
      BASEIMAGE: alpine:{{ .alpine }}
    tags:
      - base:{{ .tag }}-alpine{{ .alpine }}
      - base:alpine{{ .alpine }}
      - base:{{ .tag }}-alpine{{ .alpine | splitList "." | first }}
      - base:alpine{{ .alpine | splitList "." | first }}
      - base:{{ .tag }}
      - base:latest

  jdk:
    dockerfile: jdk/Dockerfile
    variables:
      alpine:
        - "3.20"
        - "3.21"
      java:
        - 11
        - 17
        - 21
    args:
      BASEIMAGE: openjdk:{{ .java }}-jdk-alpine{{ .alpine }}
    tags:
      - jdk:{{ .tag }}-{{ .java }}-jdk-alpine{{ .alpine }}
      - jdk:{{ .tag }}-{{ .java }}-jdk-alpine{{ .alpine | splitList "." | first }}
      - jdk:{{ .java }}-jdk-alpine{{ .alpine | splitList "." | first }}
      - jdk:{{ .java }}-jdk
      - jdk:{{ .java }}
      - jdk:{{ .tag }}
      - jdk:latest

  jre:
    dockerfile: jre/Dockerfile
    variables:
      alpine:
        - "3.20"
        - "3.21"
      java:
        - 11
        - 17
        - 21
    args:
      BASEIMAGE: {{ .registry }}/{{ .prefix }}/jdk:{{ .java }}-jdk-alpine{{ .alpine }}
    tags:
      - jre:{{ .tag }}-{{ .java }}-jre-alpine{{ .alpine }}
      - jre:{{ .tag }}-{{ .java }}-jre-alpine{{ .alpine | splitList "." | first }}
      - jre:{{ .java }}-jre-alpine{{ .alpine | splitList "." | first }}
      - jre:{{ .java }}-jre
      - jre:{{ .java }}
      - jre:{{ .tag }}
      - jre:latest

  tomcat:
    dockerfile: tomcat/Dockerfile.tpl
    variables:
      alpine:
        - "3.20"
        - "3.21"
      java:
        - 11
        - 17
        - 21
      tomcat:
        - 9.0.98
        - 10.1.34
        - 11.0.2
    excludes:
      # follow minimum required version
      # https://tomcat.apache.org/whichversion.html
      - tomcat: 11.0.2
        java: 8
      - tomcat: 11.0.2
        java: 11
      - tomcat: 10.1.34
        java: 8
    tags:
      - tomcat:{{ .tag }}-tomcat{{ .tomcat }}-jdk{{ .java }}-alpine{{ .alpine }}
      - tomcat:{{ .tag }}-{{ .tomcat }}-jdk{{ .java }}-alpine{{ .alpine }}
      - tomcat:{{ .tomcat }}-jdk{{ .java }}-alpine{{ .alpine }}
      - tomcat:{{ .tag }}-{{ .tomcat }}-jdk{{ .java }}-alpine{{ .alpine | splitList "." | first }}
      - tomcat:{{ .tomcat }}-jdk{{ .java }}-alpine{{ .alpine | splitList "." | first }}
      - tomcat:{{ .tag }}-{{ .tomcat | splitList "." | first }}-jdk{{ .java }}-alpine{{ .alpine | splitList "." | first }}
      - tomcat:{{ .tomcat | splitList "." | first }}-jdk{{ .java }}-alpine{{ .alpine | splitList "." | first }}
      - tomcat:{{ .tag }}-{{ .tomcat }}-jdk{{ .java }}
      - tomcat:{{ .tomcat }}-jdk{{ .java }}
      - tomcat:{{ .tag }}-{{ .tomcat | splitList "." | first }}-jdk{{ .java }}
      - tomcat:{{ .tomcat | splitList "." | first }}-jdk{{ .java }}
      - tomcat:{{ .tag }}-{{ .tomcat }}
      - tomcat:{{ .tomcat }}
      - tomcat:{{ .tag }}-{{ .tomcat | splitList "." | first }}
      - tomcat:{{ .tomcat | splitList "." | first }}
      - tomcat:latest
```

  

#### Fields Description

- `registry`: URL to our private registry.
- `prefix`: Add a `base` prefix to each image.
- `maintainer`: A contact team or person.
- `labels`: Bunch of [OCI Label Schema](https://github.com/opencontainers/image-spec/blob/main/annotations.md) labels.
- `excludes`: Variants we don’t want to build.
- rest as in the simple example.

I won’t be showing Dockerfiles here, as the main actor here is configuration. It would be extremely hard to follow such structure with all those variations with bash scripts or Makefiles. It’s around 100 lines of configuration, easy to read and understand that will produce tens of image variants, that can be used by your developers in a way they need. Templating abilities on `tags`, `args` or `labels` allow to dynamically describe our images.

That’s the final I wanted to achieve. Simple recipe to deliver complex build strategies for Docker images.

### More Examples

I will try to update and curate a list of good examples of `td` usage in the wild. For now, it’s mostly mine stuff:

1. [https://github.com/tgagor/docker-centos ](https://github.com/tgagor/docker-centos)
    
    Weekly updated CentOS Stream images. `td` for building, [Github actions](https://github.com/tgagor/docker-centos/blob/master/.github/workflows/build.yml) for versioning, security scans. That’s really a simple example to start with.
    
2. [https://github.com/tgagor/template-dockerfiles/tree/main/example ](https://github.com/tgagor/template-dockerfiles/tree/main/example)
    
    Examples in the [`td` repo](https://github.com/tgagor/template-dockerfiles) . They show how to build [Corretto](https://aws.amazon.com/corretto/) images based on Alpine Linux in JDK and JRE variants.
    
3. [https://github.com/tgagor/docker-chisel ](https://github.com/tgagor/docker-chisel)
    
    [Cannonical released](https://ubuntu.com/blog/craft-custom-chiselled-ubuntu-distroless) interesting [tool called Chisel](https://github.com/canonical/chisel) . It allows to “cut” minimal versions of Docker images by extracting only binary dependencies required to run our app. It allows to compete with Alpine’s images on size, but with standard glibc.
    
    **It’s a complete, multi-platform build with Github actions for automatic builds!**
    

## What more you can do?

I plan to extend the capabilities of the tool to include:

1. Image squashing for reducing the number of layers (`--squash` flag, already supported).
2. Different compression options for even better size optimizations out of the box. As savings in the base images accumulate in all child images.
3. Multi-platform builds support.

---

Enjoyed? [![Buy Me a Coffee at ko-fi.com](https://storage.ko-fi.com/cdn/kofi5.png?v=3)](https://ko-fi.com/tgagor)