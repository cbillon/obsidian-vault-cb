---
link: https://dashjoin.medium.com/jq-vs-jsonata-language-and-tooling-compared-5f0f7acc778e
byline: Andreas Eberhart
site: Medium
date: 2023-08-08T13:39
excerpt: "JQ vs. JSONata: Language and Tooling Compared JQ refers to both a functional programming language and a command line tool for processing JSON data. The jq website describes jq as “sed for …"
twitter: https://twitter.com/@dashjoin
slurped: 2026-03-28T09:59
title: "JQ vs. JSONata: Language and Tooling Compared"
tags:
  - json
  - jq
  - jsonata
---

JQ refers to both a [functional programming language](https://en.wikipedia.org/wiki/Jq_\(programming_language\)) and a [command line tool](https://jqlang.github.io/jq/) for processing JSON data. The jq website describes jq as “sed for JSONata”. It was designed by Stephen Dolan in 2012 and is very popular in the data science community. For instance, it is bundled with the Anaconda data science platform.

JSONata is a JSON query and transformation language that is inspired by the location path semantics of XPath 3.1. It was invented by Andrew Coleman and his colleagues at IBM in 2016. It is mostly used as an NPM package for browser- and NodeJS-based integration applications.

From a language point of view, jq and JSONata are quite similar, but they were inspired with different use cases in mind. For jq, it was having a JSON-aware command line tool. For JSONata, it was integrating RESTful applications. However, there are extensions to JSONata, such as [jsonata-cli](https://github.com/dashjoin/jsonata-cli) and [jfq](https://github.com/blgm/jfq), that make it available via the command line, making them competitors. Therefore, in this paper, we’re comparing the two from both a language semantics and a tooling point of view.

Note that there are several other languages like JMESPath, JSONiq, XPath 3.1, and [many others](https://stackoverflow.com/questions/777455/is-there-a-query-language-for-json) which you might consider for your use case.

### Language Semantics

Both programming languages are Turing complete and provide constructs for traversing, filtering, sorting, and transforming JSON trees. They also both offer support for variables, scoping, and closures.

The following table shows sample queries on the [JSONata invoice example](https://try.jsonata.org/) in both jq and JSONata.

**Query** jq   
JSONata 

**Entire document** .   
$ 

**Project field account name** .Account."Account Name"   
Account."Account Name" 

**Get all product IDs** .Account.Order[].Product[].ProductID   
Account.Order.Product.ProductID 

**Get a list of projects with SKU and product name (rename to Name)** .Account.Order[].Product[] | {"SKU": .SKU, "Name": "Product Name"}   
Account.Order.Product.{"SKU" : SKU, "Name": "Product Name"} 

**Products costing more than 40** .Account.Order[].Product[] | select(.Price > 40)   
Account.Order.Product[Price > 40] 

**Total price of the order** [.Account.Order[].Product[] | .Price * .Quantity] | add   
$sum(Account.Order.Product.(Price * Quantity)) 

**Get all colors** .. | .Colour? | select( . != null)   
**.Colour 

**Sort orders by price** .Account.Order[].Product | sort_by(.Price)   
Account.Order.Product^(Price) 

**Group by** .Account.Order[].Product | group_by(."Product Name")   
Account.Order.Product{`Product Name`: Price} 

**Sum implemented as reduce** reduce .Account.Order[].Product[] as $item (0;.+$item.Price*$item.Quantity)  
$reduce(Account.Order.Product, function($a,$v){$a+$v.Price*$v.Quantity},0) 

**Variables** .Account."Account Name" as $var | "The order is for: " + $var   
( $var := Account."Account Name"; "The order is for: " & $var ) 

**Defining functions** def add(x): . + x; add(1)   
( $add := function($x, $y){$x+$y}; $add($) ) 

**If then else** if .Account.Order[].Product[].Price > 30 then "expensive" else "cheap" end   
Account.Order.Product.(Price > 30 ? "expensive" : "cheap") 

**Add key value to top level object** . += {"key": "value"}   
$ ~> | $ | {"key": "value"} | 

**Built-in functions** .Account."Account Name" | length   
$length(Account."Account Name") 

**Extending** include "urldecode"; .http.referrer | url_decode   
Handled via engine’s registerFunction API 

### Similarities

As we can see from the table, the languages are quite similar in terms of their expressiveness. Jq is heavily inspired by UNIX pipes. Note that JSONata also offers an alternative syntax for nested function calls. Consider for example the selection of the account name followed by computing the string length: $length(Account.”Account Name”). This expression can also be written as: Account.”Account Name” ~> $length making is even more like its jq expression: .Account.”Account Name” | length.

### Differences

**Array Handling**

JSONata is more lenient in terms of how arrays are treated. When writing an expression, JSONata intuitively does the right thing when it encounters an array vs a simple type. Jq requires you to explicitly iterate over arrays, otherwise an error is raised. This can be seen in the frequent Order[] constructs in the examples. Likewise, subexpressions must be wrapped in square brackets before being fed into an aggregation. This can be seen in the example computing the total price of the order.

**Streaming**

Jq immediately allows you to stream data line by line. Therefore, it distinguishes between the input [1,2] and 1 newline 2. The former can be immediately piped into the add function. The latter can be converted to an array using the slurp command line option. This streaming feature is missing completely from JSONata and would have to be handled by an external component that somehow converts an input stream into a sequence of JSON trees before calling JSONata.

**Syntax**

The JSONata syntax is more concise and uses the JavaScript notation for writing functions, which seems more natural.

### Tooling

**jq**

Jq is written in C and is available for [many platforms](https://jqlang.github.io/jq/download/). It offers a rich set of command line options that support reading from files with different streaming options. There is also a popular [Go implementation](https://github.com/itchyny/gojq) available. Both options can be used from the command line or as a library within custom software.

**JSONata**

The JSONata reference is implemented in JavaScript and ships via [NPM](https://www.npmjs.com/package/jsonata). There are also implementations available in Rust, Go, Java, Python, and .NET, some of which use JavaScript interpreters to ensure compatibility. The Java implementation [jsonata-java](https://github.com/dashjoin/jsonata-java) is notable because it was ported 1:1 from the original and thus combines compatibility with very good performance. It is also available as a native-compiled [command line tool](https://github.com/dashjoin/jsonata-cli).

### Performance

When measuring performance of JSON CLI tools, the throughput of (possibly large) JSON files as such is an important base line. The performance of “common” transformations use cases must be measured against this baseline. The latter is obviously highly specific to the complexity of the transformation.

All in all, jsonata-cli offers decent performance for most JSON processing and transformation tasks. It performs comparably to, and often even faster than, jq or alternate implementations like gojq and jaq. It has reasonable memory and CPU consumption. With JSONata at its core, it is an easy to understand yet powerful JSON transformation solution.

Detailed results are [available here](https://github.com/dashjoin/jsonata-cli#performance).

### Conclusion

While jq and JSONata address JSON querying and transformation from very different angles, there are quite a few similarities. JSONata is a bit easier to use, especially with its intuitive handling of arrays. Jq excels with its built-in support for streaming. JSONata can easily be used on the web whereas jq is built for the command line. However, JSONata is available for command line use via some 3rd party extensions. Likewise, jq might be usable on the web via web assembly. Ultimately, it is a matter of choice and personal preference which one you choose.