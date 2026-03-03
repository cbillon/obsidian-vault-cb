---
link: https://www.felipecruz.es/building-a-dagger-module/
byline: Felipe Cruz
site: Felipe Cruz
date: 2024-04-10T17:23
excerpt: |-
  Bash scripts, Makefiles, Taskfiles, Bakefiles... if there is one thing developers agree on is the fact that there are too many tools used as glue to put a CI/CD pipeline together.

  Whenever I have to create a CLI tool in Go, I normally copy and paste several files from a previous repo in order to build, test, lint and distribute the Go binary as a container image in a registry.

  Recently I've been defining a reusable GitHub workflow to centralize my CI pipeline steps. However, configuring succes
twitter: https://twitter.com/@felipecruz
slurped: 2026-03-03T19:39
title: Building a Dagger Module to standardize the CI pipeline of my Go projects
tags:
  - dagger
---

Bash scripts, Makefiles, Taskfiles, Bakefiles... if there is one thing developers agree on is the fact that there are too many tools used as glue to put a CI/CD pipeline together.

Whenever I have to create a CLI tool in Go, I normally copy and paste several files from a previous repo in order to build, test, lint and distribute the Go binary as a container image in a registry.

Recently I've been defining a reusable GitHub workflow to centralize my CI pipeline steps. However, configuring successfully a workflow on the first attempts remains a challenge. My Git log ends up looking like:

```
...
065ea31 Fix YAML indentation
3881fb2 Add missing env var
6bf54f8 Fix typo in GH workflow
822180e Initial commit
```

So, every time I make changes in my workflow, I have that feeling of "push and pray". That unpleasant feeling is exactly what Dagger tries to remove.

## What's Dagger?

