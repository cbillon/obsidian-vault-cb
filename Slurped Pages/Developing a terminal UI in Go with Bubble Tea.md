---
link: https://packagemain.tech/p/terminal-ui-bubble-tea
byline: Alex Pliutau
site: packagemain.tech
date: 2025-07-07T18:04
excerpt: Developing CLIs and TUIs in Go is fun, and there are really good packages out there to make it so. Let's review one of them - Bubble Tea.
slurped: 2025-11-02T19:05
title: Developing a terminal UI in Go with Bubble Tea
tags:
  - go
  - bubble_tea
---

Many developers love using command line tools for daily development tasks, at least I do. It's fun, they are usually performant. For example there are many Desktop applications to work with git, though I believe that the majority of programmers love to use git CLI (or TUIs like [lazygit](https://github.com/jesseduffield/lazygit)) as it's faster and can be automated, and is the same on all environments: be it your dev machine or server. There are 2 main flavours of CLIs: one is imperative, command-based CLIs, such as git again, or kubectl, where you define everything in prompt using sub-commands and flags. And the second is so-called Terminal Apps or Terminal UI (commonly called TUI) which is more interactive, and is basically a whole application that runs in your terminal.

For example, there is a [circumflex](https://github.com/bensadeh/circumflex) TUI that lets you read Hacker News from your Terminal.

```
clx
```

[

![](https://substackcdn.com/image/fetch/$s_!3sWL!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe166c350-c15a-4493-9712-8febb39d6101_2670x1504.png)

](https://substackcdn.com/image/fetch/$s_!3sWL!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe166c350-c15a-4493-9712-8febb39d6101_2670x1504.png)

And there are more examples like that, check out [this article](https://packagemain.tech/p/essential-clitui-tools-for-developers) that lists just few.

Developing CLIs and TUIs in Go is fun, and there are really good packages out there to make it so. For example, for developing the command-based imperative CLIs you can use [Cobra](https://cobra.dev/) library from [Steve Francia](https://spf13.com/), which was used to build popular kubectl, hugo and github CLIs.

And when it comes to terminal apps, there is an amazing library called [Bubble Tea](https://github.com/charmbracelet/bubbletea) to build beautiful interactive TUIs.

[



For example the [circumflex](https://github.com/bensadeh/circumflex) TUI that we've seen before was developed using Bubble Tea.
In this article we will build a terminal-based note taking app using Bubble Tea and some other libraries. It will be a very simple application with SQLite store for storing the notes.

If you don’t have Bubble Tea installed yet, run the following commands to install it as well as few other adjacent packages :

```
go get github.com/charmbracelet/bubbletea
go get github.com/charmbracelet/lipgloss
go get github.com/charmbracelet/bubbles
```

Bubble Tea is usually used with other libraries, as you can see above, we installed them too.

[lipgloss](https://github.com/charmbracelet/lipgloss) is a great styling library from Charm.

[



And [bubbles](https://github.com/charmbracelet/bubbles) `is a TUI components library for Bubble Tea (also from Charm). For example File picker:`

[

![](https://substackcdn.com/image/fetch/$s_!u9WU!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5a2e24a4-16bb-4907-a370-16fdd0fb9b08_1200x600.gif)

](https://substackcdn.com/image/fetch/$s_!u9WU!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F5a2e24a4-16bb-4907-a370-16fdd0fb9b08_1200x600.gif)

Before jumping to our program, let’s review how Bubble Tea actually works.

Bubble Tea is based on the functional design paradigms of [The Elm Architecture](https://guide.elm-lang.org/architecture/), which happens to work nicely with Go. It's a delightful way to build applications.

Bubble Tea programs are comprised of a model that describes the application state and three simple methods on that model:

- **Init**, a function that returns an initial command for the application to run.
    
- **Update**, a function that handles incoming events and updates the model accordingly.
    
- **View**, a function that renders the UI based on the data in the model.
    

[

![](https://substackcdn.com/image/fetch/$s_!-Z_1!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F64ef01e8-05b0-4d3e-8a3e-78ec5b0a7ec6_1410x1016.png)

](https://substackcdn.com/image/fetch/$s_!-Z_1!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F64ef01e8-05b0-4d3e-8a3e-78ec5b0a7ec6_1410x1016.png)

Now, how does it translate to code?

A good place to start is the Model. Our note-taking application can be in multiple states (list view, edit title view, edit body view), and as you can imagine we will need UI elements such as text input, textarea, list. What’s cool is that our main model can contain other models as well, so in a way we’re building a tree of models.

[

![](https://substackcdn.com/image/fetch/$s_!UCEt!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F50bb8ad6-5ff6-450e-9b95-2c1dbd584df5_1572x1162.png)

](https://substackcdn.com/image/fetch/$s_!UCEt!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F50bb8ad6-5ff6-450e-9b95-2c1dbd584df5_1572x1162.png)

So here is how our model may look like:

[model.go](https://github.com/plutov/packagemain/blob/master/31-bubbletea/model.go)

```
package main

import (
	"log"

	"github.com/charmbracelet/bubbles/textarea"
	"github.com/charmbracelet/bubbles/textinput"
	tea "github.com/charmbracelet/bubbletea"
)

const (
	listView uint = iota
	titleView
	bodyView
)

type model struct {
	store     *Store
	state     uint
	textarea  textarea.Model
	textinput textinput.Model
	currNote  Note
	notes     []Note
	listIndex int
}

func NewModel(store *Store) model {
	notes, err := store.GetNotes()
	if err != nil {
		log.Fatalf("unable to get notes: %v", err)
	}

	return model{
		store:     store,
		state:     listView,
		textarea:  textarea.New(),
		textinput: textinput.New(),
		notes:     notes,
	}
}

func (m model) Init() tea.Cmd {
	return nil
}
```

As you can see our model is basically a state of our application. It also knows about the notes storage (sqlite in my case, but we will not focus on much on that). **NewModel()** function creates a new fresh state, and **Init()** is empty in our case, as initial command for the application is not required and not needed in our case.

With the model in place we can initiate a Bubble Tea program in [main.go](https://github.com/plutov/packagemain/blob/master/31-bubbletea/main.go)

```
package main

import (
	"log"

	tea "github.com/charmbracelet/bubbletea"
)

func main() {
	store := new(Store)
	if err := store.Init(); err != nil {
		log.Fatalf("unable to init store: %v", err)
	}

	m := NewModel(store)

	p := tea.NewProgram(m)
	if _, err := p.Run(); err != nil {
		log.Fatalf("unable to run tui: %v", err)
	}
}
```

As you can see, we can pass our model to **tea.NewProgram()** and Bubble Tea will do the rest for us, assuming that our Model implements the interface with **Init(), Update(), View()** methods.

```
type Model interface {
    Init() Cmd
    Update(msg Msg) (Model, Cmd)
    View() string
}
```

The **Update()** method handles user input (or any other events such as Ticks for example).

Here we can react to various events and update our model accordingly. For example, when a user presses the hotkeys, we switch to another view.

What’s interesting is that our Model contains other models so we must propagate the **Update()** accordingly.

```
func (m model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
	m.textarea, _ = m.textarea.Update(msg)
	m.textinput, _ = m.textinput.Update(msg)

	switch msg := msg.(type) {
	// handle key strokes
	case tea.KeyMsg:
		key := msg.String()

		switch m.state {
		// List View key bindings
		case listView:
			switch key {
			case "q":
				return m, tea.Quit
			case "n":
				// ...
			case "up", "k":
				// ...
			case "down", "j":
				// ...
			case "enter":
				// ...
			}

		// Title Input View key bindings
		case titleView:
			switch key {
			case "enter":
				// ...
			case "esc":
				m.state = listView
			}

		// Body Textarea key bindings
		case bodyView:
			switch key {
			case "ctrl+s":
				// ...
			case "esc":
				// ...
			}
		}
	}

	return m, nil
}
```

As you can see here in our **Update()** function we react to the following keystrokes:

- q - quit the app
    
- n - new note
    
- j,k - move up,down between notes
    
- enter - open note
    
- ctrl+s - save note
    
- esc - exit the step
    

The **Msg** type in Bubble Tea is flexible and can carry various data. In this scenario, it resembles browser events in JavaScript. For instance, a timer event might not carry data, while a click event specifies what was clicked.

But be aware, that messages are not necessarily received in the order they are sent, in Go, if you have more than one go routine sending to a channel, the order in which the sends and receives occur is unspecified.

The View method is where we take the state of our Model and render it. Here we will be using the libraries **libgloss** and **bubbles** that we installed previously.

[view.go](https://github.com/plutov/packagemain/blob/master/31-bubbletea/view.go)

```
package main

import (
	"strings"

	"github.com/charmbracelet/lipgloss"
)

var (
	appNameStyle = lipgloss.NewStyle().Background(lipgloss.Color("99")).Padding(0, 1)

	faint = lipgloss.NewStyle().Foreground(lipgloss.Color("255")).Faint(true)

	listEnumeratorStyle = lipgloss.NewStyle().Foreground(lipgloss.Color("99")).MarginRight(1)
)

func (m model) View() string {
	s := appNameStyle.Render("NOTES APP") + "\n\n"

	if m.state == titleView {
		s += "Note title:\n\n"
		s += m.textinput.View() + "\n\n"
		s += faint.Render("enter - save • esc - discard")
	}

	if m.state == bodyView {
		s += "Note:\n\n"
		s += m.textarea.View() + "\n\n"
		s += faint.Render("ctrl+s - save • esc - discard")
	}

	if m.state == listView {
		for i, n := range m.notes {
			// render each note
		}
		s += faint.Render("n - new note • q - quit")
	}

	return s
}
```

Again, as we have other models like textarea and textinput, we call View() on them too to embed them into our top-level view. Also, because the View() method returns a simple string, it becomes easy to test this deterministic function.

I didn’t include all the code, like SQLite storage and some utils, but you can find the [full program here](https://github.com/plutov/packagemain/tree/master/31-bubbletea) and even run it yourself, all you need is Go installed.

[

![](https://substackcdn.com/image/fetch/$s_!oTFb!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0b6a0e24-4c71-4403-be80-33b59d767b14_1224x426.png)

](https://substackcdn.com/image/fetch/$s_!oTFb!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F0b6a0e24-4c71-4403-be80-33b59d767b14_1224x426.png)

Building interactive TUIs in Go is genuinely enjoyable, thanks to powerful libraries like Bubble Tea.

To me the main strength of Bubble Tea is its simplicity with Elm Architecture, allowing developers to craft robust and responsive TUI applications, but also very modular.

The applications developed using Bubble Tea are usually very performant as well.

Let me know in the comments below if you built anything fun with Bubble Tea. I for example built this simple [ultrafocus](https://github.com/plutov/ultrafocus) TUI to block distracting websites and boost productivity.

