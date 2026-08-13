---
link: https://segflow.github.io/post/self-explained-function-signatures/
site: Segflow
date: 2024-09-26T00:00
excerpt: "A function I once shipped looked like this:"
slurped: 2026-07-03T18:27
title: Self-explained function signatures
tags:
  - golang
---

A function I once shipped looked like this:

```
func Process(data []byte) error
```

Innocent. You give it bytes, it returns an error. What more do you need?

Plenty, as it turned out. `Process` also wrote a file to `/tmp`, posted the same bytes to a webhook, and stashed a copy in a package-level global so the next call could “diff against the last one”. None of that was in the signature. None of it was in the doc comment either, because by the time anyone read the doc comment they’d already wired the function into a hot path and were wondering why CI was flaky.

The signature lied. Once a signature lies, every reader has to open the body to find the truth. That’s the cost.

This post is about a small habit with a big payoff: making function signatures carry their own intent. What the function needs. What it returns. What it might fail at. And what it _won’t_ secretly do behind your back.

Four rules. One before/after each.

## Rule 1: Inputs are explicit

If your function reads from a global, a package-level singleton, or an `os.Getenv` call, that input belongs in the signature.

The only ambient input I’ll allow is `context.Context`, because Go has standardised on it for cancellation and deadlines. Everything else goes through the door.

**Before:**

```
var defaultTimeoutMS = 5000

func FetchUser(id string) ([]byte, error) {
    // uses defaultTimeoutMS and a package-level http.Client
    ...
}
```

You can’t call this without knowing about a variable that isn’t mentioned in the signature. You can’t test it with a fake client. You can’t tell, from the signature alone, that it makes a network call.

**After:**

```
type HTTPClient interface {
    Get(ctx context.Context, url string) ([]byte, error)
}

func FetchUser(ctx context.Context, client HTTPClient, id string) ([]byte, error) {
    return client.Get(ctx, "/users/"+id)
}
```

Now the signature tells you three things: this thing is cancellable, it talks to something over HTTP, and you can swap that something in tests. No hidden wires.

## Rule 2: Outputs are explicit

Return what you compute. Don’t mutate the caller’s input unless the name of the function says, in big friendly letters, that you do.

The Go standard library is consistent about this. `sort.Slice` mutates, because “sort” mutates. `strings.ToUpper` returns a new string, because it doesn’t sound like it edits in place. When a function does need to mutate, the name says so: `Append`, `Sort`, or a pointer receiver.

**Before:**

```
// Clean removes duplicates and sorts. Also: ruins your input.
func Clean(events []string) {
    sort.Strings(events)
    j := 0
    for i, e := range events {
        if i == 0 || e != events[j-1] {
            events[j] = e
            j++
        }
    }
}
```

The name is “Clean”. It returns nothing. A reader has to guess what happened to `events`. Is the original still safe to use? No, and you only find out by reading the body or by your test failing.

**After:**

```
func Cleaned(events []string) []string {
    out := append([]string(nil), events...)
    sort.Strings(out)
    j := 0
    for i, e := range out {
        if i == 0 || e != out[j-1] {
            out[j] = e
            j++
        }
    }
    return out[:j]
}
```

The past-tense name and the return value together say: “you give me a slice, I give you a new one, your slice is fine.” If I really did need an in-place version, I’d call it `SortAndDedupInPlace` and accept the ugly name. The name is doing real work.

## Rule 3: Side effects show up in the name or the type

A function that writes to disk should either be called `Write…`, or take an `io.Writer`. A function that sends to a network should be called `Send…`, `Post…`, `Publish…`, or take a `Client`. A function that mutates a database should be called `Save…`, `Insert…`, `Update…`, or take a `Tx`.

If you can’t tell from the signature whether calling the function twice is safe, the signature is wrong.

**Before:**

```
func FormatReport(r Report) (string, error) {
    b, err := json.Marshal(r)
    if err != nil {
        return "", err
    }
    if err := os.WriteFile("/tmp/"+r.ID+".json", b, 0o644); err != nil {
        return "", err
    }
    return string(b), nil
}
```

The name is “Format”. Reasonable people will call this in a loop, in a template, in a test. The disk is going to fill up and nobody will know why.

**After (the name does the work):**

```
func WriteReport(path string, r Report) error {
    b, err := json.Marshal(r)
    if err != nil {
        return err
    }
    return os.WriteFile(path, b, 0o644)
}
```

**Or, even better, let the type do the work:**

```
func RenderReport(w io.Writer, r Report) error {
    return json.NewEncoder(w).Encode(r)
}
```

The `io.Writer` version is my favourite. The function has no idea where the bytes go. The caller decides: a file, a buffer, an HTTP response, a gzip stream. The side effect is the caller’s problem, and the test is one line: `RenderReport(&buf, r)` and compare.

## Rule 4: Errors are part of the signature

`error` is the laziest return type in Go. It tells you “something might go wrong” and nothing else. If a caller has to branch on _which_ thing went wrong, the signature has to tell them what’s possible.

That doesn’t mean returning fifteen distinct error types. It means: when your callers will reasonably handle one failure differently from another, expose a sentinel they can match on.

**Before:**

```
func Lookup(id string) (string, error) {
    if id == "" {
        return "", fmt.Errorf("not found")
    }
    if id == "x" {
        return "", fmt.Errorf("forbidden")
    }
    return "ok", nil
}
```

A caller who wants a 404 on “not found” and a 403 on “forbidden” now has to do `strings.Contains(err.Error(), "not found")`, and it will work until you fix a typo and silently break every caller. We’ve all done this.

**After:**

```
var (
    ErrNotFound  = errors.New("not found")
    ErrForbidden = errors.New("forbidden")
)

func Lookup(id string) (string, error) {
    if id == "" {
        return "", ErrNotFound
    }
    if id == "x" {
        return "", ErrForbidden
    }
    return "ok", nil
}
```

Now callers write `errors.Is(err, ErrNotFound)` and the compiler keeps them honest. The contract is: “this function can fail in these specific ways, and if I add a new one I’ll add a new sentinel and you’ll see it in the diff.”

That’s all sentinel errors are. A promise that the failure modes are part of the API, not the implementation.

## What you get back

The four rules pull in the same direction: a reader should be able to predict what a function does from its signature, and be right.

When that’s true, code review changes. Reviewers stop asking “does this also write to disk?”, “is this safe to call twice?”, “does this mutate the input?” Those questions are answered before the body is opened.

Tests change too. Explicit inputs mean no `TestMain` monkey-patching globals. Explicit outputs mean no scanning `/tmp` to assert behaviour. Errors in the signature mean error paths you can cover without parsing strings.

A short rule of thumb: if you need a comment to explain what the signature should have said, you don’t have a documentation problem. You have a signature problem.

---

Next time, the other half of the signature: errors. We covered the “they should exist” part here. The next post is about what they should look like, when wrapping helps and when it hides things, and why most `error` returns in your codebase are doing less work than they could.