---
link: https://segflow.github.io/post/against-utils-packages/
site: Segflow
date: 2025-01-09T00:00
excerpt: |-
  I argued earlier that a function’s signature should never
  lie, and then that errors are
  values only if you treat them like values. Both
  posts circled the same question at different scales: what is this code
  for? This one closes the loop at the package level. The short version: if
  your package is named utils, you don’t have an answer.
slurped: 2026-07-03T18:32
title: The case against helper packages named `utils`
tags:
  - golang
---

I argued earlier that [a function’s signature should never lie](app://obsidian.md/post/self-explained-function-signatures/), and then that [errors are values only if you treat them like values](app://obsidian.md/post/errors-are-values/). Both posts circled the same question at different scales: _what is this code for?_ This one closes the loop at the package level. The short version: if your package is named `utils`, you don’t have an answer.

## The grep that started it

I was reading a popular Go project on GitHub on a Sunday, the kind with thousands of stars and a logo, trying to learn how a piece of it worked. I ran this in the clone:

```
$ grep -rl "^package utils" . | wc -l
14
```

Fourteen `utils` packages. Different directories, different subsystems, different contents. One had retry logic and a SHA-256 helper. One had a `StringPtr(s string) *string` and three flavors of `Must`. One was just `SliceContains`. They imported each other through a chain that ran roughly:

```
cmd/api ──► internal/utils ──► pkg/utils ──► common/utils
                  │                │
                  ▼                ▼
            shared/utils ◄── infra/utils
```

That diagram is not unusual. I’ve seen the same shape in three other projects since. It’s what happens when every subsystem writes its own `utils` and then nobody agrees which one owns the next helper.

## Why `utils` is a name smell

Three failure modes, every time.

**Anti-cohesion.** A package is a unit of meaning. `auth`, `billing`, `pricing`, `httpx` all promise something. `utils` promises nothing except “we didn’t know where to put it.” Functions land there because the author ran out of patience, not because they belong together. Six months later nobody can describe what the package _is_.

**Import magnet, then primitive re-inventor.** `utils` is convenient, so everyone imports it. Once everyone imports it, it can’t import anyone back without risking a cycle. So when `utils` needs something from `auth` or `billing`, it doesn’t import them, it re-implements them. Now you have two copies of the same logic and a slowly diverging definition of “user.”

**Untestable culture turns into a graveyard.** It starts pure: a string helper, a slice helper. No domain, no tests needed, “it’s just utils.” Then someone adds `utils.NormalizeUserEmail` which knows the rules of _your_ product. Then `utils.ChargeIfTrial`. Domain logic sneaks in through the side door and lives somewhere with no owner and no tests.

## The rename game

Every helper in a `utils` package can be renamed into a real one. Five examples, each one line, each pointing at a concept that already has a name.

**1. `utils.RetryHTTP` → `httpx.Retry`.** It’s about HTTP. Put it next to HTTP.

```
package httpx

func Retry(do func() (*http.Response, error), n int, backoff time.Duration) (*http.Response, error) {
    var (
        resp *http.Response
        err  error
    )
    for i := 0; i < n; i++ {
        resp, err = do()
        if err == nil && resp.StatusCode < 500 {
            return resp, nil
        }
        if resp != nil {
            resp.Body.Close()
        }
        time.Sleep(backoff)
    }
    return resp, err
}
```

The package name carries half the meaning. `httpx.Retry` reads. `utils.RetryHTTP` reads like an apology.

**2. `utils.ParseDuration` → `timex.ParseLoose`.** The stdlib already owns `time.ParseDuration`. If you wrote your own, it does something extra (probably `"d"` for days). Say so in the name and the package.

```
package timex

func ParseLoose(s string) (time.Duration, error) {
    if strings.HasSuffix(s, "d") {
        d, err := time.ParseDuration(strings.TrimSuffix(s, "d") + "h")
        if err != nil {
            return 0, err
        }
        return d * 24, nil
    }
    return time.ParseDuration(s)
}
```

**3. `utils.SliceContains` → `slices.Contains`.** The stdlib has had this since Go 1.21. Delete yours.

```
import "slices"

ok := slices.Contains(users, "alice")
```

Half the `utils.go` files I’ve seen could be deleted by upgrading Go and running `gofmt -r`. Do that first.

**4. `utils.MustEnv` → `config.MustEnv`.** This one is sneaky. `MustEnv` isn’t a generic helper. It’s a _policy_: “at process start, this variable must be set, otherwise we refuse to boot.” That belongs with the rest of your configuration policy.

```
package config

func MustEnv(key string) string {
    v, ok := os.LookupEnv(key)
    if !ok || v == "" {
        panic(fmt.Errorf("config: missing required env var %q", key))
    }
    return v
}
```

**5. `utils.HashPassword` → `auth.HashPassword`.** If this one is in `utils` in your codebase, stop reading and go fix it. Password hashing is a security concern with an owner. Give it to the owner.

```
package auth

func HashPassword(plaintext string) (hash, salt string, err error) {
    // ...
}
```

Notice the pattern. None of these renames invented new packages out of nowhere. They moved each function next to the _concept_ it serves.

## The rule

If you can’t name the package after a concept, the code doesn’t belong together.

`httpx`, `timex`, `slices`, `config`, `auth` are concepts. `utils`, `helpers`, `common`, `misc`, `shared` are excuses. They tell you the author gave up.

A useful test: write the package’s first doc comment.

```
// Package httpx adds small extensions on top of net/http.
```

Try the same for `utils`:

```
// Package utils contains... uh... assorted things.
```

If you can’t finish the sentence without listing the functions, the package shouldn’t exist.

## Migration tip: shrink, don’t merge

The instinct when you find a 2,000-line `utils` package is to “clean it up” in one big PR. Don’t. You’ll spend two weeks rebasing and merge it half done.

Instead: every PR that touches `utils` is allowed to _only delete from it_.

- Move one function to its real home.
- Update callers.
- Open a 30-line PR.
- Merge.

After a dozen of those, `utils` is empty and you delete the directory. PRs that only delete are the most satisfying kind to review, and they never conflict with feature work.

If a function genuinely has no concept home (`Ptr[T any](v T) *T`, say), it’s almost always small enough to inline at the call site or to live in a 3-line `internal/ptr` package. Three lines is fine. Two thousand lines of “assorted” is not.

## Wrapping the trilogy

Three posts, one question.

Post 1 said: a function’s _signature_ should answer “what is this code for?” If you need a comment to clarify it, the signature is wrong.

Post 2 said: an _error_ should answer the same question on the way out. If you need to grep logs to figure out where it came from, the error is wrong.

This one says: a _package name_ answers the same question one level up. If you can’t name it after a concept, the package is wrong.

Signatures, errors, package names. Same conversation, three scales. The codebases I enjoy working in are the ones where every level of naming points at the thing it actually does. The ones I dread are the ones where every level shrugs.

Stop shrugging. Delete `utils`.