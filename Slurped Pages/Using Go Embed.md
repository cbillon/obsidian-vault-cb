---
link: https://www.bytesizego.com/blog/go-embed
excerpt: The go:embed feature simplifies the inclusion of static assets in your Go applications. By embedding files and directories at compile time, you can create more portable and self-contained binaries.
slurped: 2025-11-05T09:23
title: Using Go Embed
tags:
  - go
  - embed
---

The `go:embed` feature simplifies the inclusion of static assets in your Go applications. By embedding files and directories at compile time, you can create more portable and self-contained binaries.

With the release of [Go 1.16](https://go.dev/), `go:embed` was introduced. This new code comment (we call it a directive) allows you to embed static files and folders into your Go binaries. If you’re building web servers that serve static content or building command-line tools that require configuration files, `go:embed` simplifies the process by eliminating the need for external file handling. It also means that your binary contains everything you need to ship your code. Powerful!

## What is go:embed?

The `go:embed` directive tells the Go compiler to include files and folders into the compiled binary at build time. This means your application can access these resources directly from memory without needing to read from the disk at runtime.

To use `go:embed`, you’ll need Go version 1.16 or later. Here’s a simple example of how you could embed a single file:

Firstly, make a txt file called `message.txt`. It can contain whatever you want - something like:

```
hello from bytesizego!
```

and write the following code in `main.go`

```
package main

import (
	_ "embed"
	"fmt"
)

//go:embed message.txt
var message string

func main() {
	fmt.Println(message)
}
```

In this very small program we are:

- Importing the blank identifier `_` for the embed package to enable the `go:embed` directive.
- Using the `//go:embed message.txt` directive specifies to specify the file to embed.
- Adding a `message` variable to hold the content of `message.txt` as a string.

As you’d expect, this program outputs:

```
hello from bytesizego!
```

## Embedding Multiple Files

Let’s make it a little more complicated:

```
package main

import (
	_ "embed"
	"fmt"
)

//go:embed messages/*.txt
var messages embed.FS

func main() {
	files, _ := messages.ReadDir("messages")
	for _, file := range files {
		data, _ := messages.ReadFile("messages/" + file.Name())
		fmt.Printf("File: %snContent: %snn", file.Name(), data)
	}
}
```

Here:

- We use `embed.FS` to embed multiple files.
- The `ReadDir` and `ReadFile` methods allow us to interact with the embedded files.

## Embedding Directories

You can also embed entire folders:

```
package main

import (
	"embed"
	"fmt"
)

//go:embed static
var staticFiles embed.FS

func main() {
	data, _ := staticFiles.ReadFile("static/index.html")
	fmt.Println(string(data))
}
```

Tip: When embedding directories, the path specified in `ReadFile` is relative to the embedded root.

## Using go embed to Create a Web App

Super simple!

```
package main

import (
	"embed"
	"net/http"
)

//go:embed static/*
var staticFiles embed.FS

func main() {
	http.Handle("/", http.FileServer(http.FS(staticFiles)))
	http.ListenAndServe(":8080", nil)
}
```

Run the program, and now you can access your embedded static files via [http://localhost:8080](http://localhost:8080/).

## Using go embed to Embed Files into a Struct

Maybe you want to embed stuff to use later in your program. Putting it on a struct is probably what you want. This could be useful for configuration.

```
package main

import (
	"embed"
	"fmt"
)

//go:embed templates/home.html
var homeTemplate string

//go:embed templates/about.html
var aboutTemplate string

type Templates struct {
	Home  string
	About string
}

func main() {
	t := Templates{
		Home:  homeTemplate,
		About: aboutTemplate,
	}
	fmt.Println("Home Template:", t.Home)
	fmt.Println("About Template:", t.About)
}
```

## Using go Embed to Embed Images and PDFs

If you’re working with non-text files (like images or PDFs), you should use a byte slice:

```
package main

import (
	_ "embed"
	"fmt"
)

//go:embed image.webp
var imageData []byte

func main() {
	fmt.Printf("Image size: %d bytesn", len(imageData))
}
```

Be careful with this! Here’s some limitations:

- **File Size**: Embedding large files can significantly increase your binary size.
- **File Changes**: Changes to the embedded files require recompilation.

## Wrapping up

`go embed` is amazing! I see it rarely used, but I use it often now, especially when creating web apps.

If you found this blog useful, check out [our other blogs](https://www.bytesizego.com/blog) or [join our mailing list](https://www.bytesizego.com/golang-jobs) which will send you a list of 60+ companies hiring Go engineers remotely right now.