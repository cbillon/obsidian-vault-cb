---
link: https://www.geeksforgeeks.org/go-language/how-to-check-pointer-or-interface-is-nil-or-not-in-golang/
byline: GeeksforGeeks
site: GeeksforGeeks
date: 2020-05-01T12:22
excerpt: In Golang, nil check is frequently seen in GoLang code especially for error check. In most cases, nil check is straight forward, but in interface case, it's a bit different and special care needs to be taken. Here the task is to check pointer or interface is nil or not
tags:
  - go
  - interface
  - nil
  - pointer
slurped: 2025-04-07T16:01:00
title: How to check pointer or interface is nil or not in Golang?
---

Last Updated : 12 Jul, 2025

In [Golang](https://www.geeksforgeeks.org/go-language/go/), nil check is frequently seen in GoLang code especially for error check. In most cases, **nil** check is straight forward, but in interface case, it's a bit different and special care needs to be taken. Here the task is to check pointer or interface is nil or not in Golang, you can check with the following: **Example 1:** In this example, the pointer is checked whether it is a nil pointer or not.

// Go program to illustrate how to
// check pointer is nil or not

package main

import (
    "fmt"
)

type Temp struct {
}

func main() {
    var pnt *Temp // pointer
    var x = "123"
    var pnt1 *string = &x

    fmt.Printf("pnt is a nil pointer: %v\n", pnt == nil)
    fmt.Printf("pnt1 is a nil pointer: %v\n", pnt1 == nil)
}

**Output:**

pnt is a nil pointer: true
pnt1 is a nil pointer: false

**Example 2:** In this example, the interface is checked whether it is a nil interface or not.

// Go program to illustrate how to
// check interface is nil or not

package main

import (
    "fmt"
)

// Creating an interface
type tank interface {

    // Methods
    Tarea() float64
    Volume() float64
}

type myvalue struct {
    radius float64
    height float64
}

// Implementing methods of
// the tank interface
func (m myvalue) Tarea() float64 {

    return 2*m.radius*m.height +
        2*3.14*m.radius*m.radius
}

func (m myvalue) Volume() float64 {

    return 3.14 * m.radius * m.radius * m.height
}

func main() {

    var t tank
    fmt.Printf("t is a nil interface: %v\n", t == nil)

    t = myvalue{10, 14}

    fmt.Printf("t is a nil interface: %v\n", t == nil)

}

**Output:**

t is a nil interface: true
t is a nil interface: false

**Example 3:** In this example, the interface holding a nil pointer is checked whether it is a nil interface or not.

// Go program to illustrate how to
// check interface holding a nil
// pointer is nil or not

package main

import (
    "fmt"
)

type Temp struct {
}

func main() {

    // taking a pointer
    var val *Temp

    // taking a interface
    var val2 interface{}

    // val2 is a non-nil interface
    // holding a nil pointer (val)
    val2 = val

    fmt.Printf("val2 is a nil interface: %v\n", val2 == nil)

    fmt.Printf("val2 is a interface holding a nil"+
            " pointer: %v\n", val2 == (*Temp)(nil))
}

**Output:**

val2 is a nil interface: false
val2 is a interface holding a nil pointer: true