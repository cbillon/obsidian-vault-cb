---
link: https://gagor.pro/2025/10/running-gemini-cli-in-a-docker-container/
byline: Tom
site: Tom's Blog
date: 2025-10-31T01:00
excerpt: Learn how to run Google&rsquo;s Gemini CLI in a Docker container to avoid installing Node.js and its dependencies directly on your system. This guide provides a simple setup, a recommended shell function for seamless integration, and usage examples.
tags:
  - docker
  - cli
  - ai
  - gemini
slurped: 2026-02-24T10:39
title: Running Gemini CLI in a Docker container
---

I’ve been curious for a while about AI tools that can be used from the command line. While there are a few custom tools out there, the game-changer for me was the [release of Gemini CLI](https://blog.google/technology/developers/introducing-gemini-cli-open-source-ai-agent/) . The generous token limits on the free plan were particularly appealing.

However, there’s one aspect I’m not fond of: it’s built on Node.js. The npm ecosystem is frequently targeted by supply chain attacks[1](#fn:1)[2](#fn:2)[3](#fn:3), and I prefer not to clutter my system with Node and its many packages.

My solution was to run it in a Docker container. Since the official project doesn’t provide a Docker image and community-published ones aren’t always up-to-date, I did what I often do in these situations: I built my own and tailored it to my workflow.

I started a new [project on GitHub](https://github.com/tgagor/docker-gemini-cli) to host the solution. This project automatically builds a Docker image with the latest version of Gemini CLI and Node, with weekly updates. If I encounter issues with a specific version, I can easily revert to a previous one. For me, this is much simpler than managing npm dependencies directly.

The simplest way to check it is:

```
docker run -ti --rm tgagor/gemini-cli --version
```

  

## Recommended Setup

The best way to use this image is to create a shell function that handles the necessary mount points and permissions. Add the following function to your `~/.bash_aliases` or `~/.zsh_aliases`:

```
function gemini {
    local tty_args=""
    if [ -t 0 ]; then
        tty_args="--tty"
    fi

    docker run -i ${tty_args} --rm \
        -v "$(pwd):/home/gemini/workspace" \
        -v "$HOME/.gemini:/home/gemini/.gemini" \
        -e DEFAULT_UID=$(id -u) \
        -e DEFAULT_GID=$(id -g) \
        tgagor/gemini-cli "$@"
}
```

This setup does the following:

- Mounts your current directory into `/home/gemini/workspace` inside the container.
- Persists your Gemini CLI configuration between runs by mounting `~/.gemini`.
- Aligns the container’s user permissions with your local user’s to prevent file ownership problems.
- Correctly handles TTY for interactive sessions, allowing you to run it as an interactive command or pipe files to it.

## Usage

Now I can call it as a native command:

```
# Get help
gemini --help

# Process a local file
gemini your-prompt-file.txt

# Pipe file as context
cat doc.md | gemini -p "Correct grammar"

# Use interactive mode
gemini
```

More commands you can find in [official docs](https://geminicli.com/docs/cli/commands/) .
