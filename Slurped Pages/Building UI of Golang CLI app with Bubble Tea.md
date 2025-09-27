---
link: https://medium.com/@originalrad50/building-ui-of-golang-cli-app-with-bubble-tea-68b61e25445e
byline: Vladimir Dulenov
site: Medium
date: 2024-01-06T21:57
excerpt: Building UI of Golang CLI app with Bubble Tea In this article, I’ll guide you through the process of integrating a BubbleTea terminal interface into a straightforward CLI application built on the …
twitter: https://twitter.com/@Medium
slurped: 2025-09-27T15:54
title: Building UI of Golang CLI app with Bubble Tea
tags:
  - golang
  - tui
---

In this article, I’ll guide you through the process of integrating a BubbleTea terminal interface into a straightforward CLI application built on the Cobra library.  
As a result, the interface will look like the GIF below (I recently changed themes and the help block looks really bad, but it might look better on your device).  
Let’s kick things off.

Press enter or click to view image in full size

Let’s kick things off!

## The main app logic

Now, let’s delve into the simplicity of the logic at the core of our application. Our task is straightforward — to retrieve the temperature of a given city using the arguments supplied through the CLI.

To do this, we will use the API, which is presented below:

func New() *FakeAPI {  
 return &FakeAPI{  
  name: "my-test-api",  
 }  
}

type WeatherResponse map[string]string

// GetWetherByCity returns map[date]temp for the given city  
func (api *FakeAPI) GetWetherByCity(city string) WeatherResponse {

 switch city {  
 case "berlin":  
  return WeatherResponse{  
   "today":    "+3",  
   "tomorrow": "+4",  
   "1.12":     "+3",  
  }  
 case "frankfurt":  
  return WeatherResponse{  
   "today":    "+1",  
   "tomorrow": "0",  
   "1.12":     "+4",  
  }  
......  
......  
......  
 default:  
  return WeatherResponse{  
   "today":    "+18",  
   "tomorrow": "+22",  
   "1.12":     "+20",  
  }  
 }  
}

Our CLI structure adheres to a well-defined schema, as illustrated in the diagram below:

internal/cli/  
├── root.go  
├── tui  
│   └── tui.go  
└── weather  
    ├── city  
    │   └── city.go  
    └── weather.go

At the helm is the `weather` command, acting as the main directive. Nested beneath it is the `city` subcommand, wielding two distinct arguments — `name`and `test-name`(an argument included solely for testing purposes).

Parallelly, we introduce the `tui` command, serving as the launchpad for our terminal interface. This structure sets the stage for a CLI application.  
Full code can be found [here](https://github.com/13excite/go_tea/tree/master/internal/cli/weather).

Now, let’s embark on the journey of integrating these components seamlessly. Let’s forge ahead!

## Bubble Teas basics

In the Bubble Tea framework, the `Model` interface plays a pivotal role, defining three essential methods that dictate the behavior and appearance of the application.

type Model interface {  
    Init() Cmd  
    Update(msg Msg) (Model, Cmd)  
    View() string  
}

### Init Method

The `Init`method is invoked when the application starts, initializing the initial state. In this tutorial, no background tasks are needed, so it returns nil. Background tasks, represented by Cmd in Bubble Tea, might include loading data or handling time-related operations.

func (m Model) Init() tea.Cmd { return nil }

### View Method

The `View` method is where the entire UI display is constructed as a string. Here, the `Model` content contains a simple table with pre-defined styles

var baseStyle = lipgloss.NewStyle().  
 BorderStyle(lipgloss.NormalBorder()).  
 BorderForeground(lipgloss.Color("240"))

func (m Model) View() string   
 return baseStyle.Render(m.table.View()) + "\n"  
}

### Update Method

The `Update` method handles user input (represented by `Msg`). In this case, it specifically processes keyboard input (`tea.KeyMsg`). If the user presses Ctrl+C, the method returns `tea.Quit`, signaling to Bubble Tea that the application should exit.

