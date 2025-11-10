---
link: https://blog.candys.page/pointers-in-go-when-to-use-the-ampersand-and-the-asterisk
byline: Candy
site: Candys Page
date: 2025-10-29T10:57
excerpt: |-
  Looking back, this might be a dumb question, but that’s what blog notes are for 🤣
  tl;dr

  &: Used as an operator to get the address of a variable

  * in Type: Used to define the type as a pointer (e.g., *int)

  * in Expression: Used as an operator to d...
slurped: 2025-11-09T09:26
title: Pointers in Go - When to use the ampersand and the asterisk?
tags:
  - go
  - pointer
---

Looking back, this might be a dumb question, but that’s what blog notes are for 🤣

## tl;dr

- `&`: Used as an **operator** to get the address of a variable
    
- `*` **in Type**: Used to **define the type** as a pointer (e.g., `*int`)
    
- `*` **in Expression**: Used as an **operator** to dereference
    

In short `*` does two things and `&` can only to that one thing.

## Example

Can be run here: [https://go.dev/play/p/XdA4_qdt7lR](https://go.dev/play/p/XdA4_qdt7lR)

```
package main

import "fmt"

func main() {
    value := 42 // The original data

    // --- 1. & gets the address ---
    address := &value
    fmt.Printf("1. & Action: get Address: %p\n", address)

    // --- 2. * defines the Pointer Type ---
    // 'p' is declared as the TYPE '*int' (a pointer type).
    var p *int = address
    fmt.Printf("2. Type Definition: 'p' is of TYPE %T\n", p)

    // --- 3. * dereferences ---
    // The '*' operator reads the value at the address stored in 'p'.
    retrievedValue := *p
    fmt.Printf("3. * Operator: Value retrieved via dereference: %d\n", retrievedValue)
}
```

## The struggle - Will the ampersand show up in type definition?

**NO, the ampersand will never be used in type definitions**, because it only does one thing, it gets the address of the variable. For type definitions we use `*`.

```
// 1. * used for type definition
func updateScore(p *int) {
    *p = 99 // 2. * used for dereference and change the value at that address
}
```

## Good Reads

If `*p = 99` feels confusing, read this article: [https://dave.cheney.net/2017/04/26/understand-go-pointers-in-less-than-800-words-or-your-money-back](https://dave.cheney.net/2017/04/26/understand-go-pointers-in-less-than-800-words-or-your-money-back)