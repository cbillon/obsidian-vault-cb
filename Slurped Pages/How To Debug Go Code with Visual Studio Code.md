---
link: https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code
byline: Chris Ganga
site: DigitalOcean
date: 2019-12-12T06:00
excerpt: This article will go over how to debug your Go code in Visual Studio Code using the VSCode-Go plugin and breakpoints.
slurped: 2025-10-16T14:23
title: How To Debug Go Code with Visual Studio Code
tags:
  - go
  - debug
  - vs
---

### [Introduction](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#introduction)

This tutorial will discuss the steps necessary to debug [Go](https://golang.org/) code with [Visual Studio Code](https://code.visualstudio.com/). It will entail installing extensions, analysis tools, and debuggers.

First, we will create a sample application. Then, we will explore using breakpoints and conditional breakpoints.

With this skillset you will be better equipped to understand the value and state of your application at specific points in its code execution.

## [Prerequisites](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#prerequisites)

To complete this tutorial, you will need the following:

- An understanding of Go. To learn more, check out our [How To Code in Go](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-go) series.
- Go installed on your machine. To install Go on your machine, follow the tutorial to set up a local programming environment for your operating system from the [Go series](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-go).
- [Visual Studio Code](https://code.visualstudio.com/download) installed on your machine.
- The [VSCide-Go plugin](https://marketplace.visualstudio.com/items?itemName=golang.Go) installed.

Once you have the plugin installed, open any `.go` files in VS Code. You will be prompted on the bottom right of the status bar to **Install Analysis Tools**. Click on that link to install the necessary Go packages for the plugin to work efficiently.

We finally need to install [Delve](https://github.com/go-delve/delve), an open-source debugger for Go. To do this, there are [detailed installation instructions for specific platforms](https://github.com/go-delve/delve/tree/master/Documentation/installation).

## [Step 1 — Creating a Sample App](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#step-1-creating-a-sample-app)

We’ll use two examples to debug our Go code:

- A Go program that generates a JSON file.
- We’ll write a function, write the test, and see how we debug tests in VS Code.

Here’s the source code for the first example. Create a file `main.go`:

```
nano main.go

```

Add the following content to the file:

main.go

```
package main

import (
	"encoding/json"
	"fmt"
	"log"
)

// Avenger represents a single hero
type Avenger struct {
	RealName string `json:"real_name"`
	HeroName string `json:"hero_name"`
	Planet   string `json:"planet"`
	Alive    bool   `json:"alive"`
}

func (a *Avenger) isAlive() {
	a.Alive = true
}

func main() {
	avengers := []Avenger{
		{
			RealName: "Dr. Bruce Banner",
			HeroName: "Hulk",
			Planet:   "Midgard",
		},
		{
			RealName: "Tony Stark",
			HeroName: "Iron Man",
			Planet:   "Midgard",
		},
		{
			RealName: "Thor Odinson",
			HeroName: "Thor",
			Planet:   "Midgard",
		},
	}

	avengers[1].isAlive()

	jsonBytes, err := json.Marshal(avengers)
	if err != nil {
		log.Fatalln(err)
	}
	fmt.Println(string(jsonBytes))
}
```

Here we are defining a struct `Avenger`, and then creating an array of Avengers, changing the status of one of them to `"alive"`, then converting the results to JSON, and finally printing it to STDOUT.

You can run the app with:

```
go run main.go

```

It will produce this output:

```
Output[{"real_name":"Dr. Bruce Banner","hero_name":"Hulk","planet":"Midgard","alive":false},{"real_name":"Tony Stark","hero_name":"Iron Man","planet":"Midgard","alive":true},{"real_name":"Thor Odinson","hero_name":"Thor","planet":"Midgard","alive":false}]
```

## [Step 2 — Debugging with Breakpoints](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#step-2-debugging-with-breakpoints)

To get started with debugging, we need to create a configuration. Click on the Debug Icon on the left pane of Visual Studio Code. Next, click on the Gear Icon to create a configuration.

![Debug and Settings (Gear) icons](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/1.png)

A configuration file is created under `.vscode/launch.json` with the contents shown above. Change the configurations program to point to the `main.go` file. In this instance, since we only have a `main.go` file, we can change it to our workspace root:

launch.json

```
{
  // ...
  "configuration": [
    {
      // ...
      "program": "${workspaceRoot}",
      // ...
    }
  ]
}
```

Next we need to add a breakpoint, because that’s what debugging is all about.

Let’s add a breakpoint on **Line 21** (`func main()`) by clicking to the left of the line number. There, you will see a red dot.

![Red dot to the left of Line 22](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/2.png)

Next, either press `F5` or click on the Launch button with a green play button on the **Debug Section** on the top left to open the **Debug View**.

![Visual Studio Code Debug View](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/3.png)

Click multiple times on the `Step Over` button on the **Debug Toolbar**.

![Step Over icon in Debug Toolbar](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/4.png)

The debugger will eventually move to **Line 40**.

![Debugger moved to line 40](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/5.png)

The **Debug Section** on the left will give us the state of the current breakpoint position.

![Debug section showing breakpoint state](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/6.png)

We can see the state or value of the variables, at that particular time in the **Variables** section.

We can also see the the call stack, and at the moment the running function is the `main` function, and **Line 40**.

You can continue Stepping Over, you’ll see the value of `avengers` changes once we are past the line. `"Tony Stark"` is `Alive`.

![Debug view showing Tony Stark is still alive](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/7.png)

## [Step 3 — Adding Conditional Breakpoints](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#step-3-adding-conditional-breakpoints)

VS Code breakpoints provide you with an option to edit breakpoints by giving them an expression, which most of the time is usually a boolean expression.

For instance, on **Line 40**, `avengers[1].isAlive()`, we could add a condition here that the breakpoint is only raised when the expression evaluates to true, as in `avengers[1].Planet == "Earth"`.

To do this, right click on the breakpoint and select **Edit Breakpoint**.

!["Edit breakpoint…" menu option](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/8.png)

If you did not have a breakpoint, you can still right click, and you will be told to **Add Conditional Breakpoint**.

!["Add Conditional Breakpoint…" menu option](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/9.png)

Let’s add the condition once we’ve selected any of the above: `avengers[1].Planet == "Earth"`.

![Adding condition 'avengers[1].Planet == "Earth"'](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/10.png)

Now if you launch the debugger with `F5`, it will not stop at the breakpoint. The app will run fine, and you will see the results in the **Debug Console**.

![Debug console results](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/11.png)

Next, edit the code to match what we expect. Change Tony Stark’s planet to `Earth`:

main.go

```
// ...
{
	RealName: "Tony Stark",
	HeroName: "Iron Man",
	Planet:   "Earth",
},
// ...
```

When we launch the debugger again with `F5`, the **Debug View** opens and the execution stops at the breakpoint. We can see the JSON is not displayed in the **Debug Console**.

![Debug View, no JSON displayed](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/12.png)

## [Step 4 — Performing Further Debugging Tests](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#step-4-performing-further-debugging-tests)

Let’s add a new function to our file, enabling addition arithmetic:

main.go

```
func add(a, b int) int{
	return a+b
}
```

Create a test file `main_test.go` in the same directory, with the following contents:

main_test.go

```
package main

import "testing"

func Test_add(t *testing.T) {
	a, b, c := 1, 2, 3

	res := add(a, b)

	if res != c {
		t.Fail()
	}
}
```

The code just adds two numbers, and the test just calls the function.

If you have VSCode-Go plugin installed however, you’ll see additional options at the top of the test function - **run test** and **debug test**:

!["run test" and "debug test" options](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/13.png)

You can click on **run test** to run the test and see the results in the **Output** window.

To debug the test however, maybe because we can’t figure out something, all we need to do is add a breakpoint, like we did before, and click on **debug test**.

Add a breakpoint on **Line 10**: `if res != c`. Then, click on **debug test**.

![Adding breakpoint on Line 10](https://assets.digitalocean.com/articles/debugging-go-code-with-visual-studio-code/14.png)

The **Debug View** is opened again and we can use the **Debug Tool** to step over and inspect the state on the variables section on the left.

## [Conclusion](https://www.digitalocean.com/community/tutorials/debugging-go-code-with-visual-studio-code#conclusion)

Debugging is a critical part of developing software, and with tools such as Visual Studio Code, our lives can be made much easier.

We’ve added the debugger to an example project to explain the concepts, but feel free to add the debugger to any of your existing projects and play around with it. You’ll end up reducing your `fmt.Println` statements used to log to see the value or state of code at a give point during its execution.

[![Creative Commons](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fcreativecommons.c0a877f1.png&width=384)](https://creativecommons.org/licenses/by-nc-sa/4.0/)This work is licensed under a Creative Commons Attribution-NonCommercial- ShareAlike 4.0 International License.