The `Msg` type in Bubble Tea is flexible and can carry various data. In this scenario, it resembles browser events in JavaScript. For instance, a timer event might not carry data, while a click event specifies what was clicked.

func (s Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {  
    switch msg.(type) {  
    case tea.KeyMsg:  
        switch msg.(tea.KeyMsg).String() {  
        case "ctrl+c":  
            return s, tea.Quit  
        }  
    }  
    return s, nil  
}

In essence, these three methods collaboratively define the life cycle and behavior of the Bubble Tea application, providing a clean and structured approach to building terminal interfaces.

### **Item Delegate of List**

Additionally, what we should know is ItemDelegate.

`ItemDelegate` encapsulates the general functionality for all list items. The benefit to separating this logic from the item itself is that you can change the functionality of items without changing the actual items themselves. In this example, overrides the `Render` method of `Delegate` to make the list items look better.

type itemDelegate struct{}  
......  
....  
func (d itemDelegate) Render(w io.Writer, m list.Model, index int, listItem list.Item) {  
 i, ok := listItem.(item)  
 if !ok {  
  return  
 }  
 str := fmt.Sprintf("%d. %s", index+1, i)  
 fn := itemStyle.Render  
 if index == m.Index() {  
  fn = func(s ...string) string {  
   return selectedItemStyle.Render(">> " + str)  
  }  
 }  
 fmt.Fprint(w, fn(str))  
}

## Building TUI

In our journey to enhance the functionality of our CLI application, we encounter the need to efficiently parse commands, handle subcommands, and manage flags. Furthermore, we aim to create an user interface that remains clutter-free by excluding certain commands such as `help` or `completion` from being displayed.

To achieve this, we’ll craft two functions that seamlessly handle command parsing, subcommand validation, and flag extraction.

Below is a function that goes through the command flags and generates a structure with the necessary information that will be used in UI.

// argument represents a single argument for the command  
type argument struct {  
 name        string  
 value       pflag.Value  
 description string  
}

// ignoredFlags contains a list of flags that should be ignored  
var ignoredFlags = []string{"version", "help", "yes"}

// getArgs returns a list of arguments of the command  
func getArgs(cmd *cobra.Command) []argument {  
 args := []argument{}  
 cmd.Flags().VisitAll(func(flag *pflag.Flag) {  
  val, ok := flag.Annotations[uiAnnotationKey]  
  if ok && (val[0] == "true") {  
   args = append(args, argument{  
    name:        flag.Name,  
    value:       flag.Value,  
    description: flag.Usage,  
   })  
  }  
 })  
 return args  
}

And the same logic, but for commands.

// subCommand represents a single subcommand for the command  
type subCommand struct {  
 name    string  
 command *cobra.Command  
}

// ignoredCommands contains a list of commands that should be ignored  
var ignoredCommands = []string{"help", "version", "completion", "tui"}

// getCobraSubCommands returns a list of subcommands of the command  
func getCobraSubCommands(cmd *cobra.Command) []subCommand {  
 commands := []subCommand{}  
 for _, command := range cmd.Commands() {  
  // mark command for O(1)  
  annotateCommandAdUIRelated(command)  
  val, ok := command.Annotations[uiAnnotationKey]  
  if ok && (val == "true") {  
   commands = append(commands, subCommand{  
    name:    command.Name(),  
    command: command,  
   })  
  }  
 }  
 return commands  
}

## Creating of UI elements

The model in this example consists of 3 main components: `TableView`, `textArea` and `ListView`.  
I separated the logic for creating these components into different functions, e.g.

func createInputView() textinput.Model {  
 txt := textinput.New()  
 txt.Cursor.Style = cursorStyle  
 txt.CharLimit = 40

 txt.Placeholder = ""  
 txt.PromptStyle = noStyle  
 txt.TextStyle = noStyle  
 return txt  
}

func createTextArea(text string) textarea.Model {  
 ta := textarea.New()  
 ta.Placeholder = "Flag description"  
 ta.BlurredStyle.Base = lipgloss.NewStyle().  
  Border(lipgloss.RoundedBorder()).  
  BorderForeground(lipgloss.Color("238"))  
 ta.SetValue(text)  
 ta.ShowLineNumbers = false  
 ta.Blur()  
 return ta  
}

As a result, the `View` method looks like this.  
The components are created and then the `JoinHorizontal` method distributes them horizontally.

type Model struct {  
 isFlagMode      bool // will be true if command doesn't have subcommands  
 textAreaEnabled bool // will be true if command has flags  
 showPopup       bool // if true popUp will be generated  
 popupMsg        string  
 submitButton    string  
 textAreaMsg     string // contains a flag desctiption of command  
 argument        argument // contains args of command  
 table           table.Model  
 list            list.Model  
 input           textinput.Model  
 userChoice      *UserChoice // set command for CLI which user has chosen in the UI  
 command         *cobra.Command  
 subCommands     []subCommand // contains subcommands of the command  
}

func (m Model) View() string {  
 if m.showPopup {  
  return fmt.Sprintf("\n\n%s\n\n%s\n\n[Enter] OK", m.popupMsg, strings.Repeat("-", 15))  
 }  
 table := baseStyle.Render(m.table.View())  
 input := m.input.View()  
 list := m.list.View()  
 button := m.submitButton

 helpString := helpStyleRender(`  
        Use the arrow keys to navigate.  
        ........  
 `)

 // if textArea is enabled and it's a flag mode  
 // then textArea form of the flags description will be shown  
 if m.isFlagMode && m.textAreaEnabled {  
  tArea := createTextArea(m.textAreaMsg)  
  return lipgloss.JoinHorizontal(  
   lipgloss.Top,  
   table,  
   lipgloss.JoinVertical(lipgloss.Top, "Flag description", tArea.View()),  
   docStyle.Render(list),  
  ) + "\n\n" + input + "\n" + button + "\n\n" + helpString  
 }

 return lipgloss.JoinHorizontal(  
  lipgloss.Top,  
  table,  
  docStyle.Render(list),  
 ) + "\n\n" + input + "\n" + button + "\n\n" + helpString  
}

### Implementation of Update method

As a result, the `Update` method supports the following control commands (I hid the full implementation, but you can find the link to the repo at the end of the article).

func (m Model) Update(msg tea.Msg) (tea.Model, tea.Cmd) {  
 var cmds []tea.Cmd

 switch msg := msg.(type) {  
 case tea.KeyMsg:  
  switch msg.String() {  
  case tea.KeyUp.String():  
   m.keyUpUpdateTextArea()  
  case tea.KeyDown.String():  
   m.keyDownUpdateTextArea()  
  case "ctrl+e", tea.KeySpace.String():  
   ....   
   return m, nil  
  case "tab":  
    ....  
    ....  
  case tea.KeyEscape.String():  
    ....  
    ....  
   // or exit  
   return m, tea.Quit  
  case tea.KeyCtrlC.String():  
   // clear a user choice before exit  
   m.userChoice.Command = nil  
   return m, tea.Quit  
  case "enter":  
   ...  
   ...  
   return m, nil  
  }  
 }  
.....  
.....  
 return m, tea.Batch(cmds...)  
}

It is also necessary to implement a number of interactive handlers to enrich the user experience in our CLI application (_you can find the link to the repo at the end of the article_). From transitioning between forms to managing lists and editing table fields, each handler contributes to a dynamic and user-centric interface. As we continue to refine these elements, our CLI application transforms to engaging tool for our users =)

