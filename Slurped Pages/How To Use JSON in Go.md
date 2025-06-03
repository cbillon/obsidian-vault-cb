---
link: https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go
byline: Kristin Davidson
site: DigitalOcean
date: 2022-03-28T22:09
excerpt: Programs need to store data their own data and communicate with each other. JSON is a popular way to do both. In this tutorial, you will use Go’s encoding/json package to enode and decode JSON data so that you can save your own JSON data and interact with APIs.
slurped: 2025-05-29T09:29
title: How To Use JSON in Go
tags:
  - go
  - json
---

_The author selected the [Diversity in Tech Fund](https://www.brightfunds.org/funds/diversity-in-tech) to receive a donation as part of the [Write for DOnations](https://do.co/w4do-cta) program._

### [Introduction](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#introduction)

In modern programs, it’s important to communicate between one program and another. Whether it’s a [Go](https://www.digitalocean.com/community/tutorial_series/how-to-code-in-go) program checking if a user has access to another program, a [JavaScript](https://www.digitalocean.com/community/tutorials/what-is-javascript) program getting a list of past orders to display on a website, or a [Rust](https://en.wikipedia.org/wiki/Rust_\(programming_language\)) program reading test results from a file, programs need a way to provide other programs with data. However, many programming languages have their own way of storing data internally that other languages don’t understand. To allow these languages to interact, the data needs to be converted to a common format they can all understand. One of these formats, [JSON](https://www.digitalocean.com/community/tutorials/an-introduction-to-json), is a popular way to transmit data over the internet as well as between programs in the same system.

Many modern programming languages include a way to convert data to and from JSON in their standard libraries, and Go does as well. By using the [`encoding/json`](https://pkg.go.dev/encoding/json) package provided by Go, your Go programs will also be able to interact with any other system that can communicate using JSON.

In this tutorial, you will start by creating a program that uses the `encoding/json` package to encode data from a `map` into JSON data, then update your program to use a `struct` type to encode the data instead. After that, you will update your program to decode JSON data into a `map` before finally decoding the JSON data into a `struct` type.

## [Prerequisites](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#prerequisites)

To follow this tutorial, you will need:

- Go version 1.16 or greater installed. To set this up, follow the [How To Install Go](https://www.digitalocean.com/community/tutorials/how-to-install-go-on-ubuntu-20-04) tutorial for your operating system.
- Familiarity with JSON, which you can find in [An Introduction to JSON](https://www.digitalocean.com/community/tutorials/an-introduction-to-json).
- The ability to use Go struct tags to customize fields on `struct` types. More information can be found in [How To Use Struct Tags in Go](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go).
- Optionally, be aware of how to create date and time values in Go. You can read more in [How To Use Dates and Times in Go](https://www.digitalocean.com/community/tutorials/how-to-use-dates-and-times-in-go).

## [Using a Map to Generate JSON](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#using-a-map-to-generate-json)

Go’s support for encoding and decoding JSON is provided by the standard library’s `encoding/json` package. The first function you’ll use from that package is the [`json.Marshal`](https://pkg.go.dev/encoding/json#Marshal) function. [Marshalling](https://en.wikipedia.org/wiki/Marshalling_\(computer_science\)), sometimes also known as [serialization](https://en.wikipedia.org/wiki/Serialization), is the process of transforming program data in memory into a format that can be transmitted or saved elsewhere. The `json.Marshal` function, then, is used to convert Go data into JSON data. The `json.Marshal` function accepts an `interface{}` type as the value to marshal to JSON, so any value is allowed to be passed in as a parameter and will return the JSON data as a result. In this section, you will create a program using the `json.Marshal` function to generate JSON containing various types of data from Go `map` values, and then print those values to the output.

Most JSON is represented as an object, with a `string` key and various other types as values. Because of this, the most flexible way to generate JSON data in Go is by putting data into a `map` using `string` keys and `interface{}` values. The `string` key can be directly translated to a JSON object key, and the `interface{}` value allows the value to be any other value, whether it’s a `string`, an `int`, or even another `map[string]interface{}`.

To get started using the `encoding/json` package in a program, you’ll need to have a directory for the program. In this tutorial, you’ll use a directory named `projects`.

First, make the `projects` directory and navigate to it:

```
mkdir projects
cd projects

```

Next, make the directory for your project. In this case, use the directory `jsondata`:

```
mkdir jsondata
cd jsondata

```

Inside the `jsondata` directory use `nano`, or your favorite editor, to open the `main.go` file:

```
nano main.go

```

In the `main.go` file, you’ll add a `main` function to run your program. Next, you’ll add a `map[string]interface{}` value with various keys and types of data. Then, you’ll use the `json.Marshal` function to marshal the `map` data into JSON data.

Add the following lines to `main.go`:

main.go

```
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
	}

	jsonData, err := json.Marshal(data)
	if err != nil {
		fmt.Printf("could not marshal json: %s\n", err)
		return
	}

	fmt.Printf("json data: %s\n", jsonData)
}
```

You’ll see in the `data` variable that each value has a `string` as the key, but the values for those keys vary. One is an `int` value, another is a `bool` value, and one is even another `map[string]interface{}` value with a `[]int` value inside it.

When you pass the `data` variable to `json.Marshal`, the function will look through all the values you’ve provided and determine which type they are and how to represent them in JSON. If there are any problems in the translation, the `json.Marshal` function will return an `error` describing the issue. If the translation is successful, though, the `jsonData` variable will contain a `[]byte` of the marshalled JSON data. Since a `[]byte` value can be converted to a `string` value using `myString := string(jsonData)`, or the `%s` [verb](https://pkg.go.dev/fmt) in a format string, you can then print the JSON data to the screen using `fmt.Printf`.

Save and close the file.

To see the output of your program, use the `go run` command and provide the `main.go` file:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson data: {"boolValue":true,"intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

In the output, you’ll see that the top-level JSON value is an object represented by curly braces (`{}`) surrounding it. All of the values you included in `data` are present. You’ll also see the `objectValue`’s `map[string]interface{}` was translated into another JSON object surrounded by `{}`, and also includes `arrayValue` inside it with the array value of `[1,2,3,4]`.

### [Encoding Times in JSON](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#encoding-times-in-json)

The `encoding/json` package doesn’t just support types like `string` and `int` values, though. It can also encode more complex types. One of the more complex types it supports is the `time.Time` type from the [`time`](https://pkg.go.dev/time) package.

To see this in action, open your `main.go` file again and add a `time.Time` value to your data using the `time.Date` function:

main.go

```
package main

import (
	"encoding/json"
	"fmt"
	"time"
)

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
		"dateValue":   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
	}

	...
}
```

This update will assign the date `March 2, 2022,` and the time `9:10:00 AM` in the `UTC` time zone to the `dateValue` key.

Once you’ve saved your changes, run your program again with the same `go run` command as before:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson data: {"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

This time in the output, you’ll see a `dateValue` field in the JSON data with the time formatted using the [RFC 3339](https://datatracker.ietf.org/doc/html/rfc3339) format, a common format used to convey dates and times as `string` values.

### [Encoding `null` Values in JSON](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#encoding-null-values-in-json)

Depending on the systems your program interacts with, you may be required to send `null` values in your JSON data, and Go’s `encoding/json` package can handle that for you as well. Using a `map`, it’s only a matter of adding new `string` keys with a `nil` value.

To add a couple of `null` values to your JSON output, open your `main.go` file again and add the following lines:

main.go

```
...

func main() {
	data := map[string]interface{}{
		"intValue":    1234,
		"boolValue":   true,
		"stringValue": "hello!",
                "dateValue":   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		"objectValue": map[string]interface{}{
			"arrayValue": []int{1, 2, 3, 4},
		},
		"nullStringValue": nil,
		"nullIntValue":    nil,
	}

	...
}
```

The values you added to the data have keys that say it’s a `string` value or an `int` value, but there’s actually nothing in the code that is making it either of those values. Since the `map` has `interface{}` values, all the code knows is that the `interface{}` value is `nil`. Since you’re only using this `map` to convert from Go data to JSON data, the distinction at this point doesn’t make a difference.

Once you’ve saved your changes to `main.go`, run your program using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson data: {"boolValue":true,"dateValue":"2022-03-02T09:10:00Z","intValue":1234,"nullIntValue":null,"nullStringValue":null,"objectValue":{"arrayValue":[1,2,3,4]},"stringValue":"hello!"}
```

Now in the output, you’ll see the `nullIntValue` and `nullStringValue` fields are included with a JSON `null` value. This way you can still use a `map[string]interface{}` value to translate Go data into JSON data with the expected fields.

In this section, you created a program that can marshal a `map[string]interface{}` value into JSON data. Then, you added a `time.Time` field to the data and also included a pair of `null`-value fields.

While using a `map[string]interface{}` to marshal JSON data can be very flexible, it can also become a hassle if you need to send the same data in multiple places. If you copy this data to more than one place in your code, it can be easy to accidentally mistype a field name, or assign the incorrect data to a field. For times like these, it can be beneficial to use a `struct` type to represent the data you’re converting to JSON.

## [Using a Struct to Generate JSON](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#using-a-struct-to-generate-json)

One of the benefits of using a [statically typed](https://en.wikipedia.org/wiki/Type_system#Static_type_checking) language like Go is that you can use those types to let the compiler check for or enforce consistency in your programs. Go’s `encoding/json` package allows you to take advantage of this by defining a `struct` type to represent the JSON data. You can control how the data contained in the `struct` is translated using [struct tags](https://www.digitalocean.com/community/tutorials/how-to-use-struct-tags-in-go). In this section, you will update your program to use a `struct` type instead of a `map` type to generate your JSON data.

When you use a `struct` to define JSON data, the field names (not the `struct` type name itself) you expect to be translated must be exported, meaning they must start with a capital letter, such as `IntValue`, or the `encoding/json` package will not be able to access the fields to translate them to JSON. If you don’t use struct tags to control the naming of those fields, the field names will be translated directly as they are on the `struct`. Using the default names may be what you’d like in your JSON data, depending on how you’d like your data to be formed. If this is the case, you wouldn’t need to add any struct tags. However, many JSON consumers use name formats such as `intValue` or `int_value` for their field names, so adding these struct tags will allow you to control how that translation happens.

For example, say you had a `struct` with a field called `IntValue` that you marshalled to JSON:

```
type myInt struct {
	IntValue int
}

data := &myInt{IntValue: 1234}
```

If you marshalled the `data` variable to JSON using the `json.Marshal` function, you would end up with the following value:

```
{"IntValue":1234}
```

However, if your JSON consumer expects the field to be named `intValue` instead of `IntValue`, you’ll need a way to to tell `encoding/json`. Since `json.Marshal` doesn’t know what you expect the field to be named in the JSON data, you’ll tell it by adding a struct tag to the field. By adding a `json` struct tag to the `IntValue` field with a value of `intValue`, you tell `json.Marshal` it should use the name `intValue` when generating the JSON data:

```
type myInt struct {
	IntValue int `json:"intValue"`
}

data := &myInt{IntValue: 1234}
```

This time if you marshal the `data` variable to JSON, the `json.Marshal` function will see the `json` struct tag and know to name the field `intValue`, so you’ll get your expected result:

```
{"intValue":1234}
```

Now, you’ll update your program to use a `struct` value for your JSON data. You’ll add a `myJSON` `struct` type to define your top-level JSON object, as well as a `myObject` `struct` to define your inner JSON object for the `ObjectValue` field. You’ll also add a `json` struct tag to each of the fields to tell `json.Marshal` how to name them in the JSON data. You’ll also need to update the `data` variable assignment to use your `myJSON` struct, declaring it similar to how you would any other Go `struct`.

Open your `main.go` file and make the following changes:

main.go

```
...

type myJSON struct {
	IntValue        int       `json:"intValue"`
	BoolValue       bool      `json:"boolValue"`
	StringValue     string    `json:"stringValue"`
	DateValue       time.Time `json:"dateValue"`
	ObjectValue     *myObject `json:"objectValue"`
	NullStringValue *string   `json:"nullStringValue"`
	NullIntValue    *int      `json:"nullIntValue"`
}

type myObject struct {
	ArrayValue []int `json:"arrayValue"`
}

func main() {
	otherInt := 4321
	data := &myJSON{
		IntValue:    1234,
		BoolValue:   true,
		StringValue: "hello!",
		DateValue:   time.Date(2022, 3, 2, 9, 10, 0, 0, time.UTC),
		ObjectValue: &myObject{
			ArrayValue: []int{1, 2, 3, 4},
		},
		NullStringValue: nil,
		NullIntValue:    &otherInt,
	}

	...
}
```

Many of these changes are similar to the `IntValue` field name example from before, but some of the changes deserve to be called out specifically. One of them, the `ObjectValue` field, is using a reference type of `*myObject` to tell the JSON marshaller to expect either a reference to a `myObject` value or a `nil` value. This is how you can define a JSON object that is multiple layers of custom objects deep. If your JSON data required it, you could also have another `struct` type referenced inside the `myObject` type, and so on. Using this pattern, you can describe very complex JSON objects using Go `struct` types.

Another pair of fields to look at in the above code are `NullStringValue` and `NullIntValue`. Unlike `StringValue` and `IntValue`, the types of these values are reference types `*string` and `*int`. By default, `string` and `int` types cannot have a value of `nil` since their “empty” values are `""` and `0`. So if you want to represent a field that can be either one type or `nil`, you need to make it a reference. For example, imagine you have a user questionnaire and you want to be able to represent if a user chose not to answer the question (a `null` value), or the user didn’t have an answer to the question (a `""` value).

This code also updates the `NullIntValue` field to assign a value of `4321` to it to show how you might assign a value to a reference type such as `*int`. In Go you can only create references to [primitive types](https://en.wikipedia.org/wiki/Primitive_data_type), such as `int` and `string`, using variables. So, in order to assign a value to the `NullIntValue` field, you first assign the value to another variable, `otherInt`, and then get a reference to that using `&otherInt` (instead of doing `&4321` directly).

Once you have your updates saved, run your program using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson data: {"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullStringValue":null,"nullIntValue":4321}
```

You’ll see this output is the same as when you used a `map[string]interface{}` value, except this time `nullIntValue` has a value of `4321` because that’s the value of `otherInt`.

Initially, it may take some extra time to set up your `struct` values, but once you have them defined, you can use them over and over in your code, and the result will be the same no matter where you use them. You can also update them in one place instead of trying to find every place where a `map` may be used instead.

Go’s JSON marshaller also allows you to control whether a field should be included in the JSON output based on whether the value is empty or not. Sometimes you may have a large JSON object or optional fields you don’t want to be included all the time, so omitting those fields can be useful. Controlling whether a field is omitted when it’s empty or not is done via the `omitempty` option in the `json` struct tag.

Now, update your program to make the `NullStringValue` field `omitempty` and add a new field called `EmptyString` with the same option:

main.go

```
...

type myJSON struct {
	...
	
	NullStringValue *string   `json:"nullStringValue,omitempty"`
	NullIntValue    *int      `json:"nullIntValue"`
	EmptyString     string    `json:"emptyString,omitempty"`
}

...
```

Now, when `myJSON` is marshalled, both the `EmptyString` and `NullStringValue` fields will be excluded from the output if their values are empty.

After you’ve saved your changes, run your program using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson data: {"intValue":1234,"boolValue":true,"stringValue":"hello!","dateValue":"2022-03-02T09:10:00Z","objectValue":{"arrayValue":[1,2,3,4]},"nullIntValue":4321}
```

This time in the output, you’ll see the `nullStringValue` field no longer appears. Since it’s considered empty by having a `nil` value, the `omitempty` option excluded it from the output. You’ll also see the new `emptyString` field isn’t included either. Even though the `emptyString` value isn’t `nil`, the default `""` value for a string is considered empty so it was excluded as well.

In this section, you updated your program to use `struct` types to generate JSON data with `json.Marshal` instead of a `map` type. You also updated your program to omit empty fields from your JSON output.

In order for your programs to fit well into the JSON ecosystem, though, you need to do more than just generate JSON data. You’ll also need to be able to read JSON data being sent in response to your requests, or other systems sending requests to you. The `encoding/json` package also provides a way to decode JSON data into various Go types. In the next section, you’ll update your program to decode a JSON string into a Go `map` type.

## [Parsing JSON Using a Map](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#parsing-json-using-a-map)

Similar to the first section of this tutorial, where you used a `map[string]interface{}` as a flexible way to generate JSON data, you can also use it as a flexible way to read JSON data. The `json.Unmarshal` function, essentially the opposite of the `json.Marshal` function, will take JSON data and translate it back into Go data. You provide `json.Unmarshal` with the JSON data as well as the Go variable to put the unmarshalled data into and it will either return an `error` value if it’s unable to do it, or a `nil` error value if it succeeded. In this section, you will update your program to use the `json.Unmarshal` function to read JSON data from a pre-defined `string` value into a `map` variable. You will also update your program to print the Go data to the output.

Now, update your program to use `json.Unmarshal` to unmarshal JSON data to a `map[string]interface{}`. You’ll start by replacing your original `data` variable with a `jsonData` variable containing a JSON string. Then you’ll declare a new `data` variable as a `map[string]interface{}` to receive the JSON data. And finally, you’ll use `json.Unmarshal` with those variables to access the JSON data.

Open your `main.go` file and replace the lines in your `main` function with the following:

main.go

```
...

func main() {
	jsonData := `
		{
			"intValue":1234,
			"boolValue":true,
			"stringValue":"hello!",
			"dateValue":"2022-03-02T09:10:00Z",
			"objectValue":{
				"arrayValue":[1,2,3,4]
			},
			"nullStringValue":null,
			"nullIntValue":null
		}
	`

	var data map[string]interface{}
	err := json.Unmarshal([]byte(jsonData), &data)
	if err != nil {
		fmt.Printf("could not unmarshal json: %s\n", err)
		return
	}

	fmt.Printf("json map: %v\n", data)
}
```

In this update, the `jsonData` variable is being set using a [raw string literal](https://go.dev/ref/spec#String_literals) to allow the declaration to span multiple lines for easier reading. After declaring `data` as a `map[string]interface{}`, you pass `jsonData` and `data` to `json.Unmarshal` to unmarshal the JSON data into the `data` variable.

The `jsonData` variable is being passed to `json.Unmarshal` as a `[]byte` because the function requires a `[]byte` type and `jsonData` is initially defined as a `string` type. This works because a `string` in Go can be translated to a `[]byte`, and vice versa. The `data` variable is being passed as a reference because in order for `json.Unmarshal` to put data into the variable it needs to have a reference to where the variable is being stored in memory.

Finally, once the JSON data has been unmarshalled into the `data` variable, you print it to the screen using `fmt.Printf`.

To run your updated program, save your changes and run the program using `go run`:

```
go run main.go

```

The output will look similar to this:

```
Outputjson map: map[boolValue:true dateValue:2022-03-02T09:10:00Z intValue:1234 nullIntValue:<nil> nullStringValue:<nil> objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]
```

This time, your output shows the Go side of the JSON translation. You have a `map` value, with the various fields from the JSON data included. You’ll see that even the `null` fields from the JSON data show up in the map.

Now, because your Go data is in a `map[string]interface{}`, there’s a little bit of work that needs to go into using the data. You need to get the value from the `map` using the desired `string` key value, then you need to make sure the value you received is the one you were expecting because it’s returned to you as an `interface{}` value.

To do this, open the `main.go` file and update your program to read the `dateValue` field with the following code:

main.go

```
...

func main() {
	...
	
	fmt.Printf("json map: %v\n", data)

	rawDateValue, ok := data["dateValue"]
	if !ok {
		fmt.Printf("dateValue does not exist\n")
		return
	}
	dateValue, ok := rawDateValue.(string)
	if !ok {
		fmt.Printf("dateValue is not a string\n")
		return
	}
	fmt.Printf("date value: %s\n", dateValue)
}
```

In this update, you use `data["dateValue"]` to get the `rawDateValue` as an `interface{}` type, and use the `ok` variable to make sure the `dateValue` field is in the `map`.

Then, you use a [type assertion](https://go.dev/tour/methods/15) to assert the type of `rawDateValue` is actually a `string` value, and assign it to the variable `dateValue`. After that, you use the `ok` variable again to make sure the assertion succeeded.

Finally, you use `fmt.Printf` to print `dateValue`.

To run your updated progam again, save your changes and run it using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson map: map[boolValue:true dateValue:2022-03-02T09:10:00Z intValue:1234 nullIntValue:<nil> nullStringValue:<nil> objectValue:map[arrayValue:[1 2 3 4]] stringValue:hello!]
date value: 2022-03-02T09:10:00Z
```

You can see the `date value` line showing the `dateValue` field extracted from the `map` and converted to a `string` value.

In this section, you updated your program to use the `json.Unmarshal` function with a `map[string]interface{}` variable to unmarshal JSON data into Go data. Then, you updated the program to extract the value of `dateValue` from the Go data and print it to the screen.

However, this update does show one of the downsides of using a `map[string]interface{}` to unmarshal JSON in Go. Since Go doesn’t know which type of data each field is (the only thing it knows is it’s an `interface{}`), the best it can do to unmarshal the data is make a best guess. This means complex values like `time.Time` for the `dateValue` field can’t be unmarshaled for you and can only be accessed as a `string`. A similar problem happens if you try to access any number value in a `map` this way. Since `json.Unmarshal` doesn’t know whether the number should be an `int`, a `float`, an `int64`, and so on, the best guess it can make is to put it into the most flexible number type available, a `float64`.

While using a `map` to decode JSON data can be flexible, it also leaves more work for you when interpreting the data you have. Similar to how the `json.Marshal` function can use `struct` values to generate JSON data, the `json.Unmarshal` function can use `struct` values to read JSON data. This can help remove the type assertion complexities of using a `map` by using the type definitions on the `struct`’s fields to determine which types the JSON data should be interpreted as. In the next section, you will update your program to use `struct` types to remove these complexities.

## [Parsing JSON Using a Struct](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#parsing-json-using-a-struct)

When you’re reading JSON data, there’s a good chance you already know the structure of the data you’re receiving; otherwise, it would be difficult to interpret. You can use this knowledge of the structure to give Go some hints about what your data looks like and the type of data you’re expecting.

In a previous section, you defined the `myJSON` and `myObject` `struct` values and added `json` struct tags to let Go know how to name the fields when generating JSON. Now you can use those same `struct` values to decode the JSON string you’ve been using, which can be beneficial for reducing duplicated code in your program if you’re marshalling and unmarshalling the same JSON data. Another benefit of using a `struct` for unmarshalling JSON data is that you can tell Go the type of data expected for each field. Finally, you also benefit from using Go’s compiler to check that you’re using the correct names on fields instead of potentially missing a typo in the `string` values you’d be using with a `map` value.

Now, open your `main.go` file and update the `data` variable declaration to use a reference to the `myJSON` `struct`, and add a few `fmt.Printf` lines to show the data of various fields on `myJSON`:

main.go

```
...

func main() {
	...
	
	var data *myJSON
	err := json.Unmarshal([]byte(jsonData), &data)
	if err != nil {
		fmt.Printf("could not unmarshal json: %s\n", err)
		return
	}

	fmt.Printf("json struct: %#v\n", data)
	fmt.Printf("dateValue: %#v\n", data.DateValue)
	fmt.Printf("objectValue: %#v\n", data.ObjectValue)
}
```

Since you previously defined the `struct` types, you’ll only need to update the type of the `data` field to support unmarshalling to a `struct`. The rest of the updates show some of the data in the `struct` itself.

Now, save your updates and run your program using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson struct: &main.myJSON{IntValue:1234, BoolValue:true, StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC), ObjectValue:(*main.myObject)(0x1400011c180), NullStringValue:(*string)(nil), NullIntValue:(*int)(nil), EmptyString:""}
dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)
objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}
```

There are a couple of things to notice in the output this time. You’ll see in both the `json struct` line as well as the `dateValue` line that the date value in your JSON data has now been converted to a `time.Time` value (the `time.Date` format is what’s shown when `%#v` is used as the format verb). Since Go was able to see the `time.Time` type on `myJSON`’s `DateValue` field, it was also able to parse the `string` value for you as well.

The other thing to notice is that `EmptyString` shows up on the `json struct` line even though it wasn’t included in the original JSON data. If a field is included on a `struct` used for JSON unmarshalling and isn’t included in the JSON data being unmarshalled, that field is just set to the default value of its type and ignored. This way you can safely define all the possible fields your JSON data may have without worrying about getting an error if a field doesn’t exist on either side of the process. Both `NullStringValue` and `NullIntValue` are also set to their default value of `nil` because the JSON data said their values were `null`, but they would also be set to `nil` if those fields had been excluded from the JSON data.

Similar to how the `EmptyString` field on your `struct` was ignored by `json.Unmarshal` when the `emptyString` field was missing from the JSON data, the opposite is also true. If a field is included in the JSON data but doesn’t have a corresponding field on the Go `struct`, that JSON field is ignored and parsing continues on with the next JSON field. This way, if the JSON data you’re reading is very large and your program only cares about a small number of those fields, you can choose to create a `struct` that only includes the fields you care about. Any fields included in the JSON data that aren’t defined on the `struct` are simply ignored and Go’s JSON parser will continue on with the next field.

To see this in action, open up your `main.go` file one last time and update the `jsonData` to include a field that’s not included on `myJSON`:

main.go

```
...

func main() {
	jsonData := `
		{
			"intValue":1234,
			"boolValue":true,
			"stringValue":"hello!",
			"dateValue":"2022-03-02T09:10:00Z",
			"objectValue":{
				"arrayValue":[1,2,3,4]
			},
			"nullStringValue":null,
			"nullIntValue":null,
			"extraValue":4321
		}
	`

	...
}
```

Once you’ve added the JSON data, save your file and run it using `go run`:

```
go run main.go

```

Your output will look similar to this:

```
Outputjson struct: &main.myJSON{IntValue:1234, BoolValue:true, StringValue:"hello!", DateValue:time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC), ObjectValue:(*main.myObject)(0x14000126180), NullStringValue:(*string)(nil), NullIntValue:(*int)(nil), EmptyString:""}
dateValue: time.Date(2022, time.March, 2, 9, 10, 0, 0, time.UTC)
objectValue: &main.myObject{ArrayValue:[]int{1, 2, 3, 4}}
```

You shouldn’t see any difference between this output and the previous output because Go will have ignored the `extraValue` field in the JSON data and continued on.

In this section, you updated your program to use the `struct` types you previously defined to unmarshal your JSON data. You saw how Go was able to parse a `time.Time` value for you and ignore the `EmptyString` field defined on the `struct` type but not in the JSON data. You also added an additional field to the JSON data to see that Go will safely continue parsing the data even if you only define a subset of the fields in the JSON data.

## [Conclusion](https://www.digitalocean.com/community/tutorials/how-to-use-json-in-go#conclusion)

In this tutorial, you created a new program to use the `encoding/json` package in Go’s standard library. First, you used the `json.Marshal` function with a `map[string]interface{}` type to create JSON data in a flexible way. Then, you updated your program to use `struct` types with `json` struct tags to generate JSON data in a consistent and reliable way with `json.Marshal`. After that, you used the `json.Unmarshal` function with a `map[string]interface{}` type to decode a JSON string into Go data. Finally, you used the `struct` types you’d previously defined with the `json.Unmarshal` function to let Go do the parsing and type conversions for you based on those `struct` fields.

Using the `encoding/json` package, you’ll be able to interact with many of the APIs available on the internet to create your own integrations with popular web sites. You’ll also be able to convert Go data in your own programs into a format you can save and then load later to continue from where the program left off.

In addition to the functions you used in this tutorial, the [`encoding/json`](https://pkg.go.dev/encoding/json) package includes other useful functions and types that can be used for interacting with JSON. The [`json.MarshalIndent`](https://pkg.go.dev/encoding/json#MarshalIndent) function, for example, can be used to [pretty print](https://en.wikipedia.org/wiki/Prettyprint) JSON data for troubleshooting.

This tutorial is also part of the [DigitalOcean](https://www.digitalocean.com/) [How to Code in Go](app://obsidian.md/community/tutorial_series/how-to-code-in-go) series. The series covers a number of Go topics, from installing Go for the first time to how to use the language itself.