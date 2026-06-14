---
link: https://github.com/ente-io/ente/tree/main/cli#readme
site: GitHub
excerpt: 💚 End-to-end encrypted cloud for everything. Contribute to ente-io/ente development by creating an account on GitHub.
twitter: https://twitter.com/@github
slurped: 2026-06-12T15:18
title: ente/cli at main · ente-io/ente
tags:
  - ente
---
[site web](ente.com/fr)
The Ente CLI is a Command Line Utility for exporting data from [Ente](https://ente.com/). It also does a few more things, for example, you can use it to decrypt the export from Ente Auth.

## Install

[](https://github.com/ente-io/ente/tree/main/cli#install)

The easiest way is to download a pre-built binary from the [GitHub releases](https://github.com/ente-io/ente/releases?q=tag%3Acli-v0).

You can also build these binaries yourself

Or you can build from source

 go build -o "bin/ente" main.go

The generated binaries are standalone, static binaries with no dependencies. You can run them directly, or put them somewhere in your PATH.

There is also an option to use [Docker](https://github.com/ente-io/ente/tree/main/cli#docker).

## Usage

[](https://github.com/ente-io/ente/tree/main/cli#usage)

Run the help command to see all available commands.

### Accounts

[](https://github.com/ente-io/ente/tree/main/cli#accounts)

If you wish, you can add multiple accounts (your own and that of your family members) and export all data using this tool.

#### Add an account

[](https://github.com/ente-io/ente/tree/main/cli#add-an-account)

Note

`ente account add` does not create new accounts, it just adds pre-existing accounts to the list of accounts that the CLI knows about so that you can use them for other actions.

#### List accounts

[](https://github.com/ente-io/ente/tree/main/cli#list-accounts)

#### Change export directory

[](https://github.com/ente-io/ente/tree/main/cli#change-export-directory)

ente account update --app auth/photos --email email@domain.com --dir ~/photos

### Export

[](https://github.com/ente-io/ente/tree/main/cli#export)

#### Start export

[](https://github.com/ente-io/ente/tree/main/cli#start-export)

### CLI Docs

[](https://github.com/ente-io/ente/tree/main/cli#cli-docs)

You can view more cli documents at [docs](https://github.com/ente-io/ente/blob/main/cli/docs/generated/ente.md). To update the docs, run the following command:

## Docker

[](https://github.com/ente-io/ente/tree/main/cli#docker)

If you fancy Docker, you can also run the CLI within a container.

### Configure

[](https://github.com/ente-io/ente/tree/main/cli#configure)

Modify the `docker-compose.yml` and add volume. `cli-data` volume is mandatory, you can add more volumes for your export directory.

Build and run the container in detached mode

docker-compose up -d --build

Note that [BuildKit](https://docs.docker.com/go/buildkit/) is needed to build this image. If you face this issue, a quick fix is to add `DOCKER_BUILDKIT=1` in front of the build command.

`exec` into the container

docker-compose exec ente-cli /bin/sh -c "./ente-cli version"
docker-compose exec ente-cli /bin/sh -c "./ente-cli account add"