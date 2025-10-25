---
link: https://antonz.org/writing-package-manager/
byline: Anton Zhiyanov
site: "@ohmypy"
excerpt: Without spending a year on it.
twitter: https://twitter.com/@ohmypy
slurped: 2025-10-23T10:29
title: Writing a package manager
tags:
  - package_manager
---

Writing a package manager is not one of the most common programming tasks. After all, there are many out-of-the-box ones available. Yet, somehow I've found myself in exactly this situation.

> **How so?**
> 
> I'm a big fan of SQLite and its extensions. Given the large number of such extensions in the wild, I wanted a structured approach to managing them. Which usually involves, well, a package manager. Except there is none for SQLite. So I decided to build one!
> 
> If you haven't seen them before, SQLite extensions are just libraries (`.dll`, `.dylib` or `.so` depending on the operating system). To make an extension work, you download it and load it into SQLite.

Needless to say, building a package manager is not an easy task. In fact, Sam Boyer has written a [great article](https://medium.com/@sdboyer/so-you-want-to-write-a-package-manager-4ae9c17d9527) about the problems involved. So I won't going to dwell on it.

This article explains the design choices and implementation details that allowed me to actually build a working package manager in a couple of weeks (mostly evenings and nights, to be honest). I tried to leave out most of the SQLite specifics, so hopefully you can apply this approach to any package manager should you decide to build one.

## Design decisions

Package management is complex by nature, and there is no magic bullet that will make it simple (unless you are willing to radically narrow the scope). So let's go through the building blocks together, tackling problems as they arise.

[Spec file](#spec-file) • [Folder structure](#folder-structure) • [Scope](#project-vs-global-scope) • [Registry](#package-registry) • [Version](#version) • [Latest version](#latest-version) • [Lockfile](#lockfile) • [Source of truth](#source-of-truth) • [Checksums](#checksums) • [Dependencies](#package-dependencies) • [Install and update](#install-and-update)

### Spec file

To work with packages, the manager needs some information about them. At least the package ID and the download location. So let's design a _package spec_ file that describes a package.

Here is a simple one:

```
{
    "owner": "sqlite",
    "name": "stmt",
    "assets": {
        "path": "https://github.com/nalgeon/sqlean/releases/download/incubator",
        "files": {
            "darwin-amd64": "stmt.dylib",
            "darwin-arm64": "stmt.dylib",
            "linux-amd64": "stmt.so",
            "windows-amd64": "stmt.dll"
        }
    }
}
```

`owner` + `name` form a unique package identifier (we don't want any name conflicts, thank you very much, Python).

The `assets.path` is a base URL for the package assets. The assets themselves are listed in the `assets.files`. When the manager installs the package, it chooses the asset name according to the user's operating system, combines it with the `assets.path` and downloads the asset.

```
> install sqlite/stmt
   ↓
download spec
┌───────────────┐
│  sqlite/stmt  │
└───────────────┘
   ↓
check platform
  ↪ OS: darwin
  ↪ arch: arm64
   ↓
download asset
┌───────────────┐
│  stmt.dylib   │
└───────────────┘
```

Good start!

### Folder structure

Let's say there is a package hosted somewhere on GitHub. I tell the manager (called `sqlpkg` from now on) to install it:

```
sqlpkg install sqlite/stmt
```

The manager downloads the package and stores it locally in a folder named `.sqlpkg`:

```
.sqlpkg
└── sqlite
    └── stmt
        ├── sqlpkg.json
        └── stmt.dylib
```

(`sqlpkg.json` is the spec file and `stmt.dylib` is the package asset)

Let's add another one:

```
sqlpkg install asg017/vss
```

```
.sqlpkg
├── asg017
│   └── vss
│       ├── sqlpkg.json
│       └── vss0.dylib
│
└── sqlite
    └── stmt
        ├── sqlpkg.json
        └── stmt.dylib
```

As you can probably see, given this folder structure, it's quite easy for the manager to reason about the installed packages.

For example, if I run `sqlpkg update OWNER/NAME`, it does the following:

1. Reads the spec file from the path `.sqlpkg/OWNER/NAME/sqlpkg.json`.
2. Downloads the latest asset using the `assets.path` from the spec.
3. Replaces the old `.dylib` with the new one.

When I run `sqlpkg uninstall OWNER/NAME`, it deletes the corresponding directory.

And when I run `sqlpkg list`, it searches for all paths that match `.sqlpkg/*/*/sqlpkg.json`.

Simple, isn't it?

### Project vs. global scope

Some package managers (e.g. `npm`) use per-project scope by default, but also allow you to install packages globally using flags (`npm install -g`). Others (e.g. `brew`) use global scope.

I like the idea of allowing both project and global scope, but I do not like the flags approach. Why don't we apply a heuristic:

- If there is a `.sqlpkg` folder in the current directory, use project scope.
- Otherwise, use global scope.

This way, if users don't need separate project environments, they will just run `sqlpkg` as is and install packages in their home folder (e.g. `~/.sqlpkg`). Otherwise, they'll create a separate `.sqlpkg` for each project (we can provide a helper `init` command for this).

Project scope:

```
$ cd /my/project
$ sqlpkg init
$ sqlpkg install sqlite/stmt
$ tree .sqlpkg

.sqlpkg
└── sqlite
    └── stmt
        ├── sqlpkg.json
        └── stmt.dylib
```

Global scope:

```
$ cd /some/other/path
$ sqlpkg install sqlite/stmt
$ tree ~/.sqlpkg

/Users/anton/.sqlpkg
└── sqlite
    └── stmt
        ├── sqlpkg.json
        └── stmt.dylib
```

No flags involved!

### Package registry

For a package manager to be useful, it should support existing extensions (which, of course, are completely unaware of it at the moment). Maybe extension authors will eventually add spec files to their packages, maybe they won't — we can't rely on that.

So let's add a simple fallback algorithm. When the user runs `sqlpkg install OWNER/NAME`, the manager does the following:

1. Attempts to fetch the spec from the owner's GitHub repo `https://github.com/OWNER/NAME`.
2. If the spec is not found, fetches it from the package registry.

```
 owner/name
    ↓
┌─────────────────┐ found ┌───────────┐
│  owner's repo   │   →   │  install  │
└─────────────────┘       └───────────┘
    ↓ not found
┌─────────────────┐ found ┌───────────┐
│  pkg registry   │   →   │  install  │
└─────────────────┘       └───────────┘
    ↓ not found
 ✗ error
```

The package registry is just another GitHub repo with a two-level owner/name structure:

```
pkg/
├── asg017
│   ├── fastrand.json
│   ├── hello.json
│   ├── html.json
│   └── ...
├── daschr
│   └── cron.json
├── dessus
│   ├── besttype.json
│   ├── fcmp.json
│   └── ...
├── ...
...
```

We'll bootstrap the registry with known packages, so the manager will work right out of the box. As package authors catch up and add `sqlpkg.json` to their repos, the manager will gradually switch to using them instead of the registry.

The manager should also support links to specific GitHub repos (in case the repo has a different name than the package):

```
sqlpkg install github.com/asg017/sqlite-vss
```

And other URLs, because not everyone uses GitHub:

```
sqlpkg install https://antonz.org/downloads/stats.json
```

And also local paths:

```
sqlpkg install ./stats.json
```

All this "locator" logic complicates the design quite a bit. So if you are comfortable with requiring package authors to provide the specs, feel free to omit the fallback step and the registry altogether.

### Version

What a package without a version, right? Let's add it:

```
{
    "owner": "asg017",
    "name": "vss",
    "version": "v0.1.1",
    "repository": "https://github.com/asg017/sqlite-vss",
    "assets": {
        "path": "{repository}/releases/download/{version}",
        "files": {
            "darwin-amd64": "vss-{version}-macos-x86_64.tar.gz",
            "darwin-arm64": "vss-{version}-macos-aarch64.tar.gz",
            "linux-amd64": "vss-{version}-linux-x86_64.tar.gz"
        }
    }
}
```

We also introduced variables like `{repository}` and `{version}` so package authors don't have to repeat themselves.

When updating a package, the manager must now compare local and remote versions according to the semantic versioning rules:

```
   local spec    │    remote spec
                 │
> update         │
┌─────────────┐  │  ┌─────────────┐
│   v0.1.0    │  <  │   v0.1.1    │
└─────────────┘  │  └─────────────┘
   ↓             │
updating...      │
┌─────────────┐  │
│   v0.1.1    │  │
└─────────────┘  │
```

Nice!

### Latest version

While not required, it would be nice to support the `latest` version placeholder and automatically resolve it via API for GitHub-hosted packages:

```
{
    "owner": "asg017",
    "name": "vss",
    "version": "latest",
    "repository": "https://github.com/asg017/sqlite-vss",
    "assets": {
        "path": "{repository}/releases/download/{version}",
        "files": {
            "darwin-amd64": "vss-{version}-macos-x86_64.tar.gz",
            "darwin-arm64": "vss-{version}-macos-aarch64.tar.gz",
            "linux-amd64": "vss-{version}-linux-x86_64.tar.gz"
        }
    }
}
```

This way, package authors don't have to change the spec when releasing a new version. When installing the package, the manager will fetch the latest version from GitHub:

```
   local spec    │    remote spec    │    github api
                 │                   │
> update         │                   │
┌─────────────┐  │                   │
│   v0.1.0    │  │                   │
└─────────────┘  │                   │
   ↓             │                   │
wait a sec...    │                   │
┌─────────────┐  │  ┌─────────────┐  │  ┌─────────────┐
│   v0.1.0    │  ?  │   latest    │  →  │   v0.1.1    │
└─────────────┘  │  └─────────────┘  │  └─────────────┘
   ↓             │                   │
┌─────────────┐  │  ┌─────────────┐  │
│   v0.1.0    │  <  │   v0.1.1    │  │
└─────────────┘  │  └─────────────┘  │
   ↓             │                   │
updating...      │                   │
┌─────────────┐  │                   │
│   v0.1.1    │  │                   │
└─────────────┘  │                   │
```

In this scenario, it's important to store the specific version in the local spec, not the "latest" placeholder. Otherwise, the manager won't be able to reason about the currently installed version when the user runs an `update` command.

### Lockfile

Having an `.sqlpkg` folder with package specs and assets is enough to implement all manager commands. We can install, uninstall, update and list packages based on the `.sqlpkg` data only.

```
.sqlpkg
├── asg017
│   └── vss
│       ├── sqlpkg.json
│       └── vss0.dylib
│
└── sqlite
    └── stmt
        ├── sqlpkg.json
        └── stmt.dylib
```

But what if the user wants to reinstall the packages on another machine or CI server? That's where the _lockfile_ comes in.

The lockfile stores a list of all installed packages with just enough information to reinstall them if needed:

```
{
    "packages": {
        "asg017/vss": {
            "owner": "asg017",
            "name": "vss",
            "version": "v0.1.1",
            "specfile": "https://github.com/nalgeon/sqlpkg/raw/main/pkg/asg017/vss.json",
            "assets": {
                // ...
            }
        },
        "sqlite/stmt": {
            "owner": "sqlite",
            "name": "stmt",
            "version": "",
            "specfile": "https://github.com/nalgeon/sqlpkg/raw/main/pkg/sqlite/stmt.json",
            "assets": {
                // ...
            }
        }
    }
}
```

The only new field here is the `specfile` — it's a path to a remote spec file to fetch the rest of the package information (e.g. description, license, and authors).

Now the user can commit the lockfile along with the rest of the project, and run `install` on another machine to install all the packages listed in the lockfile:

```
   local spec    │     lockfile      │    remote spec
                 │                   │
> install        │  ┌─────────────┐  │  ┌─────────────┐
   └─ (empty)    →  │ asg017/vss  │  →  │ asg017/vss  │
                 │  │ sqlite/stmt │  │  └─────────────┘
                 │  └─────────────┘  │  ┌─────────────┐
   ┌─            ←                   ←  │ sqlite/stmt │
installing...    │                   │  └─────────────┘
┌─────────────┐  │                   │
│ asg017/vss  │  │                   │
└─────────────┘  │                   │
┌─────────────┐  │                   │
│ sqlite/stmt │  │                   │
└─────────────┘  │                   │
```

So far, so good.

### Source of truth

Lockfile sounds like a no-brainer, but in fact it introduces a major problem — we no longer have a single source of truth for any given local package.

Let's consider one of the simpler commands — `list`, which displays all installed packages. Previously, all it had to do was scan the `.sqlpkg` for spec files:

```
> list
   ↓
glob .sqlpkg/*/*/sqlpkg.json
┌─────────────┐
│ asg017/vss  │
│ sqlite/stmt │
└─────────────┘
```

But now we have _two_ sources of package information — the `.sqlpkg` folder and the lockfile. Imagine that for some reason they are out of sync:

```
   local spec    │     lockfile
                 │
> list           │
   ↓             │
let's see...     │
┌─────────────┐  │  ┌──────────────┐
│ asg017/vss  │  │  │ asg017/vss   │
└─────────────┘  │  │ nalgeon/text │
┌─────────────┐  │  └──────────────┘
│ sqlite/stmt │  │
└─────────────┘  │
   ↓             │
  ???
```

Instead of the simple "just list the .sqlpkg contents", we now have 4 possible situations for any given package:

1. The package is listed in both .sqlpkg and the lockfile with the same version.
2. The package is listed in both .sqlpkg and the lockfile, but the versions are different.
3. The package is listed in .sqlpkg, but not in the lockfile.
4. The package is listed in the lockfile, but not in .sqlpkg.

➊ is easy, but what should the manager do with ➋, ➌ and ➍?

Instead of coming up with clever conflict resolution strategies, let's establish the following ground rule:

> There is a single source of truth, and it's the contents of the .sqlpkg folder.

This immediately solves all kinds of lockfile related problems. For the `list` command example, we now only look in `.sqlpkg` (as we did before) and then synchronize the lockfile with it, adding the missing packages if necessary:

```
   local spec    │     lockfile
                 │
> list           │
   ↓             │
glob .sqlpkg/*/*/sqlpkg.json
┌─────────────┐  │  ┌─────────────┐
│ asg017/vss  │  │  │ does not    │
└─────────────┘  │  │ matter      │
┌─────────────┐  │  └─────────────┘
│ sqlite/stmt │  │
└─────────────┘  │
   ↓             │
sync the lockfile
┌─────────────┐  │  ┌─────────────┐
│ asg017/vss  │  →  │ asg017/vss  │
│ sqlite/stmt │  │  │ sqlite/stmt │
└─────────────┘  │  └─────────────┘
```

Phew.

### Checksums

So far we've assumed that there will be no problems downloading remote package assets to the user's machine. And indeed, in most cases there will be none. But just to be sure that everything was downloaded correctly, we'd better check the asset's _checksum_.

Calculating the actual checksum of the downloaded asset is easy — we'll just use the SHA-256 algorithm. But we also need a value to compare it to — the expected asset checksum.

We can specify checksums right in the package spec file:

```
{
    "owner": "asg017",
    "name": "vss",
    "version": "v0.1.1",
    "repository": "https://github.com/asg017/sqlite-vss",
    "assets": {
        "path": "https://github.com/asg017/sqlite-vss/releases/download/v0.1.1",
        "files": {
            "darwin-amd64": "vss-macos-x86_64.tar.gz",
            "darwin-arm64": "vss-macos-aarch64.tar.gz",
            "linux-amd64": "vss-linux-x86_64.tar.gz"
        },
        "checksums": {
            "vss-macos-x86_64.tar.gz": "sha256-a3694a...",
            "vss-macos-aarch64.tar.gz": "sha256-04dc3c...",
            "vss-linux-x86_64.tar.gz": "sha256-f9cc84..."
        }
    }
}
```

But this would require the package author to edit the spec _after_ each release, since the checksums are not known in advance.

It's much better to provide the checksums in a separate file (e.g. `checksums.txt`) that is auto-generated with each new release. Such a file is hosted along with other package assets:

```
https://github.com/asg017/sqlite-vss/releases/download/v0.1.1
├── checksums.txt
├── vss-macos-x86_64.tar.gz
├── vss-macos-aarch64.tar.gz
└── vss-linux-x86_64.tar.gz
```

When installing the package, the manager fetches `checksums.txt`, injects it into the local spec file, and validates the downloaded asset checksum against the expected value:

```
    local assets      │      local spec        │     remote assets
                      │                        │
> install             │                        │  ┌──────────────────┐
   └─ (empty)         →        (empty)         →  │ asg017/vss       │
                      │                        │  ├──────────────────┤
                      │                        │  │ checksums.txt    │
   ┌─                 ←          ┌─            ←  │ macos-x86.tar.gz │
download asset        │  save spec w/checksums │  └──────────────────┘
┌──────────────────┐  │  ┌──────────────────┐  │
│ macos-x86.tar.gz │  │  │ asg017/vss       │  │
└──────────────────┘  │  ├──────────────────┤  │
   ↓                  │  │ macos-x86.tar.gz │  │
calculate checksum    │  │ sha256-a3694a... │  │
┌──────────────────┐  │  └──────────────────┘  │
│ sha256-a3694a... │  │                        │
└──────────────────┘  │                        │
   ↓                  │                        │
verify checksum       │                        │
  ↪ ✗ abort if failed │    asg017/vss          │
┌──────────────────┐  │  ┌──────────────────┐  │
│ macos-x86.tar.gz │  │  │ macos-x86.tar.gz │  │
│ sha256-a3694a... │  =  │ sha256-a3694a... │  │
└──────────────────┘  │  └──────────────────┘  │
   ↓                  │                        │
install asset         │                        │
┌──────────────────┐  │                        │
│ vss0.dylib       │  │                        │
└──────────────────┘  │                        │
✓ done!
```

If the remote package is missing `checksums.txt`, the manager can warn the user or even refuse to install such a package.

### Package dependencies

Okay, it's time to talk about the elephant in the room — package dependencies.

A package dependency is when package A depends on package B:

```
┌─────┐     ┌─────┐
│  A  │ ──> │  B  │
└─────┘     └─────┘
```

A transitive dependency is when package A depends on B, and B depends on C, so A depends on C:

```
┌─────┐     ┌─────┐     ┌─────┐
│  A  │ ──> │  B  │ ──> │  C  │
└─────┘     └─────┘     └─────┘
```

Dependencies, especially transitive ones, are a major headache (read Sam's article if you don't believe me). Fortunately, in the SQLite world, extensions are usually self-contained and don't depend on other extensions. So getting rid of the dependency feature is an obvious choice. It radically simplifies things.

```
┌─────┐     ┌─────┐     ┌─────┐
│  A  │     │  B  │     │  C  │
└─────┘     └─────┘     └─────┘
```

Since all packages are independent, the manager can install and update them individually without worrying about version conflicts.

I understand that dropping dependencies altogether may not be something you are ready to accept. But all the other building blocks we've discussed are still relevant regardless of the dependency handling strategy, so let's leave it at that.

### Install and update

Now that we've seen all the building blocks, let's look at two of the most complex commands: `install` and `update`.

Suppose I tell the manager to install the `asg017/vss` package:

```
   local spec    │     lockfile      │    remote spec
                 │                   │
> install asg017/vss                 │
   ↓             │                   │
read remote spec │                   │  ┌─────────────┐
   └─            →         →         →  │ asg017/vss  │
                 │                   │  │ latest      │
                 │                   │  └─────────────┘
                 │                   │    ↓
                 │                   │  resolve version
                 │                   │  ┌─────────────┐
                 │                   │  │ asg017/vss  │
   ┌─            ←         ←         ←  │ v0.1.0      │
download spec    │                   │  └─────────────┘
┌─────────────┐  │                   │
│ asg017/vss  │  │                   │
│ v0.1.0      │  │                   │
└─────────────┘  │                   │
   ↓             │                   │
download assets  │                   │
validate checksums                   │
  ↪ ✗ abort if failed                │
   ↓             │                   │
install assets   │                   │
┌─────────────┐  │                   │
│ vss0.dylib  │  │                   │
└─────────────┘  │                   │
   └─            →  add to lockfile  │
                 │  ┌─────────────┐  │
                 │  │ asg017/vss  │  │
                 │  │ v0.1.0      │  │
                 │  └─────────────┘  │
✓ done!
```

Now let's say I heard there was a new release, so I tell the manager to update the package:

```
   local spec    │     lockfile      │    remote spec
                 │                   │
> update asg017/vss                  │
   ↓             │                   │
read local spec  │                   │
  ↪ abort if failed                  │
┌─────────────┐  │  ┌─────────────┐  │
│ asg017/vss  │  │  │ does not    │  │
│ v0.1.0      │  │  │ matter      │  │
└─────────────┘  │  └─────────────┘  │
   ↓             │                   │
read remote spec │                   │
resolve version  │                   │  ┌─────────────┐
   └─            →         →         →  │ asg017/vss  │
   ┌─            ←         ←         ←  │ v0.1.1      │
has new version? │                   │  └─────────────┘
  ↪ ✗ abort if not                   │
┌─────────────┐  │                   │  ┌─────────────┐
│ v0.1.0      │  <   is less than    <  │ v0.1.1      │
└─────────────┘  │                   │  └─────────────┘
   ↓             │                   │
download assets  │                   │
validate checksums                   │
  ↪ ✗ abort if failed                │
   ↓             │                   │
install assets   │                   │
add to lockfile  │                   │
┌─────────────┐  │  ┌─────────────┐  │
│ asg017/vss  │  →  │ asg017/vss  │  │
│ v0.1.1      │  │  │ v0.1.1      │  │
└─────────────┘  │  └─────────────┘  │
┌─────────────┐  │                   │
│ vss0.dylib  │  │                   │
└─────────────┘  │                   │
✓ done!
```

Not so complicated after all, huh?

## Implementation details

I've written the package manager in Go. I believe Go is a great choice: not only is it reasonably fast and compiles to native code, but it's also the simplest of the mainstream languages. So I think you'll be able to easily follow the code even if you don't know Go. Also, porting the code to another language should not be a problem.

Another benefit of using Go is it's well thought out standard library. It allowed me to implement the whole project with zero dependencies, which is always nice.

[spec](#spec-package) • [assets](#assets-package) • [checksums](#checksums-package) • [lockfile](#lockfile-package) • [cmd](#cmd-package) • [top-level](#command-packages)

### `spec` package

The `spec` package provides data structures and functions related to the spec file.

```
  spec
┌─────────────────────────────────────┐
│ Package{}     Read()         Dir()  │
│ Assets{}      ReadLocal()    Path() │
│ AssetPath{}   ReadRemote()          │
└─────────────────────────────────────┘
```

The spec file and its associated data structures are the heart of the system:

```
// A Package describes the package spec.
type Package struct {
    Owner       string
    Name        string
    Version     string
    Homepage    string
    Repository  string
    Specfile    string
    Authors     []string
    License     string
    Description string
    Keywords    []string
    Symbols     []string
    Assets      Assets
}

// Assets are archives of package files, each for a specific platform.
type Assets struct {
    Path      *AssetPath
    Pattern   string
    Files     map[string]string
    Checksums map[string]string
}

// An AssetPath describes a local file path or a remote URL.
type AssetPath struct {
    Value    string
    IsRemote bool
}
```

We've already discussed the most important `Package` fields in the Design section. The rest (`Homepage`, `Authors`, `License`, and so on) provide additional package metadata.

The `Package` structure provides the basic spec management methods:

- `ExpandVars` substitutes variables in `Assets` with real values.
- `ReplaceLatest` forces a specific package version instead of the "latest" placeholder.
- `AssetPath` determines the asset url for a specific platform (OS + architecture).
- `Save` writes the package spec file to the specified directory.

`Assets.Pattern` provides a way to selectively extract files from the archive. It accepts a glob pattern. For example, if the package asset contains many libraries, and we only want to extract the `text` one, the `Assets.Pattern` would be `text.*`.

The `Read` family of functions loads the package spec file from the specified path (either local or remote).

Finally, the `Dir` and `Path` functions return the directory and spec file path of the installed package.

### `assets` package

The `assets` package provides functions for managing the actual package assets.

```
  assets
┌──────────────────────┐
│ Asset{}   Download() │
│           Copy()     │
│           Unpack()   │
└──────────────────────┘
```

An `Asset` is a binary file or an archive of package files for a particular platform:

```
type Asset struct {
    Name     string
    Path     string
    Size     int64
    Checksum []byte
}
```

The `Asset` provides a `Validate` method that checks the asset's checksum against the provided checksum string.

The `Download`, `Copy` and `Unpack` package-level functions perform corresponding actions on the asset.

The `assets` and `spec` packages are independent, but both are used by a higher level `cmd` package, which we'll discuss later.

### `checksums` package

The `checksums` package has one job — it loads asset checksums from a file (`checksums.txt`) into a map (which can be assigned to the `spec.Package.Assets.Checksums` field).

```
  checksums
┌──────────┐
│ Exists() │
│ Read()   │
└──────────┘
```

`Exists` checks if a checksum file exists in the given path. `Read` loads checksums from a local or remote file into a map, where keys are filenames and values are checksums. Pretty simple stuff.

Similar to `assets`, `checksums` and `spec` packages are independent, but both are used by a higher level `cmd` package.

### `lockfile` package

Just like the `spec` package works with the spec file, the `lockfile` works with the lockfile.

```
  lockfile
┌──────────────────────────┐
│ Lockfile{}   ReadLocal() │
│              Path()      │
└──────────────────────────┘
```

A `Lockfile` describes a collection of installed packages:

```
type Lockfile struct {
    Packages map[string]*spec.Package
}
```

It has a bunch of package-related methods:

- `Has` checks if a package is in the lockfile.
- `Add` adds a package to the lockfile.
- `Remove` removes a package from the lockfile.
- `Range` iterates over packages from the lockfile.
- `Save` writes the lockfile to the specified directory.

Since the `Lockfile` is always local, there is only one read function — `ReadLocal`. The `Path` function returns the path to the lockfile.

The `lockfile` package depends on the `spec`:

```
┌──────────┐   ┌──────────┐
│ lockfile │ → │   spec   │
└──────────┘   └──────────┘
```

### `cmd` package

The `cmd` package provides command steps — the basic building blocks for top-level commands like `install` or `update`.

```
  cmd
┌─────────────────────────────────────────────────────────────────────────────┐
│ assets              spec                lockfile             version        │
├─────────────────────────────────────────────────────────────────────────────┤
│ BuildAssetPath      ReadSpec            ReadLockfile         ResolveVersion │
│ DownloadAsset       FindSpec            AddToLockfile        HasNewVersion  │
│ ValidateAsset       ReadInstalledSpec   RemoveFromLockfile                  │
│ UnpackAsset         ReadChecksums                                           │
│ InstallFiles                                                                │
│ DequarantineFiles                                                           │
└─────────────────────────────────────────────────────────────────────────────┘
```

Each step falls into a specific domain category, such as "assets" or "spec".

Steps use the `spec`, `assets` and `lockfile` packages we've discussed earlier. Let's look at the `DownloadAsset` step for example (error handling omitted for brevity):

```
// DownloadAsset downloads the package asset.
func DownloadAsset(pkg *spec.Package, assetPath *spec.AssetPath) *assets.Asset {
    logx.Debug("downloading %s", assetPath)
    dir := spec.Dir(os.TempDir(), pkg.Owner, pkg.Name)
    fileio.CreateDir(dir)

    var asset *assets.Asset
    if assetPath.IsRemote {
        asset = assets.Download(dir, assetPath.Value)
    } else {
        asset = assets.Copy(dir, assetPath.Value)
    }

    sizeKb := float64(asset.Size) / 1024
    logx.Debug("downloaded %s (%.2f Kb)", asset.Name, sizeKb)
    return asset
}
```

I think it's pretty obvious what's going on here: we create a temporary directory and then download (or copy) the asset file into it.

The `logx` and `fileio` packages provide helper functions for logging and working with the file system. There are also `httpx` for HTTP and `github` for GitHub API calls.

Let's look at another one — `HasNewVersion`:

```
// HasNewVersion checks if the remote package is newer than the installed one.
func HasNewVersion(remotePkg *spec.Package) bool {
    installPath := spec.Path(WorkDir, remotePkg.Owner, remotePkg.Name)
    if !fileio.Exists(installPath) {
        return true
    }

    installedPkg := spec.ReadLocal(installPath)
    logx.Debug("local package version = %s", installedPkg.Version)

    if installedPkg.Version == "" {
        // not explicitly versioned, always assume there is a later version
        return true
    }

    if installedPkg.Version == remotePkg.Version {
        return false
    }

    return semver.Compare(installedPkg.Version, remotePkg.Version) < 0
}
```

It's pretty simple, too: we load the locally installed spec file and compare its version with the version from the remote spec. The `semver` helper package does the actual comparison.

The `cmd` package depends on all the packages we've already discussed:

```
┌────────────────────────────────────────────────┐
│                       cmd                      │
└────────────────────────────────────────────────┘
      ↓           ↓          ↓             ↓
┌──────────┐   ┌──────┐  ┌────────┐  ┌───────────┐
│ lockfile │ → │ spec │  │ assets │  │ checksums │
└──────────┘   └──────┘  └────────┘  └───────────┘
```

### Command packages

There is a top-level package for each package manager command:

- `cmd/install` installs packages.
- `cmd/update` updates installed packages.
- `cmd/uninstall` removes installed packages.
- `cmd/list` shows installed packages.
- `cmd/info` shows package information.
- and so on.

Let's look at one of the most complex commands — `update` (error handling omitted for brevity):

```
func Update(args []string) {
    fullName := args[0]
    installedPkg := cmd.ReadLocal(fullName)

    pkg := cmd.ReadSpec(installedPkg.Specfile)
    cmd.ResolveVersion(pkg)
    if !cmd.HasNewVersion(pkg) {
        return
    }

    cmd.ReadChecksums(pkg)

    assetUrl := cmd.BuildAssetPath(pkg)
    asset := cmd.DownloadAsset(pkg, assetUrl)

    cmd.ValidateAsset(pkg, asset)
    cmd.UnpackAsset(pkg, asset)
    cmd.InstallFiles(pkg, asset)
    cmd.DequarantineFiles(pkg)

    lck := cmd.ReadLockfile()
    cmd.AddToLockfile(lck, pkg)
}
```

Thanks to the building blocks in the `cmd` package, the update logic has become straightforward and self-explanatory. Just a linear sequence of steps with a single "does it have a new version?" branch.

Here is a complete package diagram (some arrows omitted to make it less noisy):

```
┌─────────┐ ┌────────┐ ┌───────────┐ ┌──────┐
│ install │ │ update │ │ uninstall │ │ list │ ...
└─────────┘ └────────┘ └───────────┘ └──────┘
     ↓          ↓            ↓          ↓
┌─────────────────────────────────────────────────┐
│                       cmd                       │
└─────────────────────────────────────────────────┘
      ↓           ↓           ↓             ↓
┌──────────┐   ┌──────┐   ┌────────┐  ┌───────────┐
│ lockfile │ → │ spec │   │ assets │  │ checksums │
└──────────┘   └──────┘   └────────┘  └───────────┘
┌────────┐ ┌────────┐ ┌───────┐ ┌──────┐ ┌────────┐
│ fileio │ │ github │ │ httpx │ │ logx │ │ semver │
└────────┘ └────────┘ └───────┘ └──────┘ └────────┘
```

And that's it!

[Source code](https://github.com/nalgeon/sqlpkg-cli)

## Summary

We've explored design choices for a simple general-purpose package manager:

- A package spec file describing a package.
- A hierarchical owner/name folder structure for installed packages.
- Project and global scope for installed packages.
- Spec file locator with fallback to the package registry.
- Versioning and latest versions.
- The lockfile and single source of truth.
- Asset checksums.
- Package dependencies (or lack thereof).

We've also explored implementation details in Go:

- `spec` package with data structures and functions related to the spec file.
- `assets` package for managing the package assets.
- `checksums` package for loading asset checksums from a file.
- `lockfile` package for working with the lockfile.
- `cmd` package with basic building blocks for top-level commands.
- top-level packages for individual commands.

Thanks for reading! I hope you'll find this article useful if you ever need to implement a package manager (or parts of it).

[★ Subscribe](app://obsidian.md/subscribe/) to keep up with new posts.