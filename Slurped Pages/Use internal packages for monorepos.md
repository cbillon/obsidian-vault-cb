---
link: https://konradreiche.com/blog/use-internal-packages-for-monorepos/
byline: Konrad Reiche
excerpt: |-
  Using the internal package in Go is a common way to limit your public API surface. The Go toolchain recognizes it and prevents external code from importing it. Typically, you place internal packages at the root of your repository or Go module to restrict external imports.
  However, I’ve recently discovered another valuable application for internal packages: maintaining import patterns within a single repository, especially in the context of a monorepo. Imagine you have a monorepo with multiple services (a, b, and c), each in its own directory, along with shared core libraries.
slurped: 2026-06-21T10:15
title: Use internal packages for monorepos
tags:
  - go
---

Oct 30, 2023

Using the `internal` package in Go is a common way to [limit your public API surface](https://dave.cheney.net/2019/10/06/use-internal-packages-to-reduce-your-public-api-surface). The Go toolchain recognizes it and prevents external code from importing it. Typically, you place internal packages at the root of your repository or Go module to restrict external imports.

However, I’ve recently discovered another valuable application for internal packages: maintaining import patterns within a single repository, especially in the context of a monorepo. Imagine you have a monorepo with multiple services (a, b, and c), each in its own directory, along with shared core libraries.

```
├── core
│   └── core.go
├── service_a
│   └── internal
│       └── service.go
├── service_b
│   └── internal
│       └── service.go
└── service_c
    └── internal
        └── service.go
```

All service packages can import code from the `core` package. However, due to their separate internal packages, they can’t import each other’s exported code. Trying to do so would result in a compile error:

```
use of internal package not allowed
```

Furthermore, the core package and its sub-packages are also restricted from importing exported code from the service directories.

Monorepos aren’t the only context where internal packages within a single repository are valuable. However, they provide a clear way to establish and implement boundaries between different packages, relieving maintainers from the burden of enforcing these structures during code reviews.