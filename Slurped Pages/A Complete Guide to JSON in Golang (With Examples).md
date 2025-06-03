---
link: https://www.sohamkamani.com/golang/json/
byline: Soham Kamani
excerpt: This post will describe how to marshal and unmarshal JSON in Go. We will learn how to convert from JSON raw data into Go types like structs, maps, and slices
slurped: 2025-05-29T09:16
title: A Complete Guide to JSON in Golang (With Examples)
tags:
  - json
  - go
---

Last updated on Sep 11, 2023

---

see also [easyjson](https://github.com/mailru/easyjson)
[json parser](https://github.com/buger/jsonparser)

In this post, we will learn how to work with JSON in Go, in the simplest way possible.

We will learn how to convert from JSON raw data (strings or bytes) into Go types like _structs_, _arrays_, and _slices_, as well as unstructured data like _maps_ and empty interfaces.

JSON is used as the de-facto standard for data serialization in many applications, ranging from [REST APIs](https://www.sohamkamani.com/golang/http-client/#sending-json-data) to configuration management. By the end of this post, you’ll get familiar with how to marshal (encode) and unmarshal (decode) JSON in Go.

## Unmarshaling (Parsing) Raw JSON Strings

The `Unmarshal` function provided by Go’s [JSON standard library](https://pkg.go.dev/encoding/json) lets us parse raw JSON data in the form of `[]byte` variables.

We can convert JSON strings into bytes and unmarshal the data into a variables address:

```
import "encoding/json"
//...

// ... 
myJsonString := `{"some":"json"}`

// `&myStoredVariable` is the address of the variable we want to store our
// parsed data in
json.Unmarshal([]byte(myJsonString), &myStoredVariable)
//...
```

Let’s look at the different variable types for `myStoredVariable`, and when you should use them.

There are two types of data you will encounter when working with JSON:

1. Structured data
2. Unstructured data

## Structured Data (Decoding JSON Into Structs)

“Structured data” refers to data where you know the format beforehand. For example, let’s say you have a bird object, where each bird has a `species` field and a `description` field :

```
{
  "species": "pigeon",
  "description": "likes to perch on rocks"
}
```

To work with this kind of data, create a `struct` that mirrors the data you want to parse. In our case, we will create a bird struct which has a `Species` and `Description` attribute:

```
type Bird struct {
  Species string
  Description string
}
```

And unmarshal it as follows:

```
birdJson := `{"species": "pigeon","description": "likes to perch on rocks"}`
var bird Bird	
json.Unmarshal([]byte(birdJson), &bird)
fmt.Printf("Species: %s, Description: %s", bird.Species, bird.Description)
//Species: pigeon, Description: likes to perch on rocks
```

[Try it here](https://go.dev/play/p/DtA6sEppLO)

> By convention, Go uses the same title cased attribute names as are present in the case insensitive JSON properties. So the `Species` attribute in our `Bird` struct will map to the `species`, or `Species` or `sPeCiEs` JSON property.

### JSON Arrays

Let’s look at how we can decode an array of objects, like below:

```
[
  {
    "species": "pigeon",
    "description": "likes to perch on rocks"
  },
  {
    "species":"eagle",
    "description":"bird of prey"
  }
]
```

Since each element of the array has the structure of the `Bird` struct, you can unmarshal it by creating a slice of birds :

```
birdJson := `[{"species":"pigeon","description":"likes to perch on rocks"},{"species":"eagle","description":"bird of prey"}]`
var birds []Bird
json.Unmarshal([]byte(birdJson), &birds)
fmt.Printf("Birds : %+v", birds)
//Birds : [{Species:pigeon Description:} {Species:eagle Description:bird of prey}]
```

[Try it here](https://go.dev/play/p/thoUdxxmMa)

### Nested Objects

Now, consider the case when you have a property called `Dimensions`, that measures the `Height` and `Length` of the bird in question:

```
{
  "species": "pigeon",
  "description": "likes to perch on rocks"
  "dimensions": {
    "height": 24,
    "width": 10
  }
}
```

As with our previous examples, we need to mirror the structure of the object in our Go code. To add a nested `dimensions` object, lets create a `dimensions` struct :

```
type Dimensions struct {
  Height int
  Width int
}
```

Now, the `Bird` struct will include a `Dimensions` field:

```
type Bird struct {
  Species string
  Description string
  Dimensions Dimensions
}
```

We can unmarshal this data using the same method as before:

```
birdJson := `{"species":"pigeon","description":"likes to perch on rocks", "dimensions":{"height":24,"width":10}}`
var birds Bird
json.Unmarshal([]byte(birdJson), &birds)
fmt.Printf(bird)
// {pigeon likes to perch on rocks {24 10}}
```

[Try it here](https://go.dev/play/p/zOUMUNH4w9)

### Primitive Types

We mostly deal with complex objects or arrays when working with JSON, but data like `3`, `3.1412` and `"birds"` are also valid JSON strings.

We can unmarshal these values to their corresponding data type in Go by using primitive types:

```
numberJson := "3"
floatJson := "3.1412"
stringJson := `"bird"`

var n int
var pi float64
var str string

json.Unmarshal([]byte(numberJson), &n)
fmt.Println(n)
// 3

json.Unmarshal([]byte(floatJson), &pi)
fmt.Println(pi)
// 3.1412

json.Unmarshal([]byte(stringJson), &str)
fmt.Println(str)
// bird
```

[Try it out](https://go.dev/play/p/usCx_5oESBd)

### Time Values

Did you know that if you try to decode an ISO 8601 date string like `2021-10-18T11:08:47.577Z` into a `time.Time` struct, it will work out of the box?

```
dateJson := `"2021-10-18T11:08:47.577Z"`
var date time.Time
json.Unmarshal([]byte(dateJson), &date)

fmt.Println(date)
// 2021-10-18 11:08:47.577 +0000 UTC
```

[Try it out](https://go.dev/play/p/bumwZXRaOaz)

Here, `dateJson` is a JSON string type, but when we unmarshal it into a `time.Time` variable, it is able to understand the JSON data on its own. Well, this is because the `time.Time` struct has a custom [UnmarshalJSON](https://pkg.go.dev/time#Time.UnmarshalJSON) method that handles this case.

This will even work if the `time.Time` type is embedded within another struct:

```
type Bird struct {
	Species     string
	Description string
	CreatedAt   time.Time
}

func main() {
	birdJson := `{"species": "pigeon","description": "likes to perch on rocks", "createdAt": "2021-10-18T11:08:47.577Z"}`
	var bird Bird
	json.Unmarshal([]byte(birdJson), &bird)
	fmt.Println(bird)
	// {pigeon likes to perch on rocks 2021-10-18 11:08:47.577 +0000 UTC}
}
```

### Custom Parsing Logic

Similar to the `time.Time` struct, we can also create custom types that implement the [Unmarshaler](https://pkg.go.dev/encoding/json#Unmarshaler) interface. This will allow us to define custom logic for decoding JSON data into our custom types.

To illustrate this, let’s take the [nested dimension example](https://www.sohamkamani.com/golang/json/#nested-objects) from before. Suppose we receive the `dimensions` data as a formatted string:

```
{
  "species": "pigeon",
  "description": "likes to perch on rocks",
  "dimensions": "24x10"
}
```

We can modify the `Dimensions` type to implement the `Unmarshaler` interface, which will have custom parsing logic for our data:

```
type Dimensions struct {
	Height int
	Width  int
}

// unmarshals a JSON string with format
// "heightxwidth" into a Dimensions struct
func (d *Dimensions) UnmarshalJSON(data []byte) error {
	// the "data" parameter is expected to be JSON string as a byte slice
	// for example, `"20x30"`

	if len(data) < 2 {
		return fmt.Errorf("dimensions string too short")
	}
	// remove the quotes
	s := string(data)[1 : len(data)-1]
	// split the string into its two parts
	parts := strings.Split(s, "x")
	if len(parts) != 2 {
		return fmt.Errorf("dimensions string must contain two parts")
	}
	// convert the two parts into ints
	height, err := strconv.Atoi(parts[0])
	if err != nil {
		return fmt.Errorf("dimension height must be an int")
	}
	width, err := strconv.Atoi(parts[1])
	if err != nil {
		return fmt.Errorf("dimension width must be an int")
	}
	// assign the two ints to the Dimensions struct
	d.Height = height
	d.Width = width
	return nil
}
```

Now, if we try to unmarshal the JSON data, it will create a `Dimensions` struct with the correct values:

```
birdJson := `{"species": "pigeon","description": "likes to perch on rocks", "dimensions":"20x30"}`
var bird Bird
json.Unmarshal([]byte(birdJson), &bird)
fmt.Printf("%+v", bird)
// {Species:pigeon Description:likes to perch on rocks Dimensions:{Height:20 Width:30}}
```

We saw earlier that Go uses convention to determine the attribute name for mapping JSON properties.

Although sometimes, we want a different attribute name than the one provided in your JSON data. For example, consider the below data:

```
{
  "birdType": "pigeon",
  "what it does": "likes to perch on rocks"
}
```

Here, we would prefer `birdType` to remain as the `Species` attribute in our Go code. It is also not possible for us to provide a suitable attribute name for a key like `"what it does"`.

To solve this, we can use struct field tags:

```
type Bird struct {
  Species string `json:"birdType"`
  Description string `json:"what it does"`
}
```

Now, we can explicitly tell our code which JSON property to map to which attribute.

```
birdJson := `{"birdType": "pigeon","what it does": "likes to perch on rocks"}`
var bird Bird
json.Unmarshal([]byte(birdJson), &bird)
fmt.Println(bird)
// {pigeon likes to perch on rocks}
```

[Try it here](https://go.dev/play/p/-_0XddCakR)

## Decoding JSON to Maps - Unstructured Data

If you don’t know the structure of your JSON properties beforehand, you cannot use structs to unmarshal your data.

Instead you can use [maps](https://www.sohamkamani.com/golang/maps/). Consider some JSON of the form:

```
{
  "birds": {
    "pigeon":"likes to perch on rocks",
    "eagle":"bird of prey"
  },
  "animals": "none"
}
```

There is no struct we can build to represent the above data for all cases since the keys corresponding to the birds can change, which will change the structure.

To deal with this case we create a map of strings to empty interfaces:

```
birdJson := `{"birds":{"pigeon":"likes to perch on rocks","eagle":"bird of prey"},"animals":"none"}`
var result map[string]any
json.Unmarshal([]byte(birdJson), &result)

// The object stored in the "birds" key is also stored as 
// a map[string]any type, and its type is asserted from
// the `any` type
birds := result["birds"].(map[string]any)

for key, value := range birds {
  // Each value is an `any` type, that is type asserted as a string
  fmt.Println(key, value.(string))
}
```

[Try it here](https://go.dev/play/p/21STGFhcd6W)

Each string corresponds to a JSON property, and its mapped `any` type corresponds to the value, which can be of any type. We then use [type assertions](https://www.sohamkamani.com/golang/type-assertions-vs-type-conversions/#type-assertions) to convert this `any` type into its actual type.

These maps can be iterated over, so an unknown number of keys can be handled by a simple for loop.

## Validating JSON Data

In real-world applications, we may sometimes get invalid (or incomplete) JSON data. Let’s see an example where some of the data is cut off, and the resulting JSON string is invalid:

```
{
  "birds": {
    "pigeon":"likes to perch on rocks",
    "eagle":"bird of prey"
```

> In actual applications, this may happen due to network errors or incomplete data written to files

If we try to unmarshal this, our code will panic:

```
birdJson := `{"birds":{"pigeon":"likes to perch on rocks","eagle":"bird of prey"`
var result map[string]any
json.Unmarshal([]byte(birdJson), &result)
```

Output:

```
panic: interface conversion: interface {} is nil, not map[string]interface {}
```

[Try it here](https://go.dev/play/p/t5x-i1HGVBr)

We can, of course handle the [panic and recover](https://www.sohamkamani.com/golang/defer/#using-defer-with-panic) from our code, but this would not be idiomatic or readable.

Instead, we can use the [json.Valid](https://pkg.go.dev/encoding/json#Valid) function to check the validity of our JSON data:

```
if !json.Valid([]byte(birdJson)) {
	// handle the error here
	fmt.Println("invalid JSON string:", birdJson)
	return
}
```

Now, our code will return early and give the output:

```
invalid JSON string: {"birds":{"pigeon":"likes to perch on rocks","eagle":"bird of prey"
```

[Try it here](https://go.dev/play/p/GssAAdMHc2B)

## Marshaling JSON Data

Marshaling is the process of transforming structured data into a serializable JSON string. Similar to unmarshaling, we can marshal data into structs, maps, and slices.

### Marshaling Structured Data

Let’s consider our `Bird` struct from before, and see the code required to generate a JSON string from a variable of its type:

```
package main

import (
	"encoding/json"
	"fmt"
)

// The same json tags will be used to encode data into JSON
type Bird struct {
	Species     string `json:"birdType"`
	Description string `json:"what it does"`
}

func main() {
	pigeon := &Bird{
		Species:     "Pigeon",
		Description: "likes to eat seed",
	}

	// we can use the json.Marshal function to
	// encode the pigeon variable to a JSON string
	data, _ := json.Marshal(pigeon)
	// data is the JSON string represented as bytes
	// the second parameter here is the error, which we
	// are ignoring for now, but which you should ideally handle
	// in production grade code

	// to print the data, we can typecast it to a string
	fmt.Println(string(data))
}
```

This will give the output:

```
{"birdType":"Pigeon","what it does":"likes to eat seed"}
```

[Try it out](https://go.dev/play/p/jHGzfy-hEjT)

### Ignoring Empty Fields

In some cases, we would want to ignore a field in our JSON output, if its value is empty. We can use the [“omitempty” property](https://www.sohamkamani.com/golang/omitempty/) for this purpose.

For example, if the `Description` field is missing for the `pigeon` object, the key will not appear in the encoded JSON string incase we set this property:

```
package main

import (
	"encoding/json"
	"fmt"
)

type Bird struct {
	Species     string `json:"birdType"`
	// we can set the "omitempty" property as part of the JSON tag
	Description string `json:"what it does,omitempty"`
}

func main() {
	pigeon := &Bird{
		Species:     "Pigeon",
	}

	data, _ := json.Marshal(pigeon)

	fmt.Println(string(data))
}
```

This will give us the output:

```
{"birdType":"Pigeon"}
```

[Try it out](https://go.dev/play/p/jOihnSg8dGB)

If we want to always ignore a field, we can use the `json:"-"` struct tag to denote that we never want this field included:

```
package main

import (
	"encoding/json"
	"fmt"
)

type Bird struct {
	Species     string `json:"-"`
}

func main() {
	pigeon := &Bird{
		Species:     "Pigeon",
	}

	data, _ := json.Marshal(pigeon)

	fmt.Println(string(data))
}
```

This code will always print an empty JSON object:

```
{}
```

[Try it out](https://go.dev/play/p/91-Nx0aV31G)

### Marshaling Slices

This isn’t much different from structs. We just need to pass the slice or array to the `json.Marshal` function, and it will encode data like you expect:

```
pigeon := &Bird{
  Species:     "Pigeon",
  Description: "likes to eat seed",
}

// Now we pass a slice of two pigeons
data, _ := json.Marshal([]*Bird{pigeon, pigeon})
fmt.Println(string(data))
```

This will give the output:

```
[{"birdType":"Pigeon","what it does":"likes to eat seed"},{"birdType":"Pigeon","what it does":"likes to eat seed"}]
```

[Try it out](https://go.dev/play/p/3a5lWlKSvPS)

### Marshaling Maps

We can use [maps](https://www.sohamkamani.com/golang/maps/) to encode unstructured data.

The keys of the map need to be strings, or a type that can convert to strings. The values can be any serializable type.

```
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	// The keys need to be strings, the values can be
	// any serializable value
	birdData := map[string]any{
		"birdSounds": map[string]string{
			"pigeon": "coo",
			"eagle":  "squawk",
		},
		"total birds": 2,
	}

	// JSON encoding is done the same way as before	
	data, _ := json.Marshal(birdData)
	fmt.Println(string(data))
}
```

Output ([Try it out](https://go.dev/play/p/xpvgujF6p0l)):

```
{"birdSounds":{"eagle":"squawk","pigeon":"coo"},"total birds":2}
```

### Encoding “Null” Values

Sometimes, we may want to send a `null` value in our JSON data. Any `nil` values in our Go code will be encoded as `null` in the JSON string. But this means that any nullable fields in our struct must be pointers, or any other type that can be `nil`.

For example, consider the case where we want to send a `null` value for the `Description` field of our `Bird` struct. We can do this by using a pointer to the `Description` field:

```
type Bird struct {
	Species     string
	Description *string
}

func main() {
	pigeon := &Bird{
		Species:     "Pigeon",
		Description: nil,
	}

	data, _ := json.Marshal(pigeon)
	fmt.Println(string(data))
	// {"Species":"Pigeon","Description":null}
}
```

[Try it out](https://go.dev/play/p/lOKjkj8ofJ0)

When using map types, we can also use the `nil` value for any key to encode `null` values:

```
birdData := map[string]any{
	"total birds": 2,
	"nullValue":   nil,
}

data, _ = json.Marshal(birdData)
fmt.Println(string(data))
// {"nullValue":null,"total birds":2}
```

### Custom Encoding Logic

Similar to [custom decoding logic](https://www.sohamkamani.com/golang/json/#custom-parsing-logic), we can also create custom types that implement the [Marshaler](https://pkg.go.dev/encoding/json#Marshaler) interface. This will allow us to define custom logic for encoding our custom types into JSON data.

To illustrate this, let’s take the [nested dimension example](https://www.sohamkamani.com/golang/json/#nested-objects) from before. Suppose we want to encode the `dimensions` data as a formatted string `"24x10"`

We can modify the `Dimensions` type to implement the `Marshaler` interface, which will have custom encoding logic for our data:

```
type Dimensions struct {
	Height int
	Width  int
}

// marshals a Dimensions struct into a JSON string
// with format "heightxwidth"
func (d Dimensions) MarshalJSON() ([]byte, error) {
	return []byte(fmt.Sprintf(`"%dx%d"`, d.Height, d.Width)), nil
}

func main() {
	bird := Bird{
		Species: "pigeon",
		Dimensions: Dimensions{
			Height: 24,
			Width:  10,
		},
	}
	birdJson, _ := json.Marshal(bird)
	fmt.Println(string(birdJson))
	// {"Species":"pigeon","Dimensions":"24x10"}
}
```

[Try it out](https://go.dev/play/p/ajGLYpSY7rX)

## Printing Formatted (Pretty-Printed) JSON

By default, the JSON encoder will not add any whitespace to the encoded JSON string. This is done to reduce the size of the JSON string, and is useful when sending data over the network.

But, if you want to print the JSON string to the console, or write it to a file, you may want to add whitespace to make it more readable. We can do this by using the [json.MarshalIndent](https://pkg.go.dev/encoding/json#MarshalIndent) function:

```
bird := Bird{
	Species: "pigeon",
	Description: "likes to eat seed",
}

// The second parameter is the prefix of each line, and the third parameter
// is the indentation to use for each level
data, _ := json.MarshalIndent(bird, "", "  ")
fmt.Println(string(data))
```

The output in this case will be a formatted JSON string, instead of the compressed one-liner response that we’ve been getting so far:

```
{
	"Species": "pigeon",
	"Description": "likes to eat seed"
}
```

## Best Practices (Structs vs Maps)

As a general rule of thumb, if you _can_ use structs to represent your JSON data, you should use them. The only good reason to use maps would be if it were not possible to use structs due to the uncertain nature of the keys or values in the data.

If we use maps, we will either need each of the keys to have the same data type, or use a generic type and convert it later.