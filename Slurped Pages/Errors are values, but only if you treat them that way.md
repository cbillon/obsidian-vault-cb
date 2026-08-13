---
link: https://segflow.github.io/post/errors-are-values/
site: Segflow
date: 2024-11-14T00:00
excerpt: |-
  Last time I argued that a function’s signature should never
  lie. The error return is the
  most lied-about part of every Go signature I read. It gets logged,
  swallowed, repackaged, panicked on, and printed back to the user as
  internal error: internal error: internal error. Three habits keep it
  honest.
slurped: 2026-07-03T18:31
title: Errors are values, but only if you treat them that way
tags:
  - golang
  - error
---

Last time I argued that a function’s [signature should never lie](app://obsidian.md/post/self-explained-function-signatures/). The `error` return is the most lied-about part of every Go signature I read. It gets logged, swallowed, repackaged, panicked on, and printed back to the user as `internal error: internal error: internal error`. Three habits keep it honest.

## Habit 1: wrap with the verb and the inputs

If you only take one thing from this post, take this. When you wrap an error, the prefix must answer “what was I doing, with what?” in verb form. Not the name of the function. Not the package. The action.

This is the wrapper you find everywhere:

```
u, err := db.LoadUser(id)
if err != nil {
    return nil, fmt.Errorf("err: %w", err)
}
```

This is the same wrapper after it grew up:

```
u, err := db.LoadUser(id)
if err != nil {
    return nil, fmt.Errorf("loading user %d: %w", id, err)
}
```

By the time the caller logs this once at the edge, the chain reads:

```
GET /user?id=7: loading user 7: querying users table: open users.db: permission denied
```

The verb at every level, the inputs that mattered, exactly which `open` of which file failed. The trick is boring: `verb + nouns + %w`. Nothing else.

## Habit 2: typed errors at boundaries, plain `error` inside

The other thing people overdo is `errors.Is` / `errors.As` plumbing. A sentinel for every internal failure, exported, with a giant `switch` at the call site. A month later half the sentinels are dead and nobody dares delete them.

The rule I use: **typed errors live at the package boundary, and only when callers must branch on them.** Inside the package, a plain wrapped `error` is fine.

A boundary that callers will branch on:

```
type NotFoundError struct{ ID int }

func (e *NotFoundError) Error() string {
    return fmt.Sprintf("user %d not found", e.ID)
}

func FindUser(id int) (*User, error) {
    u, err := loadUserRow(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &NotFoundError{ID: id}
        }
        return nil, fmt.Errorf("finding user %d: %w", id, err)
    }
    return u, nil
}
```

`loadUserRow` is internal and its errors are never branched on, so it gets no sentinel of its own. It just wraps. If a caller one day needs to distinguish “row missing” from “DB exploded”, promote the case and define a type. Not before.

The caller side stays honest:

```
u, err := FindUser(id)
var nf *NotFoundError
switch {
case errors.As(err, &nf):
    // 404
case err != nil:
    // 500
}
```

Two branches at the boundary, none anywhere else. The public surface shrinks; the internal call sites stay quiet.

## Habit 3: log OR return, never both

If you return an error, you are telling the caller “you decide”. If you log it, you have already decided. Doing both is doubling the work and lying to whoever wired up the caller.

The smell:

```
u, err := db.LoadUser(id)
if err != nil {
    log.Printf("LoadUser: %v", err)
    return nil, err
}
```

Every layer that does this adds one more line to the eventual production output. That is how a single failure becomes the four “permission denied"s from the top of this post. The fix:

```
u, err := db.LoadUser(id)
if err != nil {
    return nil, fmt.Errorf("loading user %d: %w", id, err)
}
```

The handler at the edge owns the decision. It logs once, with the full wrapped chain, and writes a status code. Nothing below it should call `log` at all.

## Anti-pattern parade

Three smells, all common, all five seconds to fix:

```
// 1. Bare return: throws away the call site.
if err != nil { return err }

// 2. Log and return: now you log it 4 times.
log.Printf("%v", err); return err

// 3. Panic on something recoverable: kills the request because
//    a row was missing.
u, err := db.LoadUser(id)
if err != nil { panic(err) }
```

The first is “I wrote `if err != nil` and got bored”. The second is Habit 3 wearing a hat. The third shows up in glue code written fast and never revisited. None is harmless.

## Putting it together: an HTTP handler

The before is a handler that does all three sins at once. It logs at every layer, treats every failure the same, and leaks the DB error to the client:

```
func getUserBefore(w http.ResponseWriter, r *http.Request) {
    idStr := r.URL.Query().Get("id")
    id, err := strconv.Atoi(idStr)
    if err != nil {
        log.Printf("atoi: %v", err)
        http.Error(w, err.Error(), 500)
        return
    }
    u, err := db.LoadUser(id)
    if err != nil {
        log.Printf("LoadUser: %v", err)
        http.Error(w, err.Error(), 500)
        return
    }
    if err := json.NewEncoder(w).Encode(u); err != nil {
        log.Printf("encode: %v", err)
        return
    }
}
```

The after applies all three habits. One log site, one decision per branch, verb-form context, a typed error at the package boundary:

```
func loadUser(id int) (*User, error) {
    u, err := db.LoadUser(id)
    if err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return nil, &NotFoundError{ID: id}
        }
        return nil, fmt.Errorf("loading user %d: %w", id, err)
    }
    return u, nil
}

func getUserAfter(w http.ResponseWriter, r *http.Request) {
    id, err := strconv.Atoi(r.URL.Query().Get("id"))
    if err != nil {
        http.Error(w, "bad id", http.StatusBadRequest)
        return
    }
    u, err := loadUser(id)
    switch {
    case errors.As(err, new(*NotFoundError)):
        http.Error(w, "not found", http.StatusNotFound)
        return
    case err != nil:
        log.Printf("GET /user?id=%d: %v", id, err)
        http.Error(w, "internal error", http.StatusInternalServerError)
        return
    }
    if err := json.NewEncoder(w).Encode(u); err != nil {
        log.Printf("GET /user?id=%d: encode: %v", id, err)
    }
}
```

Same lines of code. A different log volume by an order of magnitude, and a production log line that actually points at the bug.

## What comes next

Once errors carry their own context, a quiet thing happens. You stop reaching for the `utils` package to “handle errors” or “pretty-print errors”, because the error already prints itself. You stop adding a `pkg/errs` with seven helper constructors, because `fmt.Errorf` is enough. The grab bag of helpers that every codebase grows starts to look like what it usually is: a symptom of values that were not doing their job.

That is the next post. The `utils` package, and how to delete it.