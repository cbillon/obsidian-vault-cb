---
link: https://dev.to/andyhaskell/intro-to-bubble-tea-in-go-21lg
byline: Andy Haskell
site: DEV Community
date: 2022-10-23T23:23
excerpt: Doing stuff in the command line is cool and can make you feel like you're the hero in a hacker movie....
twitter: https://twitter.com/@
tags:
  - go
  - bubble
slurped: 2025-07-25T10:46
title: Intro to Bubble Tea in Go
---

Doing stuff in the command line is cool and can make you feel like you're the hero in a hacker movie. But it can also feel old-school with monochromatic avalanches of text, or intimidating with all the command line flags and dollar sign prefixes and different ways to break things without warning.

But the command line isn't the only way to use your terminal, there's also **TUIs**, terminal user interfaces, which give a more user-friendly feel to your program. I really like TUIs because they feel like the new and the old at the same time. And in Go, lately the [Bubble Tea](app://github.com/charmbracelet/bubbletea) library, and [all of Charm Bracelet's other tools](https://charm.sh/), has been getting a lot of attention for making it easy to make TUIs.

The advantages of Bubble Tea that have jumped out at me so far are:

- Uses the [Elm architecture](https://guide.elm-lang.org/architecture/) that's shared with browser UI frameworks, so if you've already done some React, Vue, or Elm, it will feel familiar
- The Elm architecture isn't just familiar for modern frontend devs, it's a great way to organize UI code, so it's conducive to building starting your app simple and growing its logic in a manageable way
- Because it's in Go, the language's consistent syntax is conducive to learning by reading other people's code

In this series, I'm going to be building a basic TUI app from the ground up for logging what you've been learning in code each day if you're doing a program like #100Devs or one of the ones in the #100DaysOfCode family. At the time I'm writing this it's not finished yet, so each tutorial will be about looking at specific concepts. The approximate roadmap is going to be:

- 🐣 Writing a simple hello world app and seeing how its architecture works
- 📝 Building our first real Bubble Tea component, a menu
- ✨ Making our menu look cool with some styling, using the CSS-like [Lipgloss](app://github.com/charmbracelet/lipgloss) library
- 🚇 Adding routing to our app to display different pages
- 🫧 Using Bubble Tea components other people have made, using the [Bubbles](app://github.com/charmbracelet/bubbles) library
- 📝 Saving our check-ins to a JSON file

So get your terminal ready, and a boba-sized straw because without further ado it's time to jump into Bubble Tea!

## [](https://dev.to/andyhaskell/intro-to-bubble-tea-in-go-21lg#writing-our-first-basic-app)🚧 Writing our first basic app

As a first step, we're going to make a "hello world" app in Bubble Tea that you exit by pressing Ctrl+C, which will also introduce us to each part of a Bubble Tea app.

First, in a new directory titled "code-journal", run:

```


go mod init
go get github.com/charmbracelet/bubbletea


```

 

Then, create a file called `app.go` and add the following code:

```


package main

import (
    tea "github.com/charmbracelet/bubbletea"
)

func main() {
    p := tea.NewProgram(
        newSimplePage("This app is under construction"),
    )
    if err := p.Start(); err != nil {
        panic(err)
    }
}


```

 

Then, let's make another file called `simple_page.go` that contains our first UI, a simple page that just displays some text:

```

 go
package main

import (
    "fmt"
    "strings"

    tea "github.com/charmbracelet/bubbletea"
)

// MODEL DATA

type simplePage struct { text string }

func newSimplePage(text string) simplePage {
    return simplePage{text: text}
}

func (s simplePage) Init() tea.Cmd { return nil }

// VIEW

func (s simplePage) View() string {
    textLen := len(s.text)
    topAndBottomBar := strings.Repeat("*", textLen + 4)
    return fmt.Sprintf(
        "%s\n* %s *\n%s\n\nPress Ctrl+C to exit",
        topAndBottomBar, s.text, topAndBottomBar,
    )
}

// UPDATE

func (s simplePage) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg.(type) {
    case tea.KeyMsg:
        switch msg.(tea.KeyMsg).String() {
        case "ctrl+c":
            return s, tea.Quit
        }
    }
    return s, nil
}


```

 

Before we break down the code, let's run it and see what it does. In your terminal, run:

```


go build
./code-journal


```

 

and you should see something like this:

[![Terminal displaying the text](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frb5mwylhpft0xflquati.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Frb5mwylhpft0xflquati.png)

Cool! You've got your first Bubble Tea app running. Now let's take a closer look at the code.

## [](https://dev.to/andyhaskell/intro-to-bubble-tea-in-go-21lg#model-is-the-main-interface-of-bubble-tea)🧋 Model is the main interface of Bubble Tea

The main function starts the program by creating a new program with the `simplePage` model.

```


func main() {
    p := tea.NewProgram(
        newSimplePage("This app is under construction"),
    )
    if err := p.Start(); err != nil {
        panic(err)
    }
}


```

 

We call `tea.NewProgram`, whose signature is:

```


func NewProgram(initialModel Model) *Program


```

 

and then calling that program's Start method starts our app. But what is the `initialModel`?

`Model` is the main interface of Bubble Tea. It has three methods:

```


type Model interface {
    Init() Cmd
    Update(msg Msg) (Model, Cmd)
    View() string
}


```

 

The `Init` method is called when the app starts, returning a `tea.Cmd`. A `Cmd` is more or less "stuff happening behind the scenes" like loading data, or time flowing. But for the current tutorial, we don't have any background stuff, so our `init` method just returns `nil`.

```


func (s simplePage) Init() tea.Cmd { return nil }


```

 

Next up, we've got the `View` method. One of the cool abstractions of Bubble Tea is that **your whole UI's display is a string!** And `View` is where you make that string.

```


func (s simplePage) View() string {
    textLen := len(s.text)
    topAndBottomBar := strings.Repeat("*", textLen + 4)
    return fmt.Sprintf(
        "%s\n* %s *\n%s\n\nPress Ctrl+C to exit",
        topAndBottomBar, s.text, topAndBottomBar,
    )
}


```

 

So we put the text of our `simplePage` in a box made of asterisks, with a message at the bottom saying "Press Ctrl+C to exit"

But we can't exit our app if it doesn't handle user input, so that's where our `Update` method comes in.

```


func (s simplePage) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg.(type) {
    case tea.KeyMsg:
        switch msg.(tea.KeyMsg).String() {
        case "ctrl+c":
            return s, tea.Quit
        }
    }
    return s, nil
}


```

 

The `Update` method takes in a `tea.Msg` and returns a new `tea.Model` and sometimes a `tea.Cmd` (like if an action results in retrieving some data or a timer going off).

A `tea.Msg`'s type signature is

```


type Msg interface {}


```

 

So it can be any type and carry as much or as little data as you need. It's sort of like a browser event in JavaScript if you've done frontend there; a timer event doesn't carry any data, a click event tells you what clicked on, etc.

The kind of message we're processing is `tea.KeyMsg`, which represents keyboard input. We're checking if the user pressed Ctrl+C, and if so, we return the `tea.Quit` command, which is of the type `tea.Cmd` and tells Bubble Tea to exit the app.

For any other kind of input though, we don't do anything. We just return the model as-is. If we were doing something though like UI navigation, though, we would change some fields on the Model and then return it, causing the UI to update. And that's what we're going to see in the next tutorial where we make a menu component!