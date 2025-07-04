---
link: https://www.baeldung.com/linux/jq-command-json
site: Baeldung on Linux
date: 2019-11-07T23:16
excerpt: Explore the capabilities that jq provides for processing and
  manipulating JSON via the command line.
twitter: https://twitter.com/@CookyBear
slurped: 2024-10-30T14:58
title: Guide to Linux jq Command for JSON Processing | Baeldung on Linux
---

## **1. Overview**

[JSON](https://www.baeldung.com/java-json) is a widely employed structured data format typically used in most modern APIs and data services. It’s particularly popular in web applications due to its lightweight nature and compatibility with JavaScript.

**Unfortunately, shells such as Bash can’t interpret and work with JSON directly.** This means that working with JSON via the command line can be cumbersome, involving text manipulation using a combination of tools such as _[sed](http://man7.org/linux/man-pages/man1/sed.1.html)_ and _[grep](https://www.baeldung.com/linux/common-text-search)_.

In this tutorial, **we’ll look at how we can alleviate this awkwardness using _[jq](http://stedolan.github.io/jq/)_ — an eloquent command-line processor for JSON.**

## **2. Installation**

Let’s begin by [installing](https://github.com/stedolan/jq/wiki/Installation) _jq_, which is [available](https://stedolan.github.io/jq/download/) in most operating system packaging repositories. **It’s also possible to download the binary directly or build it from the source.**

Once we’ve installed the package, let’s verify the installation by running _jq_:

```
$ jq
jq - commandline JSON processor [version 1.6]

Usage:	jq [options] <jq filter> [file...]
	jq [options] --args <jq filter> [strings...]
	jq [options] --jsonargs <jq filter> [JSON_TEXTS...]
...
```

If the installation was successful, we’ll see the version, some usage examples and other information displayed in the console.

## **3. Working With Simple Filters**

**_jq_ is built around the concept of filters that work over a stream of JSON.** Each filter takes an input and emits JSON to standard out. As we’re going to see, there are many predefined filters that we can use. We can effortlessly combine these filters using pipes to quickly construct and apply complex operations and transformations to our JSON data.

### 3.1. Prettify JSON

Let’s start by taking a look at the simplest filter of all, which incidentally is one of the most useful and frequently used features of _jq_:

```
$ echo '{"fruit":{"name":"apple","color":"green","price":1.20}}' | jq '.'
```

We _[echo](http://man7.org/linux/man-pages/man1/echo.1.html)_ a simple JSON string and pipe it into our _jq_ command. Then we use the identity filter ‘.’ that takes the input and produces it unchanged as output with the caveat that by default _jq_ pretty-prints all output.

This gives us the following output:

```
{
  "fruit": {
    "name": "apple",
    "color": "green",
    "price": 1.2
  }
}
```

We can also apply this filter directly to a JSON file:

```
$ jq '.' fruit.json
```

**Being able to prettify JSON is particularly useful when we want to retrieve data from an API and see the response in a clear, readable format.**

Let’s hit a simple API using [_curl_](https://curl.haxx.se/) to see this in practice:

```
$ curl http://api.open-notify.org/iss-now.json | jq '.'
```

This gives us a JSON response for the current position of the International Space Station:

```
{
  "message": "success",
  "timestamp": 1572386230,
  "iss_position": {
    "longitude": "-35.4232",
    "latitude": "-51.3109"
  }
}
```

### 3.2. Accessing Properties

**We can access property values by using another simple filter: the _.field_ operator.** To find a property value, we simply combine this filter followed by the property name.

Let’s see this by building on our simple fruit example:

```
$ jq '.fruit' fruit.json
```

Here, we are accessing the fruit property, which gives us all the children of this key:

```
{
  "name": "apple",
  "color": "green",
  "price": 1.2
}
```

We can also chain property values together, allowing us to access nested objects:

```
$ jq '.fruit.color' fruit.json
```

As expected, this simply returns the color of our fruit:

```
"green"
```

If we need to retrieve multiple keys, we can separate them using a comma:

```
$ jq '.fruit.color,.fruit.price' fruit.json
```

This results in an output containing both property values:

```
"green"
1.2
```

**Note that if one of the properties has spaces or special characters, we need to wrap the property name in quotes when accessing it from the _jq_ command**:

```
$ echo '{ "with space": "hello" }' | jq '."with space"'
```

## **4. JSON Arrays**

**Let’s now look at how we can work with arrays in JSON data.** We typically use arrays to represent a list of items. And as in many programming languages, we use square brackets to denote the start and end of an array.

### 4.1. Iteration

We’ll start with a basic example to demonstrate how to iterate over an array:

```
$ echo '["x","y","z"]' | jq '.[]'
```

Here, we see the object value iterator operator_.[]_ in use, which will print out each item in the array on a separate line:

```
"x"
"y"
"z"
```

Now let’s imagine we want to represent a list of fruit in a JSON document:

```
[
  {
    "name": "apple",
    "color": "green",
    "price": 1.2
  },
  {
    "name": "banana",
    "color": "yellow",
    "price": 0.5
  },
  {
    "name": "kiwi",
    "color": "green",
    "price": 1.25
  }
]
```

Each item in the array is an object that represents a fruit.

**Let’s see how to extract the name of each fruit from each object in the array**:

```
$ jq '.[] | .name' fruits.json
```

First, we iterate over the array using _.[]_. Then we can pass each object in the array to the next filter in the command using a pipe |. The last step is to output the name field from each object using _.name_:

```
"apple"
"banana"
"kiwi"
```

**We can also use a slightly more concise version and access the property directly on each object in the array**:

```
$ jq '.[].name' fruits.json
```

### 4.2. Accessing by Index

Of course, as with all arrays, we can access one of the items in the array directly by passing the index:

```
$ jq '.[1].price' fruits.json
```

### 4.3. Slicing

**Finally, _jq_ also supports slicing of arrays, another powerful feature.** This is particularly useful when we need to return a subarray of an array.

Again, let’s see this using a simple array of numbers:

```
$ echo '[1,2,3,4,5,6,7,8,9,10]' | jq '.[6:9]'
```

The result will be a new array with a length of 3, containing the elements from index 6 (inclusive) to index 9 (exclusive):

```
[
  7,
  8,
  9
]
```

It’s also possible to omit one of the indexes when using the slicing functionality:

```
$ echo '[1,2,3,4,5,6,7,8,9,10]' | jq '.[:6]' | jq '.[-2:]'
```

Since we specified only the second argument in _.[:6]_, the slice will start from the beginning of the array and run up until index 6. It’s the same as doing _.[0:6]_.

**The second slicing operation has a negative argument, which denotes in this case that it counts backward from the end of the array.**

Note the subtle difference in the second slice — we pass the index as the first argument. **This means we will start two indexes from the end (-2), and since the second argument is empty, it will run until the end of the array.**

This gives us the following output:

```
[
  5,
  6
]
```

## 5. Using Functions

_jq_ has many powerful built-in functions that we can use to perform a variety of useful operations. Let’s take a look at some of them now.

### 5.1. Getting Keys

**Sometimes, we may want to get the keys of an object as an array instead of the values.**

We can do this using the _keys_ function:

```
$ jq '.fruit | keys' fruit.json
```

This gives us the keys sorted alphabetically:

```
[
  "color",
  "name",
  "price"
]
```

### 5.2. Returning the Length

Another handy function for arrays and objects is the _length_ function.

**We can use this function to return the array’s length or the number of properties on an object**:

```
$ jq '.fruit | length' fruit.json
```

Here, we get “3” since the fruit object has three properties.

**We can even use the _length_ function on string values as well**:

```
$ jq '.fruit.name | length' fruit.json
```

We would see “5” as the resulting output since the fruit name property has five characters: “apple”.

### 5.3. Mapping Values

**The _map_ function is a powerful function we can use to apply a filter or function to an array**:

```
$ jq 'map(has("name"))' fruits.json
```

**In this example, we’re applying the _has_ function to each item in the array and looking to see if there is a name property.** In our simple fruits JSON, we get _true_ in each result item.

**We can also use the _map_ function to apply operations to the elements in an array.**

Let’s imagine we want to increase the price of each fruit:

```
$ jq 'map(.price+2)' fruits.json
```

This gives us a new array with each price incremented:

```
[
  3.2,
  2.5,
  3.25
]
```

### 5.4. Min and Max

**If we need to find the minimum or maximum element of an input array, we can utilize the _min_ and _max_ functions**:

```
$ jq '[.[].price] | min' fruits.json
```

Likewise, we can also find the most expensive fruit in our JSON document:

```
$ jq '[.[].price] | max' fruits.json
```

Note that in these two examples, we’ve constructed a new array, using _[]_ around the array iteration. **This contains only the prices before we pass this new list to the _min_ or _max_ function.**

### 5.5. Selecting Values

**The _select_ function is another impressive utility that we can use for querying JSON.**

**We can think of it as a bit like a simple version of XPath for JSON**:

```
$ jq '.[] | select(.price>0.5)' fruits.json
```

This selects all the fruit with a price greater than 0.5.

**Likewise, we can also make selections based on the value of a property**:

```
$ jq '.[] | select(.color=="yellow")' fruits.json
```

We can even combine conditions to build up complex selections:

```
$ jq '.[] | select(.color=="yellow" and .price>=0.5)' fruits.json
```

This will give us all yellow fruit matching a given price condition:

```
{
  "name": "banana",
  "color": "yellow",
  "price": 0.5
}
```

Additionally, we can extract values of attributes based on keys containing a specific string:

```
$ jq '.[] | to_entries[] | select(.key | startswith("name")) | .value' fruits.json
"apple"
"banana"
"kiwi"
```

In this example, we select the values of attributes in the _fruits.json_ file that contain keys starting with the string _“name”_.

Here, we use the _to_entries[]_ function to convert each object in the array into an array of key-value pairs. Next, _select(.key | startswith(“name”))_ filters the key-value pairs, and selects only those where the key starts with the string _“name”_. Finally, _.value_ extracts the value from the filtered key-value pairs.

### 5.6. Support for Regular Expressions

**Next, we’re going to look at the _test_ function, which enables us to test if an input matches against a given regular expression**:

```
$ jq '.[] | select(.name|test("^a.")) | .price' fruits.json
```

Simply put, we want to output the price of all the fruit whose name starts with the letter “a”.

### 5.7. Finding Unique Values

**One common use case is to be able to see unique occurrences of a particular value within an array or remove duplicates.**

Let’s see how many unique colors we have in our fruits JSON document:

```
$ jq 'map(.color) | unique' fruits.json
```

We use the _map_ function to create a new array containing only colors. **Then we pass each color in the new array to the _unique_ function using a pipe |.**

This gives us an array with two distinct fruit colors:

```
[
  "green",
  "yellow"
]
```

### 5.8. Deleting Keys From JSON

**We’ll also sometimes want to remove a key and corresponding value from JSON objects.**

For this, _jq_ provides the _del_ function:

```
$ jq 'del(.fruit.name)' fruit.json
```

This outputs the fruit object without the deleted key:

```
{
  "fruit": {
    "color": "green",
    "price": 1.2
  }
}
```

## **6. Transforming JSON**

Frequently when working with data structures such as JSON, we might want to transform one data structure into another. **This can be useful while working with large JSON structures when we are only interested in several properties or values.**

In this example, we’ll use some Wikipedia JSON that describes a list of page entries:

```
{
  "query": {
    "pages": [
      {
        "21721040": {
          "pageid": 21721040,
          "ns": 0,
          "title": "Stack Overflow",
          "extract": "Some interesting text about Stack Overflow"
        }
      },
      {
        "21721041": {
          "pageid": 21721041,
          "ns": 0,
          "title": "Baeldung",
          "extract": "A great place to learn about Java"
        }
      }
    ]
  }
}
```

**We’re only really interested in the title and extract of each page entry.**

So, let’s see how we can transform this document:

```
$ jq '.query.pages | [.[] | map(.) | .[] | {page_title: .title, page_description: .extract}]' wikipedia.json
```

We’ll take a look at the command in more detail to understand it properly:

- First, we begin by accessing the pages array and passing that array into the next filter in the command via a pipe.
- Then we iterate over this array and pass each object inside the pages array to the _map_ function, where we simply create a new array with the contents of each object.
- **Next, we iterate over this array and for each item create an object containing the two keys _page_title_ and _page_description_.**
- The _.title_ and _.extract_ references are used to populate the two new keys.

This gives us a new, lean JSON structure:

```
[
  {
    "page_title": "Stack Overflow",
    "page_description": "Some interesting text about Stack Overflow"
  },
  {
    "page_title": "Baeldung",
    "page_description": "A great place to learn about Java"
  }
]
```

## **7. Conclusion**

**In this in-depth article, we covered some of the basic capabilities that _jq_ provides for processing and manipulating JSON via the command line.**

First, we looked at some of the essential filters _jq_ offers and how they can be used as the building blocks for more complex operations. Then we saw how to use a number of built-in functions that come bundled with _jq_.

We concluded with a complex example showing how to transform one JSON document into another.

**Of course, we can check out the excellent [cookbook](https://github.com/stedolan/jq/wiki/Cookbook) for more interesting examples**. Also, the full source code of the article is available [over on GitHub](https://github.com/eugenp/tutorials/tree/master/linux-bash-modules/linux-bash-json).