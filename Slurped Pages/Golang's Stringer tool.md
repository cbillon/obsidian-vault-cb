---
link: https://last9.io/blog/golang-stringer-tool/
byline: Arjun Mahishi
site: Last9
date: 2022-12-12T08:03
excerpt: Learn about how to use, extend and auto-generate Stringer tool of Golang
twitter: https://twitter.com/@last9io
slurped: 2026-03-27T19:25
title: Golang's Stringer tool
---

Golang's `fmt` the package exposes a ubiquitous interface called `Stringer`, that  
is used by a lot of packages. This interface has a single method called  
`String()`. Any package that needs the string representation of an identifier usually looks for implementations of this interface.

```
type Stringer interface {
  String() string
}
```

So, while printing any identifier, you can control what actually gets printed by defining a `String` method for it.

Let's say, you have an integer type called `StatusCode` and a few constants of that type. Now, if you print the constants with `Println` (of either `fmt` or `log`), you will see the status codes (200,  500, etc) getting printed.

```
type StatusCode int

const (
  statusOK                StatusCode = 200
  statusInternalServerErr StatusCode = 500
)

func main() {
  fmt.Println(statusOK)
  fmt.Println(statusInternalServerErr)

  // Output:
  // 200
  // 500
}
```

But if you want to see the string representation of these status codes, you can implement the stringer interface by simply defining a `String()` for `StatusCode`. Now, `Println` will print whatever the `String()` function returns

```
type StatusCode int

func (c StatusCode) String() string {
  switch c {
  case 200:
    return "Status OK"
  case 500:
    return "Internal server error"
  default:
    return "Unknown status"
  }
}

const (
  statusOK                StatusCode = 200
  statusInternalServerErr StatusCode = 500
)

func main() {
  fmt.Println(statusOK)
  fmt.Println(statusInternalServerErr)

  // Output:
  // Status OK
  // Internal server error
}
```

I will not blame you if you are wondering why you should care about this at all. Among a lot more, here are two main use cases where this has been useful to me:

1. **Telemetry/Logging** – identifiers in your code, especially numbers and structs, can be hard to represent in log statements and metric labels. Numbers can be used as it is, but they may be hard to understand for people who have less/no context on the code. `String()` method can help you define a sane representation of these identifiers in logs/metrics labels  
2. **User side** – On similar lines, code may be dealing with certain values that may be different from what the user sees or inputs. Stringer interface can help in this regard as well

Take a quick look at the above code snippet again. You may be able to see how hard it is to maintain such a switch case. Each constant you create will need a case. If you forget to add one, you may end up with some inconsistent behaviour.

So, to overcome this, Go's standard toolchain offers the [Stringer CLI](https://pkg.go.dev/golang.org/x/tools/cmd/stringer) tool as a solution.

Stringer is a CLI tool that lets you automate the creation of these `String()` methods. By default, the constant's name will be used as the string value. But it provides a couple of flags to control that.

💡

Stringer currently only has support for integer types

##### Here's how to install it

```
go install golang.org/x/tools/cmd/stringer@latest
```

Now, let's look at a few examples of how to run stringer and explore the customization that it offers. Assume the below setup for each example.

```
type Status int

const (
	statusOK                Status = 200
	statusBadRequest        Status = 400
	statusInternalServerErr Status = 500
)

func main() {
	fmt.Println(statusOK)
	fmt.Println(statusBadRequest)
	fmt.Println(statusInternalServerErr)
}
```

Paste the above code in a `main.go` file and run it as it is. It should give you this output:

```
200
400
500
```

Now, in the same directory as this file, run the following command:

```
$ stringer --type Status
```

That should generate a file called `status_string.go` the auto-generated `String()`method. Running the go program again should now give you an output like this:

```
statusOK
statusBadRequest
statusInternalServerErr
```

These specific values are printed because the value returned by the auto-generated `String()` is based on the identifier name by default. The flag `--type` is mandatory and accepts the name of the type for which the Stringer interface needs to be implemented.

💡

The `--type` flag can take multiple comma separated type names  
Example: `--type Status,Method,Error`

The identifiers were prefixed with _status_ to indicate that it is of the type `Status`. But in the resulting string value, we don't need to do that (as the type is not really relevant in the output). We can trim that out using the flag `--trimprefix`.

```
$ stringer --type Status --trimprefix status
```

Run the above command and run the go program again. Now, the output should look like this:

```
OK
BadRequest
InternalServerErr
```

If you want a custom value, there is the flag called `--linecomment`, which lets you specify the exact value you want them `String()` to return. Make the following change to the code:

```
const (
	statusOK                Status = 200
	statusBadRequest        Status = 400
	statusInternalServerErr Status = 500 // Something went wrong
)
```

```
$ stringer --type Status --trimprefix status --linecomment
```

After running the above command, the go program should give the following output:

```
OK
BadRequest
Something went wrong
```

You can probably see how this can be very extendable. All you have to do is, add one more entry to the `const` block. As long as you run the `stringer` command before compiling, all your constants will have a string value.

> But what if I have a lot of types, each with its own customisation, spread across multiple packages in multiple sub-directories?  Do I need to run `stringer` in each directory ?

## Enter – `go:generate`

Let's hit pause on the stringer for a second, and get into `go generate`. Feel free to skip this section if you're already familiar with it.

The main Go binary has a sub-command called `generate` which is used to trigger code generation tools (like stringer).

💡

Rob Pike has written an amazing blog post, on the Go blog, about code generation. You can check it out [here](https://go.dev/blog/generate)

Let's look at a quick example to see how `go generate` works. Open any `.go` file and add the following comment:

```
//go:generate echo "hello world"
```

Save it and run `go generate` in the directory that has this file. You will see that `hello world` gets printed.

So, by adding `//go:generate <shell command>` in your go files, you can automate a lot of things.

## Marrying them both together

_You can probably guess where this is going now..._

With the help of `go generate`, you can automate running the stringer command. Doing that will solve all the problems mentioned 2 sections ago.

- You will not have to run stringer for all the types
- You can run stringer once for all sub-packages by running this command:

```
$ go generate ./...
```

While declaring an int type as an enum,  just add this `go:generate` comment above it. Where you add this comment does not matter. But putting this comment right about the type declaration will help the reader understand that the `String()` methods will be auto-generated for this type.

**Example**

```
//go:generate stringer -type Status -trimprefix status
type Status int

const (
	statusOK                Status = 200
	statusBadRequest        Status = 400
	statusInternalServerErr Status = 500
)
```

💡

You don't have to check in the generated `_string.go` files in git. You can simply add an additional step in your CI, just before the build step, to run `go generate ./...`. And all the `String()` methods will automatically be generated.

## Things to remember

- If you have `go:generate` comments, don't forget to always run `go generate` before running `go build`. Because, without running `go generate`, the build command might not fail (If, the `String()` function is not directly used). But the behaviour that could end up being inconsistent
- `go generate` does NOT guarantee the order of execution. So, if you care about the order in which the string methods are generated, make sure to write your own orchestration script

## Closing

`stringer` is a very nifty tool to have in your developer toolbelt. But due to its lack of popularity (and documentation), this ends up confusing people when they encounter it. But it's a very interesting and good-to-know tool. It can make your code cleaner and easier to maintain.

It's a part of the `gofmt`, `goimports`, `gopls`, `godoc`, etc gang.

Make sure you check out the plethora of out-of-the-box [standard tools](https://pkg.go.dev/golang.org/x/tools#section-readme) that Go has to offer, to help you write go code. Even if you don't think you need them right now, they may help you figure out problems you didn't know you had 😛