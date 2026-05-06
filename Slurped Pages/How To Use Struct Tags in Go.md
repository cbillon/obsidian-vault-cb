---
link: https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go
byline: Gopher Guides
site: DigitalOcean
date: 2019-10-31T23:55
excerpt: Learn how to use struct tags in Go for JSON, XML, database mapping, and validation. Includes syntax rules, reflection examples, and custom tag patterns.
slurped: 2026-05-01T18:31
title: How To Use Struct Tags in Go
tags:
  - go
  - structs
  - tag
---

## [Introduction](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#introduction)

[**Structures**, or **structs**](https://www.digitalocean.com/community/tutorials/defining-structs-in-go), are used to collect multiple pieces of information together in one unit. These [structs](https://www.digitalocean.com/community/tutorials/defining-structs-in-go) are used to describe higher-level concepts, such as an Address composed of a Street, City, State, and PostalCode.

When you read or write this information to external systems—such as databases, APIs, or configuration files—you can use struct tags to control how this data is interpreted and mapped to struct fields. Struct tags are small pieces of metadata attached to fields of a struct that provide instructions to other Go code that works with the struct.

Struct tags themselves do not affect program behavior directly. Instead, they are read at runtime—typically using reflection—by standard library packages or third-party libraries to modify how data is encoded, decoded, validated, or stored.

In this article, you’ll learn how to use struct tags in Go to control how your data is encoded and mapped. You’ll explore how the `encoding/json` package uses struct tags to customize JSON output, understand the difference between `Get` and `Lookup` methods for reading tags, and discover how to combine multiple tags on a single field. You’ll also learn how to create your own custom struct tags and examine common tag conventions used throughout the [Go ecosystem](https://go.dev/).

**Key Takeaways:**

- Struct tags are metadata annotations written in backticks after field types using `key:"value"` syntax, and have no effect on code execution until read by other packages at runtime.
- The `encoding/json` package uses `json` struct tags to control field naming, with special options like `,omitempty` to skip zero values and `"-"` to exclude fields entirely from JSON output.
- Use `reflect.StructTag.Get()` when you only need a tag’s value, but use `Lookup()` when you need to distinguish between an absent tag and one explicitly set to an empty string.
- Multiple struct tags can coexist on a single field by separating them with spaces (not commas), allowing one struct to work across JSON APIs, databases, form parsing, and validation libraries simultaneously.
- You can create custom struct tags by defining your own tag keys and using the `reflect` package to read them, enabling declarative configuration patterns like environment variable mapping.
- Common struct tag conventions exist across the Go ecosystem, including `json`, `xml`, `yaml`, `db`, `validate`, `form`, and `env`, though the compiler doesn’t enforce or validate semantic meaning.
- When implementing custom tag logic, remember that reflection only works with exported fields, requires pointer receivers to modify values, and needs type checking to prevent runtime panics.
- The `go vet` tool catches basic struct tag syntax errors like missing quotes, but won’t detect typos in tag keys, invalid option values, or logical errors in tag usage.

## [What Does a Struct Tag Look Like?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#what-does-a-struct-tag-look-like)

Go struct tags are annotations that appear after the type in a Go struct field declaration. Each tag is composed of key-value pairs, where a short string key is associated with a corresponding value.

A struct tag is written as a string literal enclosed in backticks (`` ` ``) immediately following the field type:

```
type User struct {
    Name string `example:"name"`
}
```

In this example, `example` is the tag key and `"name"` is its value.

Struct tags are stored as raw string metadata and have no effect on program behavior by themselves. Other Go code, typically using the `reflect` package, can examine struct fields at runtime and extract the values associated with specific tag keys.

Try this example to see what struct tags look like, and to confirm that without additional code that reads them, they have no effect:

```
package main

import "fmt"

type User struct {
    Name string `example:"name"`
}

func (u *User) String() string {
    return fmt.Sprintf("Hi! My name is %s", u.Name)
}

func main() {
    u := &User{
        Name: "Sammy",
    }

    fmt.Println(u)
}
```

This will output:

```
OutputHi! My name is Sammy
```

This example defines a `User` type with a `Name` field. The `Name` field has been given a struct tag of `example:"name"`. We would refer to this tag by its key, `example`, which has the value `"name"`.

On the `User` type, we also define the `String()` method required by the [`fmt.Stringer` interface](https://www.digitalocean.com/community/tutorials/how-to-use-interfaces-in-go). This method is automatically called when the value is passed to `fmt.Println`, allowing us to produce a formatted string representation of the struct.

Within the body of `main`, we create a new instance of our `User` type and pass it to `fmt.Println`. Even though the struct includes a tag, it has no effect on the output because no code is inspecting or using that tag.

To make struct tags useful, other Go code must examine them at runtime. The standard library includes several packages that rely on struct tags as part of their functionality. One of the most commonly used is the `encoding/json` package.

## [Encoding JSON](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#encoding-json)

[JavaScript Object Notation (JSON)](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go) is a textual format for encoding collections of data organized under different string keys. It’s commonly used to communicate data between different programs as the format is simple enough that libraries exist to decode it in many different languages. The following is an example of JSON:

```
{
  "language": "Go",
  "mascot": "Gopher"
}
```

This JSON object contains two keys, `language` and `mascot`. Following these keys are the associated values. Here the `language` key has a value of `Go` and `mascot` is assigned the value `Gopher`.

The JSON encoder in the standard library makes use of struct tags as annotations indicating to the encoder how you would like to name your fields in the JSON output. These JSON encoding and decoding mechanisms can be found in the `encoding/json` [package](https://pkg.go.dev/encoding/json).

Try this example to see how JSON is encoded without struct tags:

```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type User struct {
    Name          string
    Password      string
    PreferredFish []string
    CreatedAt     time.Time
}

func main() {
    u := &User{
        Name:      "Sammy the Shark",
        Password:  "fisharegreat",
        CreatedAt: time.Now(),
    }

    out, err := json.MarshalIndent(u, "", "  ")
    if err != nil {
        log.Println(err)
        os.Exit(1)
    }

    fmt.Println(string(out))
}
```

This will print output similar to the following:

```
Output{
  "Name": "Sammy the Shark",
  "Password": "fisharegreat",
  "CreatedAt": "2026-03-24T04:44:07.493850641-04:00"
}
```

We defined a struct describing a user with fields including their name, password, and the time the user was created. Within the `main` function, we create an instance of this user by supplying values for all fields except `PreferredFish` (Sammy likes all fish). We then passed the instance of `User` to the `json.MarshalIndent` function. This is used so we can more easily see the JSON output without using an external formatting tool. This call could be replaced with `json.Marshal(u)` to print JSON without any additional whitespace. The two additional arguments to `json.MarshalIndent` control the prefix to the output (which we have omitted with the empty string), and the characters to use for indenting, which here are two space characters. Any errors produced from `json.MarshalIndent` are logged and the program terminates using `os.Exit(1)`. Finally, we cast the `[]byte` returned from `json.MarshalIndent` to a `string` and passed the resulting string to `fmt.Println` for printing on the terminal.

The fields of the struct appear exactly as named. This is not the typical JSON style that you may expect, though, which uses camel casing for names of fields. You’ll change the names of the field to follow camel case style in this next example. As you’ll see when you run this example, this won’t work because the desired field names conflict with Go’s rules about exported field names.

```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type User struct {
    name          string
    password      string
    preferredFish []string
    createdAt     time.Time
}

func main() {
    u := &User{
        name:      "Sammy the Shark",
        password:  "fisharegreat",
        createdAt: time.Now(),
    }

    out, err := json.MarshalIndent(u, "", "  ")
    if err != nil {
        log.Println(err)
        os.Exit(1)
    }

    fmt.Println(string(out))
}
```

This will present the following output:

```
Output{}
```

In Go, fields must be exported (start with an uppercase letter) to be accessible to the `encoding/json` package. While JSON itself commonly uses camelCase naming, Go requires exported fields for encoding and decoding.

While JSON doesn’t care how you name your fields, Go does, as it indicates the visibility of the field outside of the package. Since the `encoding/json` package is a separate package from the `main` package we’re using, we must uppercase the first character in order to make it visible to `encoding/json`. It would seem that we’re at an impasse. We need some way to convey to the JSON encoder what we would like this field to be named.

### [Using Struct Tags to Control Encoding](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#using-struct-tags-to-control-encoding)

You can modify the previous example to have exported fields that are properly encoded with camel-cased field names by annotating each field with a struct tag. The struct tag that `encoding/json` recognizes has a key of `json` and a value that controls the output. By placing the camel-cased version of the field names as the value to the `json` key, the encoder will use that name instead. This example fixes the previous two attempts:

```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type User struct {
    Name          string    `json:"name"`
    Password      string    `json:"password"`
    PreferredFish []string  `json:"preferredFish"`
    CreatedAt     time.Time `json:"createdAt"`
}

func main() {
    u := &User{
        Name:      "Sammy the Shark",
        Password:  "fisharegreat",
        CreatedAt: time.Now(),
    }

    out, err := json.MarshalIndent(u, "", "  ")
    if err != nil {
        log.Println(err)
        os.Exit(1)
    }

    fmt.Println(string(out))
}
```

This will output:

```
Output{
  "name": "Sammy the Shark",
  "password": "fisharegreat",
  "preferredFish": null,
  "createdAt": "2026-03-24T04:48:32.45984986-04:00"
}
```

We’ve changed the struct fields back to be visible to other packages by capitalizing the first letters of their names. However, this time we’ve added struct tags in the form of `json:"name"`, where `"name"` was the name we wanted `json.MarshalIndent` to use when printing our struct as JSON.

The `CreatedAt` field is of type `time.Time`, which is encoded in RFC 3339 format by default.

We’ve now successfully formatted our JSON correctly. Notice, however, that the fields for some values were printed even though we did not set those values. The JSON encoder can eliminate these fields as well, if you like.

### [Removing Empty JSON Fields](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#removing-empty-json-fields)

It is common to omit fields that are unset in JSON. Since all types in Go have a “zero value,” some default value that they are set to, the `encoding/json` package needs additional information to be able to tell that some field should be considered unset when it assumes this zero value. Within the value part of any `json` struct tag, you can suffix the desired name of your field with `,omitempty` to tell the JSON encoder to suppress the output of this field when the field is set to the zero value. The following example fixes the previous examples to no longer output empty fields:

```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type User struct {
    Name          string    `json:"name"`
    Password      string    `json:"password"`
    PreferredFish []string  `json:"preferredFish,omitempty"`
    CreatedAt     time.Time `json:"createdAt"`
}

func main() {
    u := &User{
        Name:      "Sammy the Shark",
        Password:  "fisharegreat",
        CreatedAt: time.Now(),
    }

    out, err := json.MarshalIndent(u, "", "  ")
    if err != nil {
        log.Println(err)
        os.Exit(1)
    }

    fmt.Println(string(out))
}
```

This example will output:

```
Output{
  "name": "Sammy the Shark",
  "password": "fisharegreat",
  "createdAt": "2026-03-24T04:50:43.617137191-04:00"
}
```

We’ve modified the previous examples so that the `PreferredFish` field now has the struct tag `json:"preferredFish,omitempty"`. The presence of the `,omitempty` option causes the JSON encoder to skip that field, since we decided to leave it unset.

The definition of an “empty” value includes `""`, `0`, `false`, `nil`, and empty slices or maps. However, some types such as `time.Time` may still be encoded even when set to their [zero value](https://www.digitalocean.com/community/tutorials/how-to-use-variables-and-constants-in-go#zero-values), which can lead to unexpected results when using `omitempty`.

This output is looking much better, but we’re still printing out the user’s password. The `encoding/json` package provides another way for us to ignore private fields entirely.

### [Ignoring Private Fields](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#ignoring-private-fields)

Some fields must be exported from structs so that other packages can correctly interact with the type. However, the nature of these fields may be sensitive, so in these circumstances, we would like the JSON encoder to ignore the field entirely—even when it is set. This is done using the special value `-` as the value argument to a `json:` struct tag.

This example fixes the issue of exposing the user’s password.

```
package main

import (
    "encoding/json"
    "fmt"
    "log"
    "os"
    "time"
)

type User struct {
    Name      string    `json:"name"`
    Password  string    `json:"-"`
    CreatedAt time.Time `json:"createdAt"`
}

func main() {
    u := &User{
        Name:      "Sammy the Shark",
        Password:  "fisharegreat",
        CreatedAt: time.Now(),
    }

    out, err := json.MarshalIndent(u, "", "  ")
    if err != nil {
        log.Println(err)
        os.Exit(1)
    }

    fmt.Println(string(out))
}
```

When you run this example, you’ll see this output:

```
Output{
  "name": "Sammy the Shark",
  "createdAt": "2026-03-24T04:52:04.200849459-04:00"
}
```

The only thing we’ve changed in this example from previous ones is that the password field now uses the special `"-"` value for its `json:` struct tag. In the output from this example, the `password` field is no longer present.

In real-world applications, sensitive data such as passwords should never be stored in plaintext. Instead, they should be securely hashed before storage or processing.

These features of the `encoding/json` package — `omitempty`, `"-"`, and [other options](https://pkg.go.dev/encoding/json#Marshal) — are not part of the Go language itself. Their behavior is defined by the implementation of the `encoding/json` package. Many third-party libraries follow similar conventions, but it is important to consult the documentation of any package that uses struct tags to understand how they are interpreted.

## [Using `reflect.StructTag.Get` vs `Lookup`](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#using-reflect-structtag-get-vs-lookup)

In earlier sections, we saw that struct tags do not affect program behavior on their own and must be examined by other Go code at runtime. The `reflect` package provides the tools necessary to inspect struct fields and retrieve their associated tags. Understanding how to properly access these tags becomes particularly important when you’re building libraries, validation frameworks, or any tooling that needs to interpret struct metadata accurately.

When working with struct tags using reflection, Go provides two distinct methods for retrieving tag values:

- `reflect.StructTag.Get` — Returns the value associated with a key in the tag string
- `reflect.StructTag.Lookup` — Returns both the value and a boolean indicating whether the key was found

At first glance, these methods appear similar and may seem interchangeable. However, they behave fundamentally differently in a subtle but important way: they handle the distinction between a tag key that is **absent** versus one that is **present but set to an empty string**. This distinction matters when you need to differentiate between “no tag specified” and “tag explicitly set to empty,” which can have different semantic meanings in your application logic.

Understanding when to use each method is crucial for writing robust code that correctly interprets struct tags. The `Get` method is simpler and sufficient for most straightforward use cases, while `Lookup` provides the additional precision needed when the presence or absence of a tag itself carries meaning.

Try the following example to observe this behavior in practice:

```
package main

import (
    "fmt"
    "reflect"
)

type User struct {
    Name  string `json:"name"`
    Email string `json:""`
    Age   int
}

func main() {
    t := reflect.TypeOf(User{})

    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)

        // Retrieve using Get
        getValue := field.Tag.Get("json")

        // Retrieve using Lookup
        lookupValue, ok := field.Tag.Lookup("json")

        fmt.Printf("Field: %s\n", field.Name)
        fmt.Printf("  Get: %q\n", getValue)
        fmt.Printf("  Lookup: %q, Exists: %v\n\n", lookupValue, ok)
    }
}
```

When you run this program, you will see output similar to the following:

```
Field: Name
  Get: "name"
  Lookup: "name", Exists: true

Field: Email
  Get: ""
  Lookup: "", Exists: true

Field: Age
  Get: ""
  Lookup: "", Exists: false
```

This example demonstrates three distinct scenarios that help illustrate the behavioral differences between `Get` and `Lookup`. Let’s break down each field and what the output reveals:

- The `Name` field has a `json` tag explicitly set to `"name"`. In this straightforward case, both `Get` and `Lookup` behave identically. `Get` returns the string `"name"`, and `Lookup` also returns `"name"` with the boolean `true`, confirming that the tag exists. This is the most common scenario you’ll encounter.
    
- The `Email` field is more interesting; it has a `json` tag that is explicitly defined but set to an empty string (`json:""`). Notice that both methods return an empty string as the value. However, `Lookup` returns `true` for the `ok` variable, indicating that the `json` tag **does exist** on this field, even though its value is empty. This distinction is important: the tag is present; it’s just been deliberately set to an empty value.
    
- The `Age` field has no `json` tag at all. Here’s where the critical difference emerges. `Get` returns an empty string, which looks identical to the result for the `Email` field. But `Lookup` returns an empty string with `ok` set to `false`, explicitly telling us that the `json` tag does not exist on this field. This is the key insight: without `Lookup`, you cannot distinguish between an absent tag and an empty tag value.
    

### [Understanding the Practical Implications](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#understanding-the-practical-implications)

The difference between the two methods becomes clear when comparing these cases:

- `Get("json")` returns an empty string in two situations: when the tag is completely missing (like `Age`) and when it is explicitly set to an empty string (like `Email`). Because it only returns a single string value, there is no way to distinguish between these two scenarios based on the return value alone.
    
- `Lookup("json")` returns two values: the tag value (a string) and a boolean indicating whether the key exists in the tag string. This second return value provides the critical piece of information that `Get` lacks. When the boolean is `true`, you know the tag was explicitly defined on the field, regardless of whether its value is empty. When the boolean is `false`, you know the tag key was never specified at all.
    

This distinction matters in real-world applications. For example, consider a serialization library that needs to handle three cases differently:

1. **Tag present with a value** (`json:"fieldName"`): Use the specified name
2. **Tag present but empty** (`json:""`): Use the field as-is or apply some default behavior
3. **Tag absent** (no tag): Skip the field entirely or use a fallback strategy

With `Get`, you can only distinguish case 1 from cases 2 and 3. With `Lookup`, you can distinguish all three cases, enabling more nuanced behavior in your code.

### [When to Use Each Method](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#when-to-use-each-method)

In many simple use cases, `Get` is sufficient and should be your default choice. If you only need to retrieve a tag value and don’t care whether an empty string means “tag is empty” versus “tag is absent,” then `Get` provides a cleaner API with a single return value. This is appropriate for cases where you’ll treat both scenarios the same way—for instance, if you’ll use a default value whenever `Get` returns an empty string.

However, when writing libraries, frameworks, or tooling that rely on struct tags for configuration or metadata, `Lookup` is often preferred because it provides more precise control over how tag values are interpreted. For example:

- **Validation libraries** might treat an absent tag as “no validation required” but an empty tag as “validation required with default rules”
- **ORM libraries** might treat an absent tag as “use the default column name” but an empty tag as “this field should not be persisted”
- **Configuration parsers** might treat an absent tag as “required field” but an empty tag as “optional field with no environment variable mapping”

In these scenarios, the presence or absence of the tag itself carries semantic meaning that affects how your code behaves. Using `Lookup` allows your code to make these distinctions correctly, leading to more predictable and maintainable behavior.

As a general guideline: use `Get` when you only care about the tag’s value, and use `Lookup` when the presence or absence of the tag itself is meaningful to your application logic.

In many real-world applications, a single struct is often used across multiple layers of a program. This is particularly common in web applications and microservices, where the same data structure needs to interact with various components of the system. For example, consider a typical web application flow:

1. A user submits data through an HTML form
2. Your web framework parses the form data and binds it to a struct
3. Your application validates the struct fields against business rules
4. The validated data is stored in a database
5. The same data might be serialized to JSON and sent to other services or returned as an API response

Without struct tags, you might need to create separate struct definitions for each of these layers, leading to significant code duplication and increased maintenance burden. However, Go allows you to define **multiple struct tags on a single field**, enabling a single struct definition to serve all these purposes simultaneously.

Each tag is written as a key-value pair, and multiple tags are separated by spaces within the same backtick-delimited string. This approach keeps your code DRY (Don’t Repeat Yourself) while maintaining compatibility with different libraries and frameworks.

Consider the following example that demonstrates a struct equipped to work across multiple layers of an application:

```
type User struct {
    Name  string `json:"name" db:"user_name" validate:"required" form:"name"`
    Email string `json:"email" db:"email_address" validate:"required,email" form:"email"`
}
```

In this struct, each field defines several tags that serve distinct purposes at different layers of the application:

- The `json` tag controls how the field is encoded and decoded in JSON. This is used when marshaling structs to JSON for API responses or unmarshaling JSON request bodies into structs. For example, `json:"name"` tells the `encoding/json` package to use `"name"` as the key in the JSON object.
    
- The `db` tag is used by database libraries (such as `sqlx`, `gorm`, or other ORMs) to map struct fields to column names in database tables. Here, `db:"user_name"` indicates that the `Name` field corresponds to a `user_name` column in the database. This is useful when your database naming conventions differ from your Go naming conventions.
    
- The `validate` tag defines validation rules, commonly used with validation libraries like `go-playground/validator`. The value `validate:"required"` means the field must be present and non-empty, while `validate:"required,email"` specifies that the field must be both present and contain a valid email address format.
    
- The `form` tag is used by web frameworks (such as Gin, Echo, or the standard library’s form parsing) to bind form input to struct fields. When parsing HTML form data or URL query parameters, `form:"name"` tells the framework to look for a form field named `"name"` and map its value to this struct field.
    

Each library reads only the tag key it recognizes and ignores the others. This is possible because each package uses the `reflect` package to look up only the specific tag keys it cares about. When a library encounters a struct tag, it queries for its own key (like `"json"` or `"db"`) and completely disregards any other keys present in the tag string.

This means that multiple tags can coexist on the same field without interfering with one another. For example, when you call `json.Marshal()` on a `User` struct:

- The `encoding/json` package reads only the `json` tag using `field.Tag.Get("json")`
- It completely ignores the `db`, `validate`, and `form` tags
- Other libraries operating on the same struct will similarly read only their own tags

### [Rules and Best Practices for Multiple Tags](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#rules-and-best-practices-for-multiple-tags)

There are several important rules to keep in mind when defining multiple tags:

- Tags must be separated by spaces, not commas. The space character is what separates one key-value pair from another. For example, `json:"name" db:"user_name"` is correct, while `json:"name",db:"user_name"` would be treated as a single malformed tag.
    
- Each tag key should appear only once within the tag string. Having duplicate keys like `json:"name" json:"username"` is technically allowed by Go, but the behavior when using `Get` or `Lookup` is undefined—you’ll get the first match, but this is not guaranteed.
    
- The entire tag must be enclosed in backticks (`` ` ``), not regular quotes. This is because struct tags are raw string literals, which means backslashes and other special characters within them are treated literally. Using double quotes would require escaping special characters.
    
