---
link: https://github.com/PlakarKorp/plakar
site: GitHub
excerpt: plakar is a backup solution powered by Kloset and ptar - PlakarKorp/plakar
twitter: https://twitter.com/@github
slurped: 2026-09-21T15:21
title: "GitHub - PlakarKorp/plakar: plakar is a backup solution powered by
  Kloset and ptar"
---

[![Plakar: open source backup engine](https://github.com/PlakarKorp/plakar/raw/main/docs/assets/Plakar_Logo_Simple_Primary.png)](https://github.com/PlakarKorp/plakar/blob/main/docs/assets/Plakar_Logo_Simple_Primary.png)

## Plakar: open source backup engine

[](https://github.com/PlakarKorp/plakar#plakar-open-source-backup-engine)

**Encrypted, deduplicated, verifiable, and scalable.**

Back up anything, store anywhere, restore everywhere, with zero-trust encryption and no vendor lock-in.

[![Join our Discord community](https://camo.githubusercontent.com/fd3f6635b77e8e9a0258bc2a00b709ee3929de527ab3c02e8f61cf90b7384247/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f446973636f72642d4a6f696e25323055732d707572706c653f6c6f676f3d646973636f7264266c6f676f436f6c6f723d7768697465267374796c653d666f722d7468652d6261646765)](https://discord.gg/uuegtnF2Q5) [![Subscribe on YouTube](https://camo.githubusercontent.com/1b307d214f86d9722bb1e81794a703d81f869ba5b0e3e4eb6ccd1fed07589841/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f596f75547562652d5375627363726962652d7265643f6c6f676f3d796f7574756265266c6f676f436f6c6f723d7768697465267374796c653d666f722d7468652d6261646765)](https://www.youtube.com/@PlakarKorp) [![Join our Subreddit](https://camo.githubusercontent.com/7ad8c95dfdddd9350a578fccc140b6e7138ce04924ca0269d3ecc78fe96a7bee/68747470733a2f2f696d672e736869656c64732e696f2f62616467652f5265646469742d4a6f696e25323072253246706c616b61722d6f72616e67653f6c6f676f3d726564646974266c6f676f436f6c6f723d7768697465267374796c653d666f722d7468652d6261646765)](https://www.reddit.com/r/plakar/)

[![codecov](https://camo.githubusercontent.com/32d967f0c0b513370c5d4252b6032ed084067139733e55c46236fc9a20072cd6/68747470733a2f2f636f6465636f762e696f2f67682f506c616b61724b6f72702f706c616b61722f6272616e63682f6d61696e2f67726170682f62616467652e737667)](https://codecov.io/gh/PlakarKorp/plakar)

[Deutsch](https://www.readme-i18n.com/PlakarKorp/plakar?lang=de) | [Español](https://www.readme-i18n.com/PlakarKorp/plakar?lang=es) | [français](https://www.readme-i18n.com/PlakarKorp/plakar?lang=fr) | [日本語](https://www.readme-i18n.com/PlakarKorp/plakar?lang=ja) | [한국어](https://www.readme-i18n.com/PlakarKorp/plakar?lang=ko) | [Português](https://www.readme-i18n.com/PlakarKorp/plakar?lang=pt) | [Русский](https://www.readme-i18n.com/PlakarKorp/plakar?lang=ru) | [中文](https://www.readme-i18n.com/PlakarKorp/plakar?lang=zh)

## What is Plakar?

[](https://github.com/PlakarKorp/plakar#what-is-plakar)

Plakar is an open-source backup solution powered by [Kloset](https://www.plakar.io/posts/2025-04-29/kloset-the-immutable-data-store) and [ptar](https://www.plakar.io/posts/2025-06-27/it-doesnt-make-sense-to-wrap-modern-data-in-a-1979-format-introducing-.ptar/). It creates snapshots of your data, stores them in an encrypted and deduplicated store, and lets you inspect, verify, and restore them later.

Plakar stores backups in a [Kloset](https://www.plakar.io/posts/2025-04-29/kloset-the-immutable-data-store), an open-source, immutable data store that enables the implementation of advanced data protection scenarios.

Through [integrations](https://www.plakar.io/integrations), Plakar can back up and restore databases, Kubernetes workloads, object stores, and other sources alongside regular files.

What sets Plakar apart:

- snapshots are browsable: you can inspect their contents, diff two snapshots, or restore a single file without touching the rest;
- every snapshot is independently verifiable without restoring anything;
- backups are deduplicated and compressed, so keeping many snapshots does not multiply storage costs;
- encryption covers both data and metadata, with an [audited cryptography implementation](https://www.plakar.io/posts/2025-02-28/audit-of-plakar-cryptography/);
- Plakar is extensible through integrations for additional sources, storage backends, and destinations.

Plakar can be used from the command line or through its built-in web UI.

## Quickstart

[](https://github.com/PlakarKorp/plakar#quickstart)

# Install
go install github.com/PlakarKorp/plakar@latest

# Create a local repository
plakar at /var/backups create

# Back up a directory
plakar at /var/backups backup /etc

# List snapshots
plakar at /var/backups ls

# Restore a snapshot
plakar at /var/backups restore -to /tmp/restore <snapshot-id>

# Open the web UI
plakar at /var/backups ui

Prebuilt binaries are available at [https://www.plakar.io/download](https://www.plakar.io/download). For a full walkthrough, see the [quickstart guide](https://www.plakar.io/docs/community/v1.1.0/quickstart/first-backup).

## Installation

[](https://github.com/PlakarKorp/plakar#installation)

### Prebuilt binaries

[](https://github.com/PlakarKorp/plakar#prebuilt-binaries)

Download a binary for your platform: [https://www.plakar.io/download](https://www.plakar.io/download)

### Build from source

[](https://github.com/PlakarKorp/plakar#build-from-source)

Plakar requires Go 1.23.3 or higher.

go install github.com/PlakarKorp/plakar@latest

## More with Plakar

[](https://github.com/PlakarKorp/plakar#more-with-plakar)

**Archives.** You can export one or more snapshots as a `.ptar` archive using `plakar ptar`. A `.ptar` file is a self-contained, deduplicated, compressed, and encrypted archive.

**Integrations.** Plakar supports additional backup sources and storage backends through its package manager. Available integrations include PostgreSQL, MySQL, etcd, Kubernetes, S3-compatible object stores, and more.

plakar pkg add <integration>

**Distributed stores.** Kloset stores can be synchronized across locations to implement 3-2-1 backup strategies or more advanced push, pull, and sync workflows across heterogeneous environments.

plakar at /var/backups sync to @s3

See the [documentation](https://www.plakar.io/docs/community) for the full list of supported workflows.

## Documentation

[](https://github.com/PlakarKorp/plakar#documentation)

[https://www.plakar.io/docs/community](https://www.plakar.io/docs/community)

- [Quickstart](https://www.plakar.io/docs/community/v1.1.0/quickstart/first-backup)
- [Command reference](https://www.plakar.io/docs/community/v1.1.0/references/commands)
- [Integrations](https://www.plakar.io/integrations)

## Contributing and reporting issues

[](https://github.com/PlakarKorp/plakar#contributing-and-reporting-issues)

Please read the [contributing guidelines](https://github.com/PlakarKorp/plakar/blob/main/CONTRIBUTING.md) and [code of conduct](https://github.com/PlakarKorp/plakar/blob/main/CODE_OF_CONDUCT.md) before opening a pull request.

If you find a bug or want to request a change, open an issue:

[https://github.com/PlakarKorp/plakar/issues](https://github.com/PlakarKorp/plakar/issues)

## Community

[](https://github.com/PlakarKorp/plakar#community)

- [Discord](https://discord.gg/A2yvjS6r2C)
- [Reddit](https://www.reddit.com/r/plakar/)
- [YouTube](https://www.youtube.com/@PlakarKorp)

## Changelog

[](https://github.com/PlakarKorp/plakar#changelog)

See [CHANGELOG.md](https://github.com/PlakarKorp/plakar/blob/main/CHANGELOG.md) for release notes and version history.