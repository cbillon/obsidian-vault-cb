---
link: https://betterstack.com/community/guides/scaling-go/json-in-go/
excerpt: Ship higher-quality software faster. Be the hero of your engineering teams.
slurped: 2025-05-29T09:50
title: A Comprehensive Guide to Using JSON in Go | Better Stack Community
tags:
  - go
  - json
---

JavaScript Object Notation (JSON) is a data format that has gained popularity since its introduction in the early 2000s. It has become a ubiquitous standard for data transfer across systems (such as API request bodies), surpassing previous formats like XML.

Go, with its strong typing and emphasis on simplicity and efficiency, is well-suited for working with JSON data. Whether you're building web applications, working with APIs, or storing data in databases, Go provides a range of tools and techniques for working with JSON data.

In this article, we will provide a complete guide to working with JSON in Go. We will start by covering the serialization and deserialization of JSON data in Go, and then discuss how to validate JSON payloads. We will also cover best practices and common pitfalls to avoid when working with JSON data in Go.

By the end of this article, you'll have a deep understanding of how to work with JSON data in Go, and be well-equipped to write efficient, maintainable, and error-free code that works seamlessly with JSON data.

## Prerequisites

To follow along with this article, ensure that you have the latest version of Go installed on your machine. If you are missing Go, you can [find the installation instructions here](https://go.dev/doc/install).

You can view and run the examples in this tutorial by setting up the [demo repository](https://github.com/betterstack-community/json-in-go):

```
git clone https://github.com/betterstack-community/json-in-go
```

Then, install the necessary dependencies:

This article also assumes that you are comfortable with JSON syntax. If you are unfamiliar with JSON, refer to [this resource for more information](https://www.json.org/json-en.html).

## JSON terminology in Go

There are two key terminologies to note when working with JSON in Go:

1. **Marshalling**: the act of converting a Go data structure into valid JSON.
2. **Unmarshalling**: the act of parsing a valid JSON string into a data structure in Go.

In other languages, marshalling is often referred to as “serializing”, while unmarshalling is referred to as “deserializing”. For the rest of this article, we will sticking with Go's terminology.

The following diagram illustrates these processes.

We will start by looking at how JSON access is achieved through the built-in `encoding/json` package.

## Unmarshalling JSON in Go

We will start by discussing the unmarshalling process using the `json.Unmarshal()` method:

```
func Unmarshal(data []byte, v any) error
```

This methods accepts two arguments: the first is a `[]byte` which represents the JSON object to unmarshal, and the second is `any` (introduced in Go 1.18 as an alias for `interface{}`) which should be a pointer to the target data structure for storing the result of unmarshalling the JSON data.

Here's an example that unmarshals a JSON object to a `map[string]any` type:

examples/text_to_interface/main.go

Copied!

```
func main() {
    input := `{
        "name": "John Doe",
        "age": 15,
        "hobbies": ["climbing", "cycling", "running"]
    }`

    var target map[string]any

    err := json.Unmarshal([]byte(input), &target)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    for k, v := range target {
        fmt.Printf("k: %s, v: %v\n", k, v)
    }
}
```

The `map[string]any` data type in Go is a generic container that can hold values of any type, including complex nested structures. In this case, the JSON keys will be unmarshalled into the `string` type, and their corresponding values will be unmarshalled into the `any` type of the `map[string]any`. A `nil` map is permissible here as `Unmarshal()` will allocate a new map in such cases. You can refer to the [official documentation on `Unmarshal()` for more information](https://pkg.go.dev/encoding/json#Unmarshal).

The expected output looks like this:

```
k: name, v: John Doe
k: age, v: 15
k: hobbies, v: [climbing cycling running]
```

If an invalid JSON object is provided, an error will be returned by `Unmarshal()`. A common example of an invalid JSON object is one that has a [trailing comma](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Trailing_commas):

```
input := `
  {
    name: "John Doe",
    "age": 15,
    "hobbies": ["climbing", "cycling", "running"],
  }
`
```

Attempting to unmarshal the JSON object above would yield the following error:

```
2023/03/23 07:24:39 Unable to marshal JSON due to invalid character '}' looking for beginning of object key string
```

While `map[string]any` can be used to unmarshal JSON, it is not the most optimal solution for several reasons:

- You lose the type safety and compile-time checks that are provided by Go's static type system. This can make it harder to catch errors and maintain code over time.
    
- It can be slower than working with typed structs or custom types that implement the `json.Unmarshaler` interface. This is because accessing fields in a map requires a dynamic lookup, whereas accessing fields in a struct is done statically at compile-time.
    
- It can make it more difficult to reason about the structure of the JSON data being unmarshalled, as the values can be of any type. This can lead to more verbose and error-prone code.
    

In general, it's best to use typed structs or custom types whenever possible for JSON unmarshalling in Go. These types provide better type safety, performance, and maintainability than using `map[string]any`.

### Using structs for JSON unmarshalling

When using structs for unmsarshalling JSON objects, the field names in the object are mapped to the field names in the `struct` and the values are assigned accordingly. Let's look at a simple example of a `Dog` struct type and how unmarshalling works with structs below:

examples/text_to_struct.go

Copied!

```
type Dog struct {
    Breed         string
    Name          string
    FavoriteTreat string
    Age           int
}

func main() {
    input := `{
        "Breed": "Golden Retriever",
        "Age": 8,
        "Name": "Paws",
        "FavoriteTreat": "Kibble"
    }`

    var dog Dog

    err := json.Unmarshal([]byte(input), &dog)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    fmt.Printf(
        "%s is a %d years old %s who likes %s\n",
        dog.Name,
        dog.Age,
        dog.Breed,
        dog.FavoriteTreat,
    )
}
```

Here's the result:

```
Paws is a 8 years old Golden Retriever who likes Kibble
```

In this example, the `json.Unmarshal()` function takes the `input` data, along with a pointer to a `Dog` struct. The function then populates the fields in the struct with the corresponding values from the JSON data.

You can also unmarshal more complex JSON objects such as the one shown below:

assets/complex.json

Copied!

```
{
  "name": "James Peterson",
  "age": 37,
  "address": {
    "line1": "Block 78 Woodgrove Avenue 5",
    "line2": "Unit #05-111",
    "postal": "654378"
  },
  "pets": [
    {
      "name": "Lex",
      "kind": "Dog",
      "age": 4,
      "color": "Gray"
    },
    {
      "name": "Faye",
      "kind": "Cat",
      "age": 6,
      "color": "Orange"
    }
  ]
}
```

You must design your target struct to include other structs as fields and allow `Unmarshal()` to handle the mapping of fields accordingly.

examples/complex_json/main.go

Copied!

```
type (
    FullPerson struct {
        Address Address
        Name    string
        Pets    []Pet
        Age     int
    }

    Pet struct {
        Name  string
        Kind  string
        Color string
        Age   int
    }

    Address struct {
        Line1  string
        Line2  string
        Postal string
    }
)

func main() {
    b, err := os.ReadFile("assets/complex.json")
    if err != nil {
        log.Fatalf("Unable to read file due to %s\n", err)
    }

    var person FullPerson

    err = json.Unmarshal(b, &person)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    litter.Dump(person)
}
```

```
main.FullPerson{
  Name: "James Peterson",
  Age: 37,
  Address: examples.Address{
    Line1: "Block 78 Woodgrove Avenue 5",
    Line2: "Unit #05-111",
    Postal: "654378",
  },
  Pets: []examples.Pet{
    examples.Pet{
      Name: "Lex",
      Kind: "Dog",
      Age: 4,
      Color: "Gray",
    },
    examples.Pet{
      Name: "Faye",
      Kind: "Cat",
      Age: 6,
      Color: "Orange",
    },
  },
}
```

As you can see, `Unmarshal()` easily handles both nested JSON objects and nested JSON arrays. Do note that any nested structs must also match the same fields in the JSON.

Now that we've explored how JSON unmarshalling works in Go, let's look at some gotchas when unmarshalling into structs in Go.

## Common pitfalls with JSON unmarshalling in Go

While JSON unmarshalling is a relatively simple task in Go, there are several common pitfalls that you should be aware of to avoid mistakes in your business logic. In this section, we will discuss some of the most common gotchas that you might face when unmarshalling JSON in Go and provide tips on how to avoid them

If the input JSON contains additional fields that are not a part of the target struct fields, they will be discarded when unmarshalled. Using the same `Dog` struct declared above, we can demonstrate this behavior:

examples/extra_fields/main.go

Copied!

```
func main() {
    input := `{
        "Breed": "Golden Retriever",
        "Age": 8,
        "Name": "Paws",
        "FavoriteTreat": "Kibble",
        "Dislikes": "Cats"
    }`


    var dog Dog

    err := json.Unmarshal([]byte(input), &dog)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    litter.Dump(dog)
}
```

Notice that the input JSON contains the additional field `Dislikes` but that field is not included in the target `Dog` struct. Therefore, it is discarded:

```
main.Dog{
  Breed: "Golden Retriever",
  Age: 8,
  Name: "Paws",
  FavoriteTreat: "Kibble",
}
```

### 2. Missing fields fallback to zero values

Missing fields in the input JSON will cause the zero value of the corresponding struct field to be used instead:

examples/missing_fields/main.go

Copied!

```
func main() {
    input := `{
        "Breed": "Golden Retriever",
        "Age": 8,
        "Name": "Paws"
    }`

    var dog Dog

    err := json.Unmarshal([]byte(input), &dog)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    fmt.Printf("%s likes %s\n", dog.Name, dog.FavoriteTreat)
}
```

Since the `FavoriteTreat` field is been omitted from the input JSON, it will be an empty string in the resulting struct. If you wish to guarantee that a field is not omitted in the input JSON, you must use a validation library which will be discussed in a subsequent section.

### 3. Unmarshalling is case insensitive

The `Unmarshal()` method will match the field name of the input JSON to the field in the `struct` in a case insensitive manner as long as the characters and their order are the same.

examples/case_sensitivity/main.go

Copied!

```
func main() {
    input := `{
        "BreED": "Golden Retriever",
        "age": 8,
        "NaME": "Paws",
        "favoriTeTrEat": "Kibble"
    }`
    var dog Dog

    err := json.Unmarshal([]byte(input), &dog)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    fmt.Printf(
        "%s is a %d years old %s who likes %s\n",
        dog.Name,
        dog.Age,
        dog.Breed,
        dog.FavoriteTreat,
    )
}
```

```
Paws is a 8 years old Golden Retriever who likes Kibble
```

Notice how the `Dog` struct was populated successfully despite the casing of the fields in the input JSON.

### 4. Field names must match JSON keys exactly

When defining structs for unmarshalling JSON, it's important to ensure that the names of struct fields match the keys in the JSON data exactly. If there is a mismatch, the field will not be populated with the corresponding value from the JSON data.

examples/symbol_sensitivity/main.go

Copied!

```
func main() {
    input := `{
      "Breed": "Golden Retriever",
      "Age": 8,
      "Name": "Paws",
      "favorite_treat": "Kibble"
  }`

    var dog Dog

    err := json.Unmarshal([]byte(input), &dog)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    fmt.Printf(
        "%s is a %d years old %s who likes %s\n",
        dog.Name,
        dog.Age,
        dog.Breed,
        dog.FavoriteTreat,
    )
}
```

```
Paws is a 8 years old Golden Retriever who likes
```

As you can see, the input JSON uses the key `favorite_treat` but the `Dog` struct declares the field as `FavoriteTreat` so the unmarshalled `struct` does not use the input JSON’s value for `favorite_treat`. There is a workaround [using struct tags](https://betterstack.com/community/guides/scaling-go/json-in-go/#customizing-json-field-names-with-struct-tags) that will be discussed in a subsequent section.

### 5. Type aliases are preserved

If there are type alias fields in your struct, their values and type alias will be preserved when unmarshalled:

examples/type_alias.go

Copied!

```
type (
    TypeAliasExample string

    TypeAliasStruct struct {
        Example TypeAliasExample
    }
)

type (
    TypeAliasExample string

    TypeAliasStruct struct {
        Example TypeAliasExample
    }
)
```

```
main.TypeAliasStruct{
  Example: "Hello world",
}
```

Now that we have understood some of the gotchas of unmarshalling data into structs, let us explore the more complex components of unmarshalling.

## JSON Marshalling in Go

The `json.Marshal()` method does the opposite of `Unmarshal()` by converting a given data structure into a JSON. When working with the basic types in Go (strings, integers, slices, maps), it generates the corresponding JSON accordingly.

examples/basic_marshal/main.go

Copied!

```
func marshal(in any) []byte {
    out, err := json.Marshal(in)
    if err != nil {
        log.Fatalf("Unable to marshal due to %s\n", err)
    }

    return out
}

func main() {
    first := marshal(14)
    second := marshal("Hello world")
    third := marshal([]float32{1.66, 6.86, 10.1})
    fourth := marshal(map[string]int{"num": 15, "other": 17})
    fmt.Printf(
        "first: %s\nsecond: %s\nthird: %s\nfourth: %s\n",
        first,
        second,
        third,
        fourth,
    )
}
```

```
first: 14
second: "Hello world"
third: [1.66,6.86,10.1]
fourth: {"num":15,"other":17}
```

Note that we have abstracted `Marshal()` into a separate function to simplify the error handling process. Regardless, the example above illustrates how basic types in Go will be marshalled accordingly.

In most cases, you'll be working with more complex types like structs that represent database models or server responses. Marshalling these types requires more careful handling of the data, as the structure of the JSON output will depend on the structure of the Go type being marshalled.

### Marshalling structs

Similar to unmarshalling JSON into structs, you can also marshal a struct into JSON.

examples/struct_to_text/main.go

Copied!

```
func main() {
    p := Person{
        Name:  "John Jones",
        Age:   26,
        Email: "johnjones@email.com",
        Phone: "89910119",
        Hobbies: []string{
            "Swimming",
            "Badminton",
        },
    }

    b, err := json.Marshal(p)
    if err != nil {
        log.Fatalf("Unable to marshal due to %s\n", err)
    }

    fmt.Println(string(b))
}
```

```
{"Name":"John Jones","Age":26,"Email":"johnjones@email.com","Phone":"89910119","Hobbies":["Swimming","Badminton"]}
```

The `Marshal()` method produces a valid JSON from the given struct, including any nested JSON arrays or JSON objects.

Notice that the generated JSON is a single line without proper formatting. Although this is an ideal form when transmitting information through a network, it is not a very user friendly representation of the JSON.

If you wish to format the JSON object, you can use the `MarshalIndent()` method which performs the same function as `Marshal()` but applies some indentation to format the output.

```
b, err := json.MarshalIndent(p, "", "  ")
```

You should now observe the following output:

```
{
    "Name": "John Jones",
    "Age": 26,
    "Email": "johnjones@email.com",
    "Phone": "89910119",
    "Hobbies": [
        "Swimming",
        "Badminton"
    ]
}
```

You can configure two aspects of formatting with `MarshalIndent()`. The first is the prefix per line which appears at the start of every line. For most purposes, you would set this parameter to be an empty string. The second configures the indentation level which is two spaces in the above example.

In Go, struct tags are annotations that can be added to the fields of a struct to provide additional information about how the fields should be treated by various tools and libraries. Struct tags are strings that are added to the end of a field declaration, enclosed in backticks.

The most common use case for struct tags is to specify how a struct should be marshalled and unmarshalled to and from JSON. By adding tags to the fields of a struct, you can control how the fields are named, which fields are ignored, and how they are encoded and decoded.

For example, consider the following struct definition:

```
type Dog struct {
    Breed         string
    Name          string
    FavoriteTreat string
    Age           int
}

var dog = Dog{
  Breed: "Golden Retriever",
  Age: 8,
  Name: "Paws",
  FavoriteTreat: "Kibble",
}
```

At the moment, the `dog` variable will be marshalled into the following JSON object where the properties correspond exactly to the struct field names:

```
{"Breed":"Golden Retriever","Name":"Paws","FavoriteTreat":"Kibble","Age":8}
```

You can customize this output by using `json` struct tags on the `Dog` type:

```
type Dog struct {
    Breed         string `json:"breed"`
    Name          string `json:"name"`
    FavoriteTreat string `json:"favorite_treat"`
    Age           int    `json:"age"`
}
```

Note the syntax of the struct tag. It appears after the type of the field, surrounded by "``", and takes on the following format: `json:"<name>"`. Here, the `Name` field will be mapped to the JSON key "name", `Age` will be mapped to the JSON key "age", `FavoriteTreat` to "favorite_treat", and `Breed` to "breed". You will now observe the following JSON output:

```
{"breed":"Golden Retriever","name":"Paws","favorite_treat":"Kibble","age":8}
```

It also works the same way for unmarshalling. Assuming you have the following JSON input:

```
{
  "name": "Coffee",
  "breed": "Toy Poodle",
  "age": 5,
  "favorite_treat": "Kibble"
}
```

You can unmarshal it into the `Dog` struct shown below:

examples/struct_tags/main.go

Copied!

```
func main() {
    input := `{
  "name": "Coffee",
  "breed": "Toy Poodle",
  "age": 5,
  "favorite_treat": "Kibble"
}`

    var coffee Dog

    err := json.Unmarshal([]byte(input), &coffee)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    litter.Dump(coffee)
}
```

```
examples.UniversalDog{
  Breed: "Toy Poodle",
  Age: 5,
  Name: "Coffee",
  FavoriteTreat: "Kibble",
}
```

Do note that the case insensitivity and symbol sensitivity of unmarshalling will still apply with struct tags, but only to the standardized name declared in the tag (not the original field name).

### Other uses of struct tags

Beyond customizing the field names, struct tags can also be used to omit empty fields or to ignore fields altogether during marshalling and unmarshalling.

To omit an empty field (one with its zero value in Go), add the `omitempty` struct tag to the existing struct tag. To ignore a field whether it is empty or not, use `json:"-"`.

examples/struct_tags_2/main.go

Copied!

```
type User struct {
    Username string   `json:"username"`
    Password string   `json:"-"`
    Email    string   `json:"email"`
    Hobbies  []string `json:"hobbies,omitempty"`
}

func main() {
    user := User{
        Username: "admin",
        Password: "root",
        Email:    "admin@email.com",
    }

    b, err := json.MarshalIndent(user, "", "  ")
    if err != nil {
        log.Fatalf("Unable to marshal due to %s\n", err)
    }

    fmt.Println(string(b))
}
```

The above code generates the following JSON:

```
{
  "username": "admin",
  "email": "admin@email.com"
}
```

Notice that the `Password` field excluded from the marshalled JSON since it is ignored through `json:"-"`. This can be useful in situations where you want to include a field in a struct for internal use, but you don't want it to be exposed or serialized in the JSON output. Do note that ignored fields will not be populated during unmarshalling.

On the other hand, the `Hobbies` field is omitted since it was not initialized to a value in the `user` variable. This creates a more concise JSON output that only includes the necessary fields.

## Converting JSON to Go

Suppose you're working with an external API with complex JSON responses. For instance, the Spotify API has the following JSON schema for fetching a user:

```
{
    "country": "string",
    "display_name": "string",
    "email": "string",
    "explicit_content": {
        "filter_enabled": true,
        "filter_locked": true
    },
    "external_urls": {
        "spotify": "string"
    },
    "followers": {
        "href": "string",
        "total": 0
    },
    "href": "string",
    "id": "string",
    "images": [
        {
            "url": "https://i.scdn.co/image/ab67616d00001e02ff9ca10b55ce82ae553c8228\n",
            "height": 300,
            "width": 300
        }
    ],
    "product": "string",
    "type": "string",
    "uri": "string"
}
```

To manually map the JSON schema to a struct in Go would be a time consuming process, but this task can be eased through tools that can perform this conversion such as [JSON-to-Go](https://mholt.github.io/json-to-go/) and [JSON Typedef](https://jsontypedef.com/docs/go-codegen/).

For instance, the following `struct` is generated using the JSON-to-Go web interface. As you can see, the generated code is not entirely perfect, but it provides a good starting point for working with the JSON responses from the API.

While these tools can reduce the time spent manually converting the JSON schema to a Go `struct`, the generated code must be checked to ensure that the output meets your requirements.

## Validating JSON data

There are two main forms of JSON validation. The first kind is involves validating that a given JSON string is proper (i.e. not malformed). The second kind checks if the input JSON conforms to a predefined schema.

First, let's use the `json.Valid()` method to check for malformed JSON in Go:

examples/check_valid/main.go

Copied!

```
func main() {
    good := `{"name": "John Doe"}`
    bad := `{name: "John Doe"}`

    fmt.Println(json.Valid([]byte(good)))
    fmt.Println(json.Valid([]byte(bad)))
}
```

The `bad` input JSON string does not wrap the `name` key in double quotes.

As expected, the `good` JSON string will return `true` while the `bad` one will return `false`. This is a good first step to ensure that you are working with valid JSON data. However, its often necessary to enforce a particular schema for the input JSON. This can be achieved using third-party validation packages such as [go-playground/validator](https://github.com/go-playground/validator). It also relies on struct tags to perform data validation.

To begin using `validator`, install the package in your project. It has already been installed in the demo repository:

```
go get github.com/go-playground/validator/v10
```

The `validator` package provides [several struct tags](https://pkg.go.dev/github.com/go-playground/validator/v10) that can be used to validate JSON input. However, we will only go through a handful of the most commonly used ones in this section.

To implement data validation on a struct, you must use the `validate` tags shown below:

examples/validator/main.go

Copied!

```
type User struct {
    Username string `json:"username" validate:"required"`
    Password string `json:"password" validate:"required"`
    Email    string `json:"email"    validate:"required,email"`
    Age      int    `json:"age"      validate:"required,min=18,max=99"`
}
```

Notice that the validation rules are provided as a set of comma-separated values with some properties having a value after an `=` symbol.

In English, the validation rules can be understood as such:

1. Username must be present (non-empty string).
2. Password must be present (non-empty string).
3. Email must be present and it must follow the standard email format.
4. Age must be present and it must at least 18 and at most 99.

We can test these validation rules by attempting to unmarshal a valid and invalid JSON.

Given the following invalid JSON, we should expect the validation to flag out the invalid fields:

```
{
  "username": "johndoe",
  "email": "johndoe@emai",
  "age": -14
}
```

examples/validator.go

Copied!

```
func main() {
    input := `{
    "username": "johndoe",
    "email": "johndoe@emai",
    "age": -14
  }`

    var user User

    err := json.Unmarshal([]byte(input), &user)
    if err != nil {
        log.Fatalf("Unable to marshal JSON due to %s", err)
    }

    fmt.Printf("User before validation: %v\n", user)

    err = validator.New().Struct(user)
    if err != nil {
        log.Fatalf("Validation failed due to %v\n", err)
    }
}
```

As expected, the validator returns an error that flags the erroneous fields in the input JSON after unmarshalling:

```
User before validation: {johndoe  johndoe@emai -14}
Validation failed due to Key: 'ValidatedUser.Password' Error:Field validation for 'Password' failed on the 'required' tag
Key: 'ValidatedUser.Email' Error:Field validation for 'Email' failed on the 'email' tag
Key: 'ValidatedUser.Age' Error:Field validation for 'Age' failed on the 'min' tag
```

Note that the prior to using the `validator` package, the JSON was successfully unmarshalled since the validation struct tags are only evaluated when explicitly called.

By altering the input JSON to abide by the validation rules set out above, we can expect the validation to pass.

```
{
  "username": "johndoe",
  "password": "root",
  "email": "johndoe@email.com",
  "age": 18
}
```

This time, with a valid JSON input, the validation passes:

```
User before validation: {johndoe root johndoe@email.com 18}
User after validation: {johndoe root johndoe@email.com 18}
```

The `validator` package supports many more struct tags for validation so you are encouraged to give the [official documentation](https://pkg.go.dev/github.com/go-playground/validator/v10#section-readme) a thorough read.

## Defining custom behavior for marshalling and unmarshalling data

In Go, you can define custom behavior for marshalling data by implementing the `json.Marshaler` interface. This interface defines a single method, `MarshalJSON()` which takes no arguments and returns a byte slice and an error.

To implement the `json.Marshaler` interface, you need to define a new type that wraps the original type you want to marshal. This new type should have a method named `MarshalJSON()` that returns a byte slice and an error.

examples/custom_timestamp/main.go

Copied!

```
type (
    CustomTime struct {
        time.Time
    }

    Baby struct {
        BirthDate CustomTime `json:"birth_date"`
        Name      string    `json:"name"`
        Gender    string    `json:"gender"`
    }
)
```

In the above snippet, we defined a new `CustomTime` type that wraps a `time.Time` value. In is subsequently used in the `Baby` struct as the type of the `BirthDate` value.

Here's an example that marshals a value of type `Baby` below:

examples/custom_timestamp/main.go

Copied!

```
func main() {
    baby := Baby{
        Name:   "johnny",
        Gender: "male",
        BirthDate: CustomTime{
            time.Date(2023, 1, 1, 12, 0, 0, 0, time.Now().Location()),
        },
    }

    b, err := json.Marshal(baby)
    if err != nil {
        log.Fatalf("Unable to marshal due to %s\n", err)
    }

    fmt.Println(string(b))
}
```

You will observe the following output:

```
{"birth_date":"2023-01-01T12:00:00+01:00","name":"johnny","gender":"male"}
```

Notice how the `birth_date` presented in the [RFC 3339 format](https://www.rfc-editor.org/rfc/rfc3339). You can now define the custom marshalling behavior that will return a different format for `CustomTime` values (such as `DD-MM-YYYY`) instead of the default RFC 3339 timestamp format.

You only need to define a `MarshalJSON()` method for the type as shown below:

examples/custom_timestamp/main.go

Copied!

```
func (ct CustomTime) MarshalJSON() ([]byte, error) {
    return []byte(fmt.Sprintf(`%q`, ct.Time.Format("02-01-2006"))), nil
}
```

Now, when you marshal the `baby` variable once more, you will observe the new format for `birth_date`:

```
{"birth_date":"01-01-2023","name":"johnny","gender":"male"}
```

Customizing the unmarshalling behavior for a type works in a similar way. You need to implement the `json.Unmarshaler` type instead:

```
type Unmarshaler interface {
    UnmarshalJSON([]byte) error
}
```

For example, you can use the [dateparse](https://github.com/araddon/dateparse) package to ensure that various date formats can be used to supply a birth date instead of the fixed formats allowed by `time.Parse()`:

examples/custom_timestamp/main.go

Copied!

```
func (ct *CustomTime) UnmarshalJSON(input []byte) error {
    value := strings.Trim(string(input), `"`)

    t, err := dateparse.ParseAny(value)
    if err != nil {
        return err
    }

    ct.Time = t

    return nil
}
```

Here, the `UnmarshalJSON()` method unmarshals a JSON string in variety of formats into a `CustomTime` value as long as it is supported by the `dateparse` package. With this in place, any of the following date formats (and many others) will be unmarshalled successfully:

```
{"name": "Mary", "gender": "F", "birth_date": "19-02-2023"}
{"name": "Mary", "gender": "F", "birth_date": "Mon 30 Sep 2022 09:09:09 PM UTC"}
{"name": "Mary", "gender": "F", "birth_date": "2022/12/31"}
```

As you can see, customizing the unmarshalling behavior allows for tremendous flexibility when parsing all kinds of JSON data.

### The difference between JSON encoding and marshalling

The `encoding/json` package also provides two other constructs for working with JSON in Go which are `json.Encoder` and `json.Decoder`. These types essentially do the same thing as `Marshal` and `Unmarshal` but they operate on streams of data instead of JSON objects that are already fully loaded in memory.

For example, `json.Decoder` can read from an `io.Reader` (such as an `os.File`) and decode JSON values into a struct:

examples/json_decoder/main.go

Copied!

```
func main() {
    coffeeFile, err := os.Open("assets/coffee.json")
    if err != nil {
        log.Fatalf("Unable to read file due to %s\n", err)
    }

    var coffee Dog

    decoder := json.NewDecoder(coffeeFile)

    err = decoder.Decode(&coffee)
    if err != nil {
        log.Fatalf("Unable to decode due to %s\n", err)
    }

    litter.Dump(coffee)
}
```

You will observe the following output:

```
main.Dog{
  Breed: "Toy Poodle",
  Name: "Coffee",
  FavoriteTreat: "Kibble",
  Age: 5,
}
```

One difference between `json.Decode()` and `json.Unmarshal` is that the former allows you to display an error when the input JSON contains properties that do not match any non-ignored, exported fields in the destination unlike the latter where such fields are simply ignored.

This is done through the `DisallowUnknownFields()` method on the `Decoder`:

```
decoder := json.NewDecoder(coffeeFile)
decoder.DisallowUnknownFields()
```

Assuming the input JSON contains a property that is not present in the `Dog` struct:

```
{
  "name": "Coffee",
  "breed": "Toy Poodle",
  "age": 5,
  "favorite_treat": "Kibble",
  "color": "brown"
}
```

You will observe the following error:

```
2023/03/23 15:29:51 Unable to decode due to json: unknown field "color"
```

The `json.Encoder` type, on the other hand, writes the JSON encoding of a Go type into a provided writable stream (`io.Writer`). It is often used to write a JSON response to a client request:

examples/json_encoder/main.go

Copied!

```
func main() {
    newDog := Dog{
        Breed:         "Poodle",
        Age:           15,
        Name:          "Chloe",
        FavoriteTreat: "Dog Sticks",
    }

    mux := http.NewServeMux()
    mux.HandleFunc("/", func(w http.ResponseWriter, _ *http.Request) {
        encoder := json.NewEncoder(w)

        err := encoder.Encode(newDog)
        if err != nil {
            log.Fatalf("Unable to encode due to %s\n", err)
        }
    })

    log.Fatal(http.ListenAndServe(":3000", mux))
}
```

When you start the server and make a request to it, you will observe the following response:

```
{"breed":"Poodle","name":"Chloe","favorite_treat":"Dog Sticks","age":15}
```

## Third-party alternatives to encoding/json

While `encoding/json` is relatively dynamic and powerful, it is not the fastest JSON package out there. In performance critical situations, it might be worthwhile to consider a using a third-party package such as the ones shown below (not a comprehensive list):

- [fastjson](https://github.com/valyala/fastjson)
- [easyjson](https://github.com/mailru/easyjson)
- [jsonparser](https://github.com/buger/jsonparser)
- [gjson](https://github.com/tidwall/gjson)
- [json-iterator](https://github.com/json-iterator/go)

Each library has their pros and cons so be sure to investigate each one thoroughly before making a decision on what to use.

## An improved encoding/json implementation (experimental)

Work is currently being done on a [reimplementation of the encoding/json](https://github.com/go-json-experiment/json) package that aims to provide a more flexible, performant, and easy to use package for JSON access in Go. It eventually aims to be proposed for addition to the Go standard library, but it remains in the design and experimentation phase

There are many behavior changes introduced in this new implementation. Some of the key ones to note include:

- Unmarshalling will now be case-sensitive (it was case-insensitive name matching previously),
- `nil` slices will be marshalled as an empty JSON array (it currently produces `null`)
- `nil` maps will be marshalled as an empty JSON object (it currently produces `null`).
- To improve performance, JSONv2 no longer sorts the keys of a Go map.
- [and several more](https://github.com/go-json-experiment/json#behavior-changes).

Given that this is still at an experimental stage, it may not become a part of the standard library if it does not provide significant benefit over the existing `encoding/json` package. As such, do not depend on it until it has been officially added to the Go standard library.

## Final thoughts

To sum up, Go delivers solid capabilities for handling JSON data. With its `encoding/json` package, a diverse set of powerful tools is provided for encoding, decoding, marshalling, and unmarshalling JSON data within your Go applications.

Throughout this article, we have explored various aspects of JSON handling in Go, such as the fundamentals of encoding and decoding JSON data, employing structs for marshalling and unmarshalling purposes, and identifying common gotchas to avoid when working with JSON data in Go.

By mastering these concepts and adhering to the presented best practices, you will be well-prepared to manage JSON data in your Go applications, leading to more efficient, maintainable, and error-free code.

Thanks for reading!

**Further reading:**

- [Redis caching in Node.js](https://betterstack.com/community/guides/scaling-nodejs/nodejs-caching-redis/)
- [Redis caching in Golang](https://betterstack.com/community/guides/scaling-go/redis-caching-golang/)
- [Best Golang logging libraries](https://betterstack.com/community/guides/logging/best-golang-logging-libraries/)