// keyUpUpdateTextArea updates textArea for the previous flag if up button was pressed  
func (m *Model) keyUpUpdateTextArea() {  
 if m.table.Focused() && m.command.HasFlags() && m.isFlagMode {  
  cmdIndex := m.table.Cursor()  
  args := getArgs(m.command)  
  // decrease the table cursor index before it's not equal 0  
  switch cmdIndex {  
  case 0:  
   cmdIndex = 0  
  default:  
   cmdIndex--  
  }  
  m.textAreaMsg = args[cmdIndex].description  
 }  
}

func (m *Model) activateInput(cmdIndex int) {  
 m.argument = getArgs(m.command)[cmdIndex]  
 m.input.PromptStyle = focusedStyle  
 m.input.TextStyle = focusedStyle  
 m.input.Placeholder = fmt.Sprintf("Enter value for: %s", m.argument.name)  
 m.input.Focus()  
}

func (m *Model) switchToInputForm() {  
 m.table.Blur()  
 cmdIndex := m.table.Cursor()  
 m.activateInput(cmdIndex)  
}  
...  
...  
...

### Attaching TUI to Cobra

With our dynamic Bubble Tea interface in place, the next crucial step is to seamlessly integrate it with Cobra commands. This bridge between the interactive interface and the business logic encapsulated in our commands ensures a cohesive user experience. In this section, we’ll explore the process of connecting our Bubble Tea interface to Cobra commands.

