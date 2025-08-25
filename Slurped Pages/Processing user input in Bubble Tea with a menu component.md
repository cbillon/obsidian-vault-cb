---
link: https://dev.to/andyhaskell/processing-user-input-in-bubble-tea-with-a-menu-component-222i
byline: Andy Haskell
site: DEV Community
date: 2022-10-23T23:25
excerpt: In the last tutorial, we did a "hello world" app, and it processed just a bit of user input ("press...
twitter: https://twitter.com/@
tags:
  - go
  - bubble
slurped: 2025-07-25T10:49
title: Processing user input in Bubble Tea with a menu component
---
g
In the last tutorial, we did a "hello world" app, and it processed just a bit of user input ("press Ctrl+C to exit").

But we didn't really get a feel for actually using user input to change the model's data, and in turn change what we see in the app. So in this tutorial, we're going to create a menu component that lets us move between buttons.

## [](https://dev.to/andyhaskell/processing-user-input-in-bubble-tea-with-a-menu-component-222i#defining-our-data)📝 Defining our data

The first thing we need for any Bubble Tea component is the data our model is in charge of. If you recall, in our simplePage model, the data was just the text we were displaying:

```


type simplePage struct { text string }


```

 

In our menu, what we need to do is:

- Display our options
- Show which option is selected
- Additionally, let the user press the enter to go to another page. But we'll add that in a later tutorial.
    - For now, we can still make an onPress function passed in that tells us what we do if the user presses enter.

So our model's data will look like this; if you're following along, write this in a file named `menu.go`.

```


type menu struct {
    options       []menuItem
    selectedIndex int
}

type menuItem struct {
    text    string
    onPress func() tea.Msg
}


```

 

A menu is made up of menuItems, and each menuItem has text and a function handling pressing enter. In this tutorial we'll just have the app toggle between all-caps and all-lowercase so it's at least doing something.

It returns a `tea.Msg` because that's we're able to change the data in response to this user input. We'll see why in the next section, when we're implementing the `Model` interface.

## [](https://dev.to/andyhaskell/processing-user-input-in-bubble-tea-with-a-menu-component-222i#implementing-the-model-interface)🧋 Implementing the Model interface

If you recall, for us to use our model as a UI component, it needs to implement this interface:

```


type Model interface {
    Init() Cmd
    Update(msg Msg) (Model, Cmd)
    View() string
}


```

 

First let's write the Init function.

```


func (m menu) Init() tea.Cmd { return nil }


```

 

Again, we still don't have any initial `Cmd` we need to run, so we can just return `nil`.

For the `View` function, let's make an old-school menu with an arrow to tell us which item is currently selected.

```


func (m menu) View() string {
    var options []string
    for i, o := range m.options {
        if i == m.selectedIndex {
            options = append(options, fmt.Sprintf("-> %s", o.text))
        } else {
            options = append(options, fmt.Sprintf("   %s", o.text))
        }
    }
    return fmt.Sprintf(`%s

Press enter/return to select a list item, arrow keys to move, or Ctrl+C to exit.`,
    strings.Join(options, "\n"))
}


```

 

As mentioned in the last tutorial, one of the things that makes Bubble Tea really learnable is that the display for your UI is basically one big string. So in `menu.View` we make a slice of strings where the selected option has an arrow and the non-selected options have leading spaces. Then we join them all together and add our contols to the bottom.

Finally, let's write our Update method to handle user input.

```


func (m menu) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
    switch msg.(type) {
    case tea.KeyMsg:
        switch msg.(tea.KeyMsg).String() {
        case "ctrl+c":
            return m, tea.Quit
        case "down", "right", "up", "left":
            return m.moveCursor(msg.(tea.KeyMsg)), nil
        }
    }
    return m, nil
}

func (m menu) moveCursor(msg tea.KeyMsg) menu {
    switch msg.String() {
    case "up", "left":
        m.selectedIndex--
    case "down", "right":
        m.selectedIndex++
    default:
        // do nothing
    }

    optCount := len(m.options)
    m.selectedIndex = (m.selectedIndex + optCount) % optCount
    return m
}


```

 

The `Update` method is the most complex part of this app, so let's break that down.

```


case "ctrl+c":
    return m, tea.Quit


```

 

Like before, we're handling the `KeyMsg` type, and we're the Ctrl+C keypress to quit the app by returning the Quit cmd.

```


case "down", "right", "up", "left":
    return m.moveCursor(msg.(tea.KeyMsg)), nil


```

 

For the arrow keys, though, we use a helper function, `moveCursor`, which returns an updated model.

```


func (m menu) moveCursor(msg tea.KeyMsg) menu {
    switch msg.String() {
    case "up", "left":
        m.selectedIndex--
    case "down", "right":
        m.selectedIndex++
    default:
        // do nothing
    }

    optCount := len(m.options)
    m.selectedIndex = (m.selectedIndex + optCount) % optCount
    return m
}


```

 

The up and left KeyMsg strings serve as our "navigate up" keys, and the down and right ones navigate us down, decrementing and incrementing `m.selected`.

Then, we use the mod operator to ensure that `m.selected` is one of the indices of our options.

Finally, with the model updated, `moveCursor` returns the model that in turn is returned by `Update`, and the new model ultimately gets processed by our `View` method.

Before we move on to processing the enter key though, we should see our app run. So let's put our new `menu` component into a `main` function and run it.

```


func main() {
    m := menu{
        options: []menuItem{
            menuItem{
                text: "new check-in",
                onPress: func() tea.Msg { return struct{}{} },
            },
            menuItem{
                text: "view check-ins",
                onPress: func() tea.Msg { return struct{}{} },
            },
        },
    }

    p := tea.NewProgram(m)
    if err := p.Start(); err != nil {
        panic(err)
    }
}


```

 

For now, onPress is just a no-op that returns an empty struct. Now, let's run our app.

```


go build
./check-ins


```

 

You should see something like this:

[![List of options you can select, with an arrow pointed to the selected one, and instructions at the bottom](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feogrt056069ea53lmwfy.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Feogrt056069ea53lmwfy.png)

Cool! Now the menu can toggle what's selected! Now let's handle that user input.

## [](https://dev.to/andyhaskell/processing-user-input-in-bubble-tea-with-a-menu-component-222i#handling-the-enter-key-and-seeing-what-the-teacmd-type-actually-does)✅ Handling the enter key and seeing what the tea.Cmd type actually does

So far, we haven't really taken a close look at the `tea.Cmd` type. It's one of the two return values for the `Update` method, but we've only used it so far to exit the app. Let's take a closer look at its type signature.

```


type Cmd func() tea.Msg


```

 

A `Cmd` is some sort of function that does some stuff, and then gives us back a `tea.Msg`. That function can be time passing, it can be I/O like retrieving some data, really anything goes! The `tea.Msg` in turn gets used by our `Update` function to update our model and finally our view.

So handling a user pressing the enter key, and then running an arbitrary onPress function, is one such way to use a Cmd. So let's start with an enter button handler.

```


  case tea.KeyMsg:
      switch msg.(tea.KeyMsg).String() {
      case "q":
          return m, tea.Quit
      case "down", "right", "up", "left":
          return m.moveCursor(msg.(tea.KeyMsg)), nil
+     case "enter", "return":
+         return m, m.options[m.selectedIndex].onPress
      }


```

 

Notice that when the user presses enter, we return the model, unchanged, but we **also** return the selected item's `onPress` function. If you recall when we defined the `menuItem` type, the type of its `onPress` field was `func() tea.Msg`. In other words, that exactly matches the `Cmd` type alias!

There's one other thing we need to do inside the `Update` method though. Right now, we're only handling the `tea.KeyMsg` type. The type we're returning for toggling the selected item's capitalization will be a brand new type ot `tea.Msg`, so we need to define it, and then add a case to our Update method for it. First, let's define the struct.

```


type toggleCasingMsg struct{}


```

 

We don't need any data to be passed in, so our Msg is just an empty struct; if you recall, the `tea.Msg` type is just an empty interface, so we can have a Msg contain as much or as little data as we need.

Now back in the Update method, let's add a case for `toggleCasingMsg`!

First add the method `toggleSelectedItemCase`

```


func (m menu) toggleSelectedItemCase() tea.Model {
    selectedText := m.options[m.selectedIndex].text
    if selectedText == strings.ToUpper(selectedText) {
        m.options[m.selectedIndex].text = strings.ToLower(selectedText)
    } else {
        m.options[m.selectedIndex].text = strings.ToUpper(selectedText)
    }
    return m
}


```

 

Then add it to the `Update` method.

```


  func (m menu) Update(msg tea.Msg) (tea.Model, tea.Cmd) {
      switch msg.(type) {
+     case toggleCasingMsg:
+         return m.toggleSelectedItemCase(), nil
      case tea.KeyMsg:
          // our KeyMsg handlers here


```

 

On a toggleCasingMsg, we update the casing of the selected menu item, and then return the updated model.

Finally, in app.go, let's use our toggleCasingMsg

```


  menuItem{
      text:    "new check-in",
-     onPress: func() tea.Msg { return struct{}{} },
+     onPress: func() tea.Msg { return toggleCasingMsg{} },
  },
  menuItem{
      text:    "view check-ins",
-     onPress: func() tea.Msg { return struct{}{} },
+     onPress: func() tea.Msg { return toggleCasingMsg{} },
  },


```

 

Now let's try our app out!

```


go build
./check-ins


```

 

The app should now look like this:

[![List of options you can select, with an arrow pointed to the selected one that's now in all caps because the user had pressed enter, and instructions at the bottom](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlbzzbqoqv7qucl1pe8h.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fnlbzzbqoqv7qucl1pe8h.png)

Note, by the way, that at this stage of the app, this isn't the only way we could have processed enter; we also could have just processed all the toggling entirely in the update function, rather than having to process it with a Cmd. The reason I chose to use a Cmd were:

- To show a simple use case for a non-Quit Cmd in Bubble Tea
- By using a Cmd, we can pass arbitrary event handler functions into our components, a similar pattern if you've coded in React.

Next up, we've got a menu, but it's not very flashy just yet. In the next tutorial, we'll see how to use Bubble Tea to make our app look cool; first by hand, then with Bubble Tea's CSS-like Lip Gloss package!