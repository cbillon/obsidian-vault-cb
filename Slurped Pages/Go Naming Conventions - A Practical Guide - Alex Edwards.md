---
link: https://www.alexedwards.net/blog/go-naming-conventions
byline: Alex Edwards
excerpt: Choosing the right names in your codebase is an important (and
  sometimes difficult!) part of programming in Go. It's a small thing that makes
  a big difference — good names make your code clearer, more predictable, and
  easier to navigate; bad names do the opposite.
twitter: https://twitter.com/@ajmedwards
slurped: 2026-03-27T19:34
title: "Go Naming Conventions: A Practical Guide - Alex Edwards"
---

Choosing the right names in your codebase is an important (and sometimes difficult!) part of programming in Go. It's a small thing that makes a big difference — good names make your code clearer, more predictable, and easier to navigate; bad names do the opposite.

Go has fairly strong conventions — and a few hard rules — for naming things. In this post we're going to explain these rules and conventions, provide some practical tips, and demonstrate some examples of good and bad names in Go. If you're new to the language, all this information might feel like a lot to take in, but it'll quickly become second nature with a bit of practice 😊

## Identifiers

Let's start with the hard rules for _identifiers_. By identifiers, I mean the names that you use for the variables, constants, types, functions, parameters, struct fields, methods and receivers in your code.

- Identifiers can contain unicode letters, digits, and underscores only.
- Identifiers cannot begin with a digit.
- You cannot use any of the following Go keywords as identifiers:

```
  break   default      func         interface    select 
  case    defer    go           map          struct       chan         else        goto   package   switch       const        fallthrough  if           range       type continue   or           import       return       var
```

So long as you stick to those three rules, any identifier name is technically valid and your code will compile all OK. But there are a bunch of other guidelines that it's good practice to follow:

- You should use `camelCase` for unexported identifiers, or `PascalCase` for exported identifiers. Don't use alternative casing variants like `snake_case`, `Pascal_Snake_Case`, `SCREAMING_SNAKE_CASE` or `ALLUPPERCASE`.
    
- Words that are acronyms or initialisms (like API, URL or HTTP) should use a consistent case _within_ the identifier. So, for example, `apiKey` or `APIKey` are conventional, but `ApiKey` is not. This rule also applies to `ID` when it is used as shorthand for the words "identity" or "identifier" — so that means write `userID` rather than `userId`.
    
- Although all unicode letters are allowed, using non-ASCII letters can often make your code harder to read and more awkward to write, and it's rare to see them used. Unless you have a really appropriate use-case, you should stick to using ASCII letters in identifiers. For example, use `pi` instead of `π`, use `beta` instead of `β`, use `naiveBayes` instead of `naïveBayes`.
    
- To prevent confusion for readers and potential bugs, avoid choosing identifiers that clash with Go's [builtin types](https://pkg.go.dev/builtin). So, for example, don't create variables with names like `int`, `bool` or `any`. Similarly, avoid creating functions with names that clash with Go's builtin functions. So, for example, don't create functions with names like `min`, `max`, `len` or `clear`.
    
- Generally, avoid including the _type_ in identifiers — for example, don't use names like `fullNameString`, `scoreInt` or `float64Amount`. The main exception to this is when you have to convert a variable to a different type, and you want to distinguish between the original variable and the one containing the converted value. In this situation, including the type in the identifier is a common and acceptable way to distinguish between the two. For example, code like this is OK:
    
    `userID := 42 userIDStr := strconv.Itoa(userID)`
    
- Where possible, try to avoid choosing identifiers that clash with the standard library package names. This is a 'softer' convention than the others because the standard library steals a lot of good identifier names — such as `json`, `js`, `mail`, `user`, `csv`, `path`, `filepath`, `log`, `regexp`, `time` and `url` — and sometimes it can be hard to come up with decent alternatives. However, you definitely _should_ avoid creating identifiers that clash with the package names that your code is _actually importing and using_. So, for example, if you are writing code that imports the `url` and `net/mail` packages, then don't use the words `url` and `mail` as identifiers in that code.
    

Here are a few examples of good and bad identifier names:

|Bad|Reason|Better|
|---|---|---|
|`order.total := 99.99`  <br>`func load-user()`|Punctuation not allowed|`orderTotal := 99.99`  <br>`func loadUser()`|
|`const 3rdParty = "x"`  <br>`func 2FactorAuth()`|Cannot start with a digit|`const thirdParty = "x"`  <br>`func twoFactorAuth()`|
|`max_value := 10`  <br>`func Fetch_user()`|Non-standard casing|`maxValue := 10`  <br>`func FetchUser()`|
|`type HttpClient struct{}`  <br>`func parseXml()`|Inconsistent acronym casing|`type HTTPClient struct{}`  <br>`func parseXML()`|
|`func GetSessionId()`  <br>`type OrderId string`|ID should be all caps|`func GetSessionID()`  <br>`type OrderID string`|
|`résuméCount := 2`  <br>`const Σ = 100`|Non-ASCII letters|`resumeCount := 2`  <br>`const sum = 100`|
|`func clear()`  <br>`int := cache.Internal()`|Clashes with builtin types or functions|`func clearQueue()`  <br>`data := cache.Internal()`|
|`intCount := 42`  <br>`resultSlice := []int{}`|Type included in name|`count := 42`  <br>`results := []int{}`|
|`type json struct{}`  <br>`var log = newLogger()`|Clashes with stdlib package names|`type payload struct{}`  <br>`var logger = newLogger()`|

## Exported and unexported identifiers

Identifiers in Go are case-sensitive. For example, the identifiers `apiKey`, `apikey` and `APIKey` are all different.

As you [probably already know](https://www.alexedwards.net/blog/an-introduction-to-packages-imports-and-modules#exported-vs-unexported), when an identifier starts with a capital letter it is exported — that is, it's visible to code outside of the package it's declared in. This means that the casing of the first letter is _significant_. It impacts the behavior of your codebase. In turn, this means that you shouldn't start identifiers with a capital letter just because they look nice — you should only start them with a capital letter if you want them to be exported and accessible to code outside the package they are declared in.

As a tip, try to write packages using unexported identifiers by default. Only export them when you actually have a need to.

Typically, the less you export, the easier it is to refactor code within a package without affecting other parts of your codebase. There's a nice quote from [The Pragmatic Programmer](https://en.wikipedia.org/wiki/The_Pragmatic_Programmer), which I'll adapt slightly for the Go nomenclature:

> Write shy code - packages that don't reveal anything unnecessary to other packages and don't rely on other packages' implementations.

As a second tip, it's very rare for a `main` package to be imported by anything, so the identifiers in it should normally all be unexported and start with a lowercase letter. The most frequent exception to this is when you need to export a struct field so that it's visible to packages that use reflection to work, like `encoding/json`, `encoding/gob` or `github.com/jmoiron/sqlx`.

## Identifier length and descriptiveness

In general, the further away that an identifier is used from where it is declared, the more descriptive the name should be.

If you have an identifier which is narrow in scope and only used close to where it is declared, it's generally OK to use a short and not-very-descriptive name. For example, if you're naming something that is only used in a small `for` loop, `range` block, or very short function, then using short or even single letter names is very common in Go.

But it you're naming something that has a larger scope, or is used far away from where it is declared, you should use a name that clearly describes what the thing represents.

Here is a nice example that Dave Cheney gave as part of his [Practical Go](https://dave.cheney.net/practical-go/presentations/qcon-china.html#_identifier_length) presentation:
```
type Person struct {
     Name string
     Age  int
}

func AverageAge(people []Person) int {
    if len(people) == 0 {
         return 0
    }
    
    var count, sum int
    for _, p := range people {
        sum += p.Age
        count += 1
    }
    
    return sum / count
} 

```


In this code, within the short `range` block we use the identifier `p` to represent a value in the `people` slice — the `range` block is so small and tight that using a single letter name is clear enough.

In contrast, the `count` and `sum` variables are declared, then used inside the range, then again in the return statement. Giving them more descriptive names makes it immediately clearer what the code is doing and what they represent, compared to single letter names like `c` and `s`. But these variables are only ever used inside the `AverageAge` function, so giving them even-more-descriptive names like `peopleCount` and `agesSum` would be unnecessarily verbose.

It's not an exact science, but when writing Go code you are encouraged to use the _right length_ identifier — sometimes that might be long and descriptive, sometimes it might be short and terse.

## Naming packages

The hard rules for package names are the same as for identifiers: they can contain unicode letters, numbers and underscores, must not begin with a number, and must not be a Go keyword. But in practice, the conventions for naming a package are much tighter. Conventionally:

- Package names should contain lower case ASCII letters and numbers only.
    
- Because package names will need to be typed out a lot when writing code, the name should ideally be short, easy to type, and reflect the contents of the package. Often simple one-word nouns (like `orders`, `customer` and `slug`) work well.
    
- If you want to use more than one word in the package name, you should concatenate the words all in lowercase with no separator. So, for example, `ordermanager` is a conventional package name — `orderManager` or `order_manager` are not.
    
- If a package name feels too long, it can be OK to use abbreviations in the name. You can see this in some of the standard library package names, like `expvar` (instead of `exportedvariables`) and `strconv` (instead of `stringconversion`).
    
- To prevent conflicts and confusion, try to avoid using the same name as commonly-used standard library packages.
    
- Package names with the prefix `.` or `_` are 'invisible' to Go and completely ignored when you run `go build`, `go run`, `go test` etc. So don't start your package name with these characters, unless you specifically want them to be ignored.
    
- Directories with the names `vendor`, `testdata` and `internal` have a special meaning in Go, so to avoid any confusion or bugs, don't use these words as package names.
    
- Avoid using 'catch all' package names like `common`, `util`, `helpers`, `types` or `interfaces`, which don't _really_ give any clue to what the package contains. For example, does a package called `helpers` contain validation helpers, formatting helpers, SQL helpers? A mix of all the above? You can't guess from just the name alone. As well as not being clear, these kind of 'catch all' names provide little natural boundary or scope, which can lead to the package becoming a dumping ground for lots of different things. In turn the package may become imported and used throughout your codebase — which increases the risk of import cycles and means that changes to the package potentially affect the whole codebase, rather than just a specific part of it. In other words, catch all package names encourage creating packages which have a large 'blast radius'. If you find yourself wanting to create a `utils` or `helpers` package, ask yourself if you can break up the contents into smaller packages with a specific focus and clearer names instead.
    

|Bad|Reason|Better|
|---|---|---|
|`package 3rdparty`  <br>`package 2fa`|Cannot start with a digit|`package thirdparty`  <br>`package twofa`|
|`package OrderManager`  <br>`package order_manager`|Non-standard casing / separators|`package ordermanager`|
|`package o`  <br>`package stuff`|Too vague and not descriptive|`package orders`  <br>`package slug`|
|`package ordermanagementsystem`|Too long / hard to type|`package orders`  <br>`package ordermgr`|
|`package url`  <br>`package mail`|Clashes with stdlib package names|`package links`  <br>`package mailer`|
|`package _cache`  <br>`package .hidden`|Ignored by Go tooling|`package cache`  <br>`package hidden`|
|`package internal`  <br>`package vendor`  <br>`package testdata`|Special directory names in Go|`package internalauth`  <br>`package supplier`|
|`package utils`  <br>`package helpers`|Catch-all names with unclear scope|`package validation`  <br>`package formatting`|

## Naming files

In an ideal world, a `.go` filename should summarize what the file contains, be one word long, and all in lowercase. Some examples of good filenames from the standard library `net/http` package are `cookie.go`, `server.go` and `status.go`.

If you can't think of a good one-word name, and want to use two or more words, there is no clear convention for how those words should be separated. Even in the Go standard library itself there isn't consistency. Sometimes underscores are used to separate the words in filenames (like `routing_index.go` and `routing_tree.go`), and other times they are concatenated with nothing between them (like `batchcursor.go`, `textreader.go` and `reverseproxy.go`).

Because there isn't a strong convention around this, I recommend just picking one of these two approaches and sticking to it consistently within a codebase. Personally, I think it's better to concatenate words with nothing between them (like `routingindex.go`), and reserve the underscore character for only when you want to use a _special filename suffix_.

Talking of which, there are some filename prefixes and suffixes that have a special meaning in Go. You should avoid using these in your filenames unless you want to trigger the special behavior. Specifically:

- Like packages, filenames with the prefix `.` or `_` are 'invisible' to the Go tooling and completely ignored when you run `go build`, `go run`, `go test` etc.
    
- Files with the suffix `_test.go` are only run by the `go test` tool. They are ignored when using `go run` or `go build`.
    
- Files with any of the following suffixes will only be included when compiling for that specific operating system: `_aix.go`, `_android.go`, `_darwin.go`, `_dragonfly.go`, `_freebsd.go`, `_illumos.go`, `_ios.go`, `_js.go`, `_linux.go`, `_netbsd.go`, `_openbsd.go`, `_plan9.go`, `_solaris.go`, `_wasip1.go`, `_windows.go`.
    
- Similarly, files with any of the following suffixes will only be included when compiling for that specific architecture: `_386.go`, `_amd64.go`, `_arm.go`, `_arm64.go`, `_loong64.go`, `_mips.go`, `_mips64.go`, `_mips64le.go`, `_mipsle.go`, `_ppc64.go`, `_ppc64le.go`, `_riscv64.go`, `_s390x.go`, `_wasm.go`.
    

## Avoiding chatter

When you are naming exported functions, try to avoid repeating the name of the package they are declared in.

For example, if you have a package called `customer`, then function names like `NewCustomer()` or `CustomerOrders()` would be 'chattery' and unnecessarily repeat the word 'customer' when you call them from outside the package — like `customer.NewCustomer()` and `customer.CustomerOrders()`. Calling the functions `New()` and `Orders()` is sufficient and reads better at the call site — like `customer.New()` and `customer.Orders()`.

The same advice also applies to exported types. For example, if you want to represent an address or phone number in a `customer` package, it's sufficient and less chattery to name the types `Address` and `PhoneNumber` rather than `CustomerAddress` and `CustomerPhoneNumber`.

|Bad|Reason|Better|
|---|---|---|
|`customer.NewCustomer()`  <br>`customer.CustomerOrders()`|Chattery function call|`customer.New()`  <br>`customer.Orders()`|
|`customer.CustomerAddress`  <br>`customer.CustomerPhoneNumber`|Chattery type reference|`customer.Address`  <br>`customer.PhoneNumber`|

Similar to function and type names, method names should ideally not 'chatter' too much when calling them. For example, if you are writing methods on a `Token` type, for example, it's probably OK to call a method `Validate()` rather than `ValidateToken()`, or `IsExpired()` rather than `IsTokenExpired()`.

## Method receivers

When you are creating methods, it is conventional for the method _receiver_ to have a short name, normally between 1 and 3 characters long and often an abbreviation of the type that the method is implemented on. For example, if you are implementing a method on a `Customer` type, an idiomatic receiver name would be something like `c` or `cus`. Or if you were implementing a method on a `HighScore` type, a good receiver name would be `hs`.

The Go code review comments [advise against](https://go.dev/wiki/CodeReviewComments#receiver-names) using generic names like `this`, `self` or `me` for the receiver.

Also, you should be consistent with the receiver name. All methods on the _same type_ should use the _same receiver name_ — don't use `c` for one method and `cus` for another.

```
type Order struct {
     Items int
}
// Good: uses a short receiver  
func (o *Order) Validate() bool {
     return o.Items > 0 
}
// Bad: uses a longer receiver name 
func (order *Order) Validate() bool {
     return order.Items > 0
}
// Bad: uses a generic receiver name 
func (self *Order) Validate() bool {
     return self.Items > 0
}
```

## Getter and setter methods on structs

Typically, it is not necessary to create 'getter' and 'setter' methods on struct types in Go. Instead, you just access the struct field directly to read or change the data. The major exception to this is when you have a struct with an unexported field, but want to provide a way to get or set the field value from outside the package. To do this, you need to create exported 'getter' and 'setter' methods that read and write to the unexported field.

When doing this, it is conventional to prefix the setter method name with `Set`, but _not_ prefix the getter method name with `Get`. Like so:

`type Customer struct {     address string }  func (c *Customer) Address() string {     return c.address }  func (c *Customer) SetAddress(addr string)  {     c.address = addr }`

## Interfaces

By convention, interfaces that only contain one method should be named by the method name plus an '-er' suffix or similar. For example:
```
type Speaker interface {
     Speak() string
}

type Authorizer interface {
     Authorize(ctx context.Context, action string) error 
}

type Authenticator interface {
     Authenticate(ctx context.Context) (User, error) 
}
```


The Go standard library has quite a few examples of interfaces that follow this convention, such as [`io.Reader`](https://pkg.go.dev/io#Reader), [`io.Writer`](https://pkg.go.dev/io#Writer) and [`fmt.Stringer`](https://pkg.go.dev/fmt#Stringer) (and also some, like [`http.Handler`](https://pkg.go.dev/net/http#Handler), which don't).

Also note that the guidance to avoid including the type in the name still applies to interfaces. Don't give your interfaces names like `UserInterface` or `OrderInterface` unless you really can't think of a decent alternative.

## Breaking from conventions

There are rare occasions when breaking a convention can actually make your code clearer — and in my view, _it can be OK to do that_... especially if it's in a private codebase worked on by a small team.

For example, a couple of years ago I was working on a Go program that synchronizes data between some other external systems. In this project, I ended up breaking some of the Go naming conventions around casing and separators — instead using exactly the same identifiers that the external systems used. This actually made the intent of the program clearer, and more immediately obvious what was being synchronized with what.

But the vast majority of the time, you should endeavour to follow the naming rules and conventions we've discussed in this post. They exist for good reasons: they make your code more predictable and consistent, easier for other Gophers to quickly understand, and reduce the risk of certain bugs.