The UI itself doesn’t contain any business logic, it only collects user input and passes it to user-selected commands. Executing Cobra commands within our Bubble Tea application ensures that the business logic is executed while the interface remains responsive and user-centric.

Here’s our code:

func NewTUICommand() *cobra.Command {  
 shellCmd := &cobra.Command{  
  Use:   "tui",  
  Short: "run terminal UI for the cli",  
  RunE: func(cmd *cobra.Command, args []string) error {  
   // stuct contains the command which user defined in UI  
   cmdSelectedByUser := &prompt.UserChoice{}  
   prompt.Interactive(cmd.Parent(), cmdSelectedByUser)

   var argList []string  
   if cmdSelectedByUser.Command != nil {  
    // generate a list of arguments for the Run method of commands  
    // We cannot use the method Execute here. Therefore, we need to get  
    // values which were set by the user and pass them to the Run method  
    cmdSelectedByUser.Command.Flags().VisitAll(func(flag *pflag.Flag) {  
     argList = append(argList, "--"+flag.Name, flag.Value.String())  
    })  
    // if command has a preRun method, then call it before Run method  
    // this method can be used for validation of user input  
    if cmdSelectedByUser.Command.PreRunE != nil {  
     err := cmdSelectedByUser.Command.PreRunE(cmdSelectedByUser.Command, argList)  
     if err != nil {  
      return err  
     }  
    }  
    err := cmdSelectedByUser.Command.RunE(cmdSelectedByUser.Command, argList)  
    if err != nil {  
     return fmt.Errorf("error apierred during the execution of the command: %v", err)  
    }  
   }

   return nil  
  },  
 }  
 return shellCmd  
}

## Conclusion

In this exploration of building a CLI application, we’ve navigated through the integration of a dynamic Bubble Tea interface with Cobra commands, creating a harmonious connection that empowers users while maintaining a clean and modular codebase. The significance of this union lies in the clarity of purpose: the interface excels at collecting user input, and Cobra commands handle the intricate business logic.

By implementing interactive handlers, managing form transitions, and editing table fields, we’ve not only crafted a feature-rich CLI but also ensured that the interface remains responsive to diverse user actions. The deliberate separation of concerns ensures scalability and maintainability, allowing for easy updates and expansions in the future.

## Links

Project repo: [https://github.com/13excite/go_tea](https://github.com/13excite/go_tea)  
Bubble tea repo: [https://github.com/charmbracelet/bubbletea](https://github.com/charmbracelet/bubbletea)  
Cobra repo: [https://github.com/spf13/cobra](https://github.com/spf13/cobra)