- Tag values must be properly quoted. Each value in a key-value pair must be enclosed in double quotes: `json:"value"` is correct, while `json:value` is not.
    
- Maintain consistent spacing. While the Go compiler will parse tags with irregular spacing, consistent formatting (a single space between tag pairs) makes tags easier to read and maintain.
    

### [When to Use Multiple Tags](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#when-to-use-multiple-tags)

This pattern of combining multiple tags is widely used in Go applications, especially in web services and APIs where a single struct often serves as the central representation of data flowing through different parts of the system. Common scenarios where multiple tags are beneficial include:

- **REST API services** that need to handle JSON encoding/decoding, database persistence, and input validation
- **Form-handling applications** where user input needs to be bound, validated, and stored
- **Microservices** that exchange data in JSON format while persisting to databases with different naming conventions
- **GraphQL servers** that might use `json` tags for response serialization and `gql` tags for schema generation
- **Configuration management** where structs are used to parse config files (YAML, TOML) and validate their contents

By leveraging multiple struct tags, you can keep your data models concise while still supporting the requirements of different libraries and frameworks. This reduces duplication, maintains a single source of truth for your data structures, and makes your codebase easier to maintain as requirements evolve.

So far, we have seen how struct tags are used by standard library packages such as `encoding/json`, as well as how multiple libraries can interpret different tags on the same struct field. However, struct tags are not limited to predefined keys like `json`, `xml`, or `db`. You can define and use your own custom tags to control the behavior of your programs.