In a nutshell, [**Dagger**](http://dagger.io/?ref=felipecruz.es) **is an OSS programmable CI/CD engine**. It tries to restore the confidence when running CI/CD pipelines by **allowing developers to write their pipelines using code and run them both locally and remotely using their favorite programming language.**

> _At the moment only Go, Python and TypeScript SDKs are available. There're plans to add more languages, though._

And Dagger does that by leveraging a powerful and widespread technology to make sure things run consistently everywhere. Does it sound familiar? Yes, **Linux containers!**

### Dagger engine

The Dagger engine is a custom version of BuildKit - the same component that powers `docker build`. It is responsible for efficiently running your pipeline as a DAG (Directed Acyclic Graph). It's shipped as a container image - `registry.dagger.io/engine` - and runs as a privileged container.

### Dagger CLI

The Dagger CLI is the interface between you and the Dagger engine. It's used to call a module function, among many other things. It requires a container runtime to bootstrap the Dagger engine. Once the bootstrapping is done, Dagger will directly run your pipeline creating its own containers (container-in-container).

## Defining my CI/CD pipeline

The goal is to use Dagger to write a programmable CI/CD pipeline in Go that **can be executed both locally and remotely** (from a GHA workflow) in a predictable and consistent manner.

![](https://www.felipecruz.es/content/images/2024/04/image-7.png)

The pipeline will consist of the following steps:

1. Build a cross-platform Go program for multiple platforms.
2. Lint it with `golangci-lint`.
3. Run the unit tests with `go test`.
4. Build a minimal multi-platform image that contains the statically linked Go binary.
5. Analyze the image for critical and high CVEs using Docker Scout.
6. Push the multi-arch image to a container registry.

For this pipeline, I've written two Dagger modules: a [`go` module](https://github.com/felipecruz91/daggerverse/tree/main/go?ref=felipecruz.es) for Go-specific operations and a [`scout` module](https://github.com/felipecruz91/daggerverse/tree/main/scout?ref=felipecruz.es) to analyze the CVEs of a container image. In this blog post I'll be focusing on the former.

## Creating the Go Dagger module

**A Dagger module is just a collection of functions** written in a supported programming language by Dagger. A module defines every pipeline step as a [Go/Python/TS] function that can be invoked and executed independently of each other using the Dagger CLI. You can bootstrap a new module with `dagger init --sdk=go`.

In my case, the purpose of my [Go Dagger module](https://daggerverse.dev/mod/github.com/felipecruz91/daggerverse/go@12b847cf74dde253bc3a541cff07f6cb08049a79?ref=felipecruz.es#module-about) is to define a set of functions responsible for building, testing, listing and pushing a Go program as a container image to a registry.

```
$ dagger functions
Name           Description
build          Build builds the Go binary for the specified go version and platforms
docker-build   DockerBuild packages the Go binary into a container image
docker-push    DockerPush packages the Go binary into a container image and pushes it to a registry
lint           Lint runs the Go linter
test           Test runs the Go tests
```

List of functions exposed by my Go Dagger module.

### Defining the Build function in the module

To create a new function in the module I just had to define a new method on the auto-generated `GoDagger` struct which is named after the module's name. Here's the method signature:

```
func (m *GoDagger) Build(ctx context.Context,
	// source is the directory containing the Go source code
	// +required
	source *Directory,
	// goVersion is the version of Go to use for building the binary
	// +optional
	// +default="1.22.0"
	goVersion string,
	// platforms is the list of platforms to build the binary for
	// +optional
	// +default=["linux/amd64", "linux/arm64"]
	platforms []string) (*Directory, error) { ... }
```

![](https://www.felipecruz.es/content/images/2024/04/image-5.png)

The [`Build` function](https://github.com/felipecruz91/daggerverse/blob/main/go/build.go?ref=felipecruz.es#L13-L43) takes a source directory with the Go source code and returns a `*Directory` which contains the cross-platform Go binaries, and whether there's an error. It iterates for very platform and calls a [private function](https://github.com/felipecruz91/daggerverse/blob/main/go/build.go?ref=felipecruz.es#L45) that builds a platform-specific Go binary which returns a `*Container` object. The binaries will then be extracted from it later on.

### Running the function from the CLI

You can find my module available in the `go` subdirectory of my [GitHub repository](https://github.com/felipecruz91/daggerverse?ref=felipecruz.es). If you have the Dagger CLI and Docker installed, you could compile a Go program yourself just by running:

```
# Build the binary in the current dir and export it
dagger call -m github.com/felipecruz91/daggerverse/go \
  build --source . --platforms "darwin/arm64" -o bin

# Execute the binary
./bin/app_darwin_arm64
```

0:00

/0:27

![](https://www.felipecruz.es/content/media/2024/04/output_thumb.jpg)

Let's pause here for a second.

The `Build` function creates a container from a Golang image where the compilation of the source code happens into a statically linked binary for a specific platform.

💡

What you just witnessed was the compilation of a Go program without requiring the Go toolchain to be installed on your computer. All these steps take place within a container, eliminating the need for any additional tools to be installed, apart from a container runtime, for this function to operate.

In the implementation of the private function, the Go program is compiled using several methods available in the Go SDK, which are very similar to Dockerfile instructions.

```
...

return cli.Container().
    From("golang:"+goVersion).
    WithWorkdir("/src").
    WithMountedCache("/go/pkg/mod", m.goModCacheVolume()).
    WithDirectory("/src", source, ContainerWithDirectoryOpts{
        Include: []string{"**/go.mod", "**/go.sum"},
    }).
    WithExec([]string{"go", "mod", "download"}).
    WithEnvVariable("GOMODCACHE", "/go/pkg/mod").
    WithMountedCache("/go/build-cache", m.goBuildCacheVolume()).
    WithEnvVariable("GOCACHE", "/go/build-cache").
    WithEnvVariable("CGO_ENABLED", "0").
    WithEnvVariable("GOOS", os).
    WithEnvVariable("GOARCH", arch).
    WithMountedDirectory("/src", source).
    WithExec([]string{"go", "build", "-ldflags", "-s -w", "-o", binaryName, "."}).
    Sync(ctx)
...
```

In summary, what I'm doing here is telling Dagger to create a container from a Golang image, mount the source directory into the container filesystem at `/src`, create a volume to cache the dependencies, and compile the Go program for a specific platform along with some environment variables.

### Dagger functions can return core types

Notice that Dagger functions can use core types as return values such as `Container`, `Directory` or `File`. The private function returns a `*Container` object that includes the Go binary for a specific platform.

Hold on, what does it mean to return a `*Container`?

Dagger defines powerful core types, such as `Container`, which represents the state of an OCI container managed by the Dagger engine. This type provides a complete API that can be used, for instance, to retrieve the a file (e.g. the Go binary) from the container filesystem.

The `Build` function ends up collecting all the platform-specific Go binaries from the filesystem of each container object and returns a directory with all of them:

```
...
file := ctr.File(filepath.Join("/src", binaryName))
		files = append(files, file)
	}

return cli.Directory().WithFiles(".", files), nil
...
```

### Chaining function invocations

Because the `Build` function returns a `*Directory`, another Dagger module or the Dagger CLI could invoke other built-in functions exposed by these core types.

For instance, if your function returns a `*Directory`, you could tell it to execute the `entries` built-in function to list the files in that directory. Similarly, you could also call `export` to write the contents of the directory (i.e. the binaries) to a path on the host.

```
$ dagger call build --source . --platforms "linux/amd64,linux/arm64" entries

app_linux_amd64
app_linux_arm64
```

In short, depending on the core type, there're different functions available. For instance, if your function returns a `*Container`, you can use the built-in `publish` function to push the image to a registry.

💡

An interesting fact about Dagger Modules is that it doesn't matter in which supported programming language the module is written: your module could be written in Go and invoke a function from another module written in Python.

This is incredibly powerful as it allows us to consume modules without having to re-write their logic into the same programming language as our module.

## Publishing the module to the Daggerverse

The [Daggerverse](https://daggerverse.dev/?ref=felipecruz.es) serves as the entry point for exploring a wide variety of modules developed by both the community and partners. It functions similarly to DockerHub's role for container images, providing a centralized platform for accessing Dagger modules, but with some differences:

- **Dagger does not host your module's code**, just happens to **index** modules hosted in **public** repositories in GitHub.
- **You don't need to publish your module explicitly to the Daggerverse in order for it to be shown there**. As long as you or someone else calls your module with `dagger call -m github.com/your/module/...` the module will be automatically indexed.

In my case, that was exactly what happened. I was surprised when I saw my module showing up in the Daggerverse without publishing it explicitly. I think this is an unexpected default behavior that (hopefully) may change in the future.

![](https://www.felipecruz.es/content/images/2024/04/image-1.png)

## Using the Go Dagger module in a GHA workflow

Once the module is available in my public GitHub repository, I can create a GitHub Actions workflow that will:

1. Install the Dagger CLI.
2. Call my build/lint/test... functions individually with `dagger call`.
3. Stop the Dagger engine.

```
- name: Install Dagger CLI
  run: |
    cd /usr/local && { curl -L https://dl.dagger.io/dagger/install.sh | DAGGER_VERSION=$DAGGER_VERSION sh; cd -; }
    dagger version

- name: Go build
  run: |
    dagger call -m "$DAGGER_GO_MODULE" build --source . --platforms "linux/amd64,linux/arm64" -o bin

# Rest of functions omitted for brevity...

  - name: Stop Dagger engine
  run: docker stop -t 300 $(docker ps --filter name="dagger-engine-*" -q)
  if: always()
```

Excerpt from the GHA workflow that runs the Dagger functions defined in the module.

You can find the complete GHA workflow [here](https://github.com/felipecruz91/go-dagger-pipeline/blob/main/.github/workflows/ci.yml?ref=felipecruz.es).

![](https://www.felipecruz.es/content/images/2024/04/image.png)

## The Dagger cache in GitHub Actions

Github-hosted runners are ephemeral, meaning you get a brand new one every time a workflow execution happens. Because of this, the Dagger engine which runs as a container in the VM, persists its state in their own container filesystem at `/var/lib/dagger`.

Because of this, I couldn't benefit from the Dagger cache. Every time the workflow run, the pipeline was running completely from scratch without any caching at all. However, there're 2 alternatives for this:

a) Use Dagger Cloud which offers distributed caching.  
b) Run the Dagger engine in a VM and mount a volume to persist its state.

Then you can tell the Dagger CLI to point to that Dagger engine whose state is persisted on disk by using the `_EXPERIMENTAL_DAGGER_RUNNER_HOST` environment variable.

## Current limitations

As of the time of this writing, I came across the following limitations in Dagger:

- ~~Installing Dagger Modules from a private GitHub repo is not supported yet (~~[~~GH issue~~](https://github.com/dagger/dagger/issues/6113?ref=felipecruz.es)~~). Your module must be hosted in a public GitHub repo.~~ **Update: Added support in Dagger engine v0.13.**
- Calling functions defined in a Dagger Module is only allowed from another module or the Dagger CLI ([GH issue](https://github.com/dagger/dagger/issues/5993?ref=felipecruz.es)). This means that a regular Go program cannot directly invoke a Dagger function by `go get`ting the module - you'd need to Daggerize it as a Dagger module.
- Lack of support to push SBOM and provenance attestations in container images ([GH issue](https://github.com/dagger/dagger/issues/5621?ref=felipecruz.es)).

## Wrapping up

While there's a lot of documentation, tutorials and hundreds of GitHub Actions to help define your CI/CD pipeline, iterating fast when making changes in a GitHub workflow still remains a challenge. Lacking the ability to run a pipeline both locally and get it to work the same in a remote CI system it's something we, developers, are dealing with pretty much everyday. However, rewriting your CI/CD pipeline in Dagger from scratch can take some time and there're a some limitations important to consider.

Building a Dagger module for the first time was a nice experience. The fact that I could define my CI/CD pipeline in the programming language of my choice was very convenient _and_ run it both locally and remotely it's fascinating. The Dagger documentation has lots of examples and it's very well structured, so I'd recommend you to start from there and grasp some basic concepts. Furthermore, the Dagger team is very kind and active in their Discord server and they're always willing to help.

Finally, I'd like to especially thank Lev, Jed and Marcos for his help and invite you to build your own module too!

---

**_Disclaimer: The content I have contributed to this blog post is my own and does not necessarily represent the views or opinions of my employer._**