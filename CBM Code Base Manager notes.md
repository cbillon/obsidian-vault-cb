---
tags:
  - cbm
---
## Composer

Composer packages are distributed as ZIP archives downloaded directly from source (usually GitHub/GitLab archives) or from Packagist.

Edge cases:

- Composer itself doesn't define a package archive format; it downloads repository archives
- `composer archive` creates a ZIP of your project
- The `vendor/` directory and `vendor/autoload.php` are generated locally, not distributed

[Composer basic usage](https://getcomposer.org/doc/01-basic-usage.md)

[Composer json schema](https://getcomposer.org/doc/04-schema.md)

## ## Algorithm Categories

[Resolver](https://github.com/ecosyste-ms/package-manager-resolvers#algorithm-categories)

Package managers generally fall into a few algorithmic families.

Some registries only list one version per package in their index (APT, DNF, Pacman, Homebrew, Alpine), which simplifies resolution since there's no version selection. Others (npm, PyPI, RubyGems, crates.io, Maven Central) list all historical versions, requiring the resolver to choose among candidates.

## Criteres de choix

[Stars and forks tell me almost nothing](https://www.ias.cs.tu-bs.de/publications/GithubTranco.pdf). They measure how many people have looked at a repository, which correlates with marketing and visibility more than quality. Some of the most reliable libraries I use have modest star counts because they’re boring infrastructure that just works. Conversely, I’ve seen heavily-starred projects with broken APIs and unresponsive maintainers.

I also ignore commit frequency. Stable libraries [often don’t need regular commits](https://arxiv.org/abs/1707.02327), especially small ones that do one thing well. A library that hasn’t been touched in a year might be abandoned, or it might just be finished. The way to tell the difference is to look at whether maintainers respond to issues and pull requests, not at the commit graph.

I apply the same [cooldown logic](https://blog.yossarian.net/2025/11/21/We-should-all-be-using-dependency-cooldowns) I use for updating dependencies: let other people find the sharp edges first.

Authors will be expected to tag releases with semantic version and we encourage using tagged version for published stable version