Struct tags are simply strings associated with struct fields. Any Go code can read and interpret them using the `reflect` package. This makes it possible to build declarative APIs where users express their intent through struct tags rather than imperative configuration code.

Consider a practical example where you want to populate a struct using [environment variables](https://www.digitalocean.com/community/tutorials/how-to-read-and-set-environmental-and-shell-variables-on-linux). Instead of manually reading each environment variable and assigning it to a field, you can define a custom `env` tag and write generic code to handle the mapping automatically.

```
package main

import (
    "fmt"
    "os"
    "reflect"
)

type Config struct {
    Port string `env:"APP_PORT"`
    Host string `env:"APP_HOST"`
}
```

In this struct, each field has an `env` tag that specifies which environment variable to read from. This is a declarative approach; you can see at a glance where each field’s value comes from.

Next, we can write a function that reads these tags using reflection and assigns values to the struct fields:

```
func loadEnv(cfg interface{}) {
    v := reflect.ValueOf(cfg).Elem()
    t := v.Type()

    for i := 0; i < v.NumField(); i++ {
        fieldValue := v.Field(i)
        fieldType := t.Field(i)

        tagValue := fieldType.Tag.Get("env")
        if tagValue == "" {
            continue
        }

        if value, exists := os.LookupEnv(tagValue); exists {
            if fieldValue.CanSet() && fieldValue.Kind() == reflect.String {
                fieldValue.SetString(value)
            }
        }
    }
}
```

This function uses reflection to inspect and modify the struct. Here’s how it works:

- **Get the reflect value**: The function accepts an `interface{}` parameter, allowing it to work with any struct type. It calls `reflect.ValueOf(cfg).Elem()` to dereference the pointer and access the actual struct value. Reflection can only modify values through pointers.
- **Get type information**: Calling `v.Type()` returns metadata about the struct, including field names, types, and tags.
- **Iterate over fields**: The function loops through each field in the struct. For each field, `fieldValue` represents its current value, and `fieldType` provides metadata including the struct tag.
- **Read the struct tag**: For each field, `fieldType.Tag.Get("env")` retrieves the `env` tag value. If the tag is empty or missing, the field is skipped.
- **Look up the environment variable**: If an `env` tag exists, `os.LookupEnv(tagValue)` checks if that environment variable is set. Unlike `os.Getenv`, this function returns both the value and a boolean indicating whether it exists.
- **Set the field value**: Before setting a value, the function checks that the field is exported (using `CanSet()`) and is a string type (using `Kind()`). These checks prevent runtime panics.

You can use this function as follows:

```
func main() {
    os.Setenv("APP_PORT", "8080")
    os.Setenv("APP_HOST", "localhost")

    cfg := &Config{}
    loadEnv(cfg)

    fmt.Printf("%+v\n", cfg)
}
```

```
Output{Port:8080 Host:localhost}
```

Both fields are successfully populated from their corresponding environment variables. The `loadEnv` function works for any struct that uses the `env` tag; you can add more fields or create different structs, and the same function handles them all.

### [Important Considerations](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#important-considerations)

When building your own struct tag processing logic, keep these points in mind:

- **Only exported fields work**: Reflection can only read and modify [fields that start with an uppercase letter](https://www.digitalocean.com/community/tutorials/understanding-package-visibility-in-go). Unexported fields will cause `CanSet()` to return `false`.
- **Pass a pointer to modify fields**: The `loadEnv` function requires a pointer to a struct, not a struct value. Passing a struct value directly will cause a panic.
- **Check types before setting values**: Always verify the field’s [type](https://www.digitalocean.com/community/tutorials/understanding-data-types-in-go) using `Kind()` before calling type-specific setters like `SetString()` or `SetInt()`. This prevents runtime panics.
- **Use `Lookup` when tag presence matters**: We used `Get` to retrieve tag values, but `Lookup` is better when you need to distinguish between a missing tag and an empty tag. For example, an empty `env` tag might mean “use the field name as the variable name,” while no tag means “skip this field.”
- **Handle errors properly**: This simple example silently skips fields it can’t process. Production code should collect and return errors for debugging.

### [Extending the Pattern](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#extending-the-pattern)

Custom struct tags can be used for many purposes beyond environment variables:

- **Feature flags**: Control whether fields should be logged, cached, or included in responses
- **Database migrations**: Specify column types, constraints, or indexes
- **API documentation**: Attach descriptions or example values for automatic doc generation
- **Validation**: Define custom validation rules specific to your domain

Many popular Go libraries use this pattern, including `envconfig` and `viper` for configuration, `go-playground/validator` for validation, `gorm` and `sqlx` for database operations, and web frameworks like `gin` and `echo` for request binding. By understanding how to read and interpret struct tags, you can build similar abstractions for your own applications.

Throughout this article, we have primarily focused on the `json` struct tag provided by the `encoding/json` package. However, struct tags are widely used across the Go ecosystem, and many libraries define their own tag keys to control behavior.

While there is no official standard for struct tags in the Go language itself, a number of conventions have emerged over time. Many of these are documented in the [Go wiki](https://go.dev/wiki/Well-known-struct-tags).

The following table summarizes some of the most commonly used struct tag keys and their typical use cases:

|Tag Key|Description|
|---|---|
|`json`|Controls JSON encoding and decoding using `encoding/json`|
|`xml`|Controls XML encoding and decoding using `encoding/xml`|
|`yaml`|Used for YAML serialization (commonly with `gopkg.in/yaml.v3`)|
|`bson`|Used for BSON serialization (MongoDB Go driver)|
|`db`|Maps struct fields to database columns (SQL libraries, ORMs)|
|`mapstructure`|Decodes generic maps into structs (used in configuration libraries)|
|`validate`|Defines validation rules for struct fields|
|`form`|Binds HTTP form data to struct fields in web frameworks|
|`query`|Maps URL query parameters to struct fields|
|`env`|Maps environment variables to struct fields|
|`toml`|Used for TOML configuration parsing|

Each of these tags is interpreted by a specific library or framework. For example:

- The `encoding/json` package reads the `json` tag.
- YAML libraries read the `yaml` tag.
- Validation libraries process the `validate` tag to enforce rules.
- Configuration libraries often use `mapstructure`, `env`, or `toml` tags.

Despite their widespread use, struct tags are not enforced or interpreted by the Go compiler. Instead, they are treated as plain string metadata attached to struct fields. It is up to each library to parse and apply the tag values according to its own rules.

This flexibility is what makes struct tags so powerful. They allow different parts of a program—or even entirely different libraries—to share a common data structure while applying their own logic through tags.

As you work with more Go libraries, you will encounter additional struct tag conventions. Understanding how struct tags are defined and interpreted will help you quickly adapt to new tools and frameworks.

## [FAQs](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#faqs)

### [1. What is the correct syntax for a struct tag in Go, and where does it go in the struct definition?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#1-what-is-the-correct-syntax-for-a-struct-tag-in-go-and-where-does-it-go-in-the-struct-definition)

A struct tag is a string literal enclosed in backticks (`` ` ``) that appears immediately after the field type in a struct definition. The tag consists of one or more key-value pairs in the format `key:"value"`, with multiple pairs separated by spaces:

```
type User struct {
    Name  string `json:"name"`
    Email string `json:"email" db:"email_address" validate:"required,email"`
}
```

Each key-value pair must have the value enclosed in double quotes. The tag must use backticks (raw string literals), not regular quotes, to avoid escaping issues.

### [2. What is the difference between `reflect.StructTag.Get` and `reflect.StructTag.Lookup` in Go?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#2-what-is-the-difference-between-reflect-structtag-get-and-reflect-structtag-lookup-in-go)

Both methods retrieve struct tag values, but they handle missing tags differently:

- `Get(key)` returns the value for the specified key as a string. If the key doesn’t exist, it returns an empty string (`""`).
- `Lookup(key)` returns two values: the tag value as a string and a boolean indicating whether the key exists.

The key difference is distinguishing between an absent tag and an empty tag value:

```
type Example struct {
    Field1 string `json:"name"`   // tag present with value
    Field2 string `json:""`       // tag present but empty
    Field3 string                  // tag absent
}
```

For `Field2` and `Field3`, `Get("json")` returns `""` for both, making them indistinguishable. However, `Lookup("json")` returns `("", true)` for `Field2` and `("", false)` for `Field3`, allowing you to differentiate between them.

Use `Get` when you only care about the tag value. Use `Lookup` when the presence or absence of a tag itself carries meaning.

### [3. How do you add multiple tag keys to a single struct field, for example both json and db?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#3-how-do-you-add-multiple-tag-keys-to-a-single-struct-field-for-example-both-json-and-db)

Separate multiple tag keys with spaces within the same backtick-delimited string:

```
type User struct {
    Name  string `json:"name" db:"user_name"`
    Email string `json:"email" db:"email_address" validate:"required"`
}
```

Each library reads only its own tag key and ignores the others. The `encoding/json` package reads the `json` tag, database libraries read the `db` tag, and validation libraries read the `validate` tag. Multiple tags can coexist without interfering with each other.

Important rules:

- Separate tags with spaces, not commas
- Each key should appear only once
- All tag values must be in double quotes
- The entire tag string must be in backticks

### [4. What does `omitempty` do in a Go struct tag, and does it behave the same way in `encoding/json` and `encoding/xml`?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#4-what-does-omitempty-do-in-a-go-struct-tag-and-does-it-behave-the-same-way-in-encoding-json-and-encoding-xml)

The `omitempty` option tells the encoder to skip a field when it has a zero value. In JSON:

```
type User struct {
    Name  string   `json:"name"`
    Email string   `json:"email,omitempty"`
    Age   int      `json:"age,omitempty"`
}
```

If `Email` is an empty string or `Age` is `0`, those fields won’t appear in the JSON output.

Zero values include: `""` (empty string), `0`, `false`, `nil`, and empty slices or maps.

While both `encoding/json` and `encoding/xml` support `omitempty`, they may handle certain types differently. For example, `time.Time` might encode even when set to its zero value, depending on the package. Always consult the specific package documentation for exact behavior.

### [5. How do you define and read a custom struct tag using the reflect package?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#5-how-do-you-define-and-read-a-custom-struct-tag-using-the-reflect-package)

Define a custom tag just like any other struct tag:

```
type Config struct {
    Port string `env:"APP_PORT"`
    Host string `env:"APP_HOST"`
}
```

Read the tag using reflection:

```
import "reflect"

func readTag(obj interface{}) {
    t := reflect.TypeOf(obj)
    
    for i := 0; i < t.NumField(); i++ {
        field := t.Field(i)
        tagValue := field.Tag.Get("env")
        
        if tagValue != "" {
            fmt.Printf("Field %s has env tag: %s\n", field.Name, tagValue)
        }
    }
}
```

You can use `field.Tag.Get("key")` to retrieve the tag value, or `field.Tag.Lookup("key")` to check both the value and whether the tag exists.

### [6. What does `go vet` check for in struct tags, and what kinds of tag errors does it not catch?](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#6-what-does-go-vet-check-for-in-struct-tags-and-what-kinds-of-tag-errors-does-it-not-catch)

`go vet` checks for basic struct tag syntax errors, such as:

- Malformed key-value pairs
- Improperly quoted values
- Common mistakes in standard library tags (like `json`, `xml`)

For example, it will catch:

```
type Bad struct {
    Name string `json:name`      // vet catches: missing quotes
    Age  int    `json:"age"bad`  // vet catches: malformed syntax
}
```

However, `go vet` does **not** catch:

- Typos in tag keys (e.g., `jsn:"name"` instead of `json:"name"`)
- Invalid tag values that are syntactically correct (e.g., `json:"name,omiempty"` - typo in “omitempty”)
- Custom tag keys that don’t exist in any library
- Logical errors in tag usage

`go vet` only validates syntax, not semantic correctness. You need runtime testing to catch logical errors in struct tags.

## [Conclusion](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go#conclusion)

In this article, you learned how to use struct tags in Go to attach metadata to struct fields. You explored how the `encoding/json` package uses struct tags to control JSON serialization, including customizing field names with the `json` tag, omitting empty values with `omitempty`, and excluding fields with `"-"`. You also learned the difference between `reflect.StructTag.Get` and `Lookup` for reading tag values.

You saw how to combine multiple struct tags on a single field to make structs work across different application layers, and how to create custom struct tags for your own needs. With this knowledge, you can now work effectively with Go libraries that use struct tags, write more maintainable code using declarative metadata, and build your own tag-based abstractions when needed.

For more information, refer to the [Go reflection documentation](https://pkg.go.dev/reflect) and the [list of well-known struct tags](https://go.dev/wiki/Well-known-struct-tags) maintained by the Go community. Check out the following articles for more Go-related tutorials:

- [How To Use JSON in Go](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go)
- [Understanding Maps in Go](https://www.digitalocean.com/community/tutorials/understanding-maps-in-go)
- [Defining Structs in Go](https://www.digitalocean.com/community/tutorials/defining-structs-in-go)
- [How To Write Unit Tests in Go](https://www.digitalocean.com/community/tutorials/how-to-write-unit-tests-in-go-using-go-test-and-the-testing-package)

[![Creative Commons](https://www.digitalocean.com/api/static-content/v1/images?src=%2F_next%2Fstatic%2Fmedia%2Fcreativecommons.c0a877f1.png&width=384)](https://creativecommons.org/licenses/by-nc-sa/4.0/)This work is licensed under a Creative Commons Attribution-NonCommercial- ShareAlike 4.0 International License.