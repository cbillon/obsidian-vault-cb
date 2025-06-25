---
link: https://stegard.net/tag/shell/page/4/
excerpt: Jq(1) is a surprisingly powerful command line JSON stream processing tool. I have not used it much, so this post will be a growing collection of random and useful examples to help me remember.
slurped: 2025-06-05T16:24
title: shell – Page 4 – Øyvind Stegard
tags:
  - json
  - jq
  - tips
---

[Jq(1)](http://manpages.ubuntu.com/manpages/focal/man1/jq.1.html) is a surprisingly powerful command line JSON stream processing tool. I have not used it much, so this post will be a growing collection of random and useful examples to help me remember.

## Table of contents

1. [Flatten an array of JSON objects into lines (tabular form), selecting only particular fields as columns](https://stegard.net/tag/shell/page/4/#flatten-to-lines)
2. [Create a JSON array of a list of string values, with or without pretty printing](https://stegard.net/tag/shell/page/4/#createarray)
3. ==[Extract two particular values from the first of multiple deeply nested JSON-objects](https://stegard.net/tag/shell/page/4/#extractvalues)==
4. [Filter stream of JSON objects based on some condition](https://stegard.net/tag/shell/page/4/#filterstreamunchanged)
5. [Create JSON from command line arguments](https://stegard.net/tag/shell/page/4/#createfromargs)
6. [Drill into, transform and select specific objects from structure based on matching](https://stegard.net/tag/shell/page/4/#drilltransformselect)

---

##### Flatten an array of JSON objects into lines (tabular form), selecting only particular fields as columns

```
# Given JSON data:
$ cat data.json
{
  "items": [
    {
      "id": "obj-1",
      "info": {
        "desc": "Description of Object 1",
        "type": "sometype"
      }
    },
    {
      "id": "obj-2",
      "info": {
        "desc": "Description of Object 2",
        "type": "sometype"
      }
    }
  ]
}

# Transform this into a table,
# where each line is "<id> <TAB> <description>":

$ jq -r '.items[]
         | .id + "\t" + .info.desc' data.json
obj-1   Description of Object 1
obj-2   Description of Object 2
```

Use option `-r` to avoid quoting the output strings, making them more suitable for parsing by for instance shell scripts.

##### **Create** a **JSON array** of a **list of string** values, with or without pretty printing

```
$ echo '"foo" "bar" "b a z"'|jq -s .
[
  "foo",
  "bar",
  "b a z"
]
```

```
$ echo '"foo" "bar" "b a z"'|jq -cs .
["foo","bar","b a z"]
```

The option `-s/--slurp` is what makes jq pack the values in an array here. The option `-c/--compact-output` toggles pretty printing.

```
$ cat data.json
{
  "data": [
    {
      "id": 1,
      "results": [
        {
          "type": "result",
          "objs": [
            {
              "a": "foo",
              "b": "bar"
            }
          ]
        }
      ]
    },
    {
      "id": 2,
      "results": [
        {
          "type": "result",
          "objs": [
            {
              "a": "foo2",
              "b": "bar2"
            }
          ]
        }
      ]
    }
  ]
}
$ cat data.json|\
  jq -r '.data[0].results[0].objs[0].a,
         .data[0].results[0].objs[0].b'
foo
bar
```

The option `-r/--raw-output` causes jq not to quote the extracted string values. Makes use of the values in shell scripts easier.

##### Filter stream of JSON objects based on some condition

```
$ cat data.ndjson
{ "id": 1, "color": "red" }
{ "id": 2, "color": "green" }
{ "id": 3, "color": "blue" }

$ cat data.ndjson | jq -s 'map(select(.color == "blue"))'
[
  {
    "id": 3,
    "color": "blue"
  }
]
```

Again we see the `-s/--slurp` option to read all objects into an array before applying our filtering code, causing the result to be an array of matching objects.

##### Create JSON from command line arguments

```
$ jq -n --arg a 1 --arg b 2 '{"a":$a, "b":$b}'
{
  "a": "1",
  "b": "2"
}
```

You can declare variables which jq makes available when constructing JSON, as can be seen in the example. The option `-n/--null-input` tells jq to not read anything on stdin, just output stuff. Notice that `--arg` by default treats values as JSON strings. If you need to encode values of **a** and **b** as real numbers in the above example, use `--argjson` instead:

```
$ jq -n --argjson a 1 --argjson b 2 '{"a":$a, "b":$b}'
{
  "a": 1,
  "b": 2
}
```

##### Drill into, transform and select specific objects from structure based on matching

Given the following input structure:

```
$ cat structure.json
{
  "items": [
    {
      "id": 1,
      "name": "The first item",
      "subitems": [
        {
          "name": "foo", "value": 0
        },
        {
          "name": "bar", "value": 1
        },
        {
          "name": "baz", "value": 2
        }
      ]
    },
    {
      "id": 2,
      "name": "The second item",
      "subitems": [
        {
          "name": "a", "value": 0
        },
        {
          "name": "b", "value": 1
        },
        {
          "name": "c", "value": 2
        }
      ]
    },
    {
      "id": 3,
      "name": "The third item",
      "subitems": [
        {
          "name": "Alpha α", "value": 0
        },
        {
          "name": "Beta β", "value": 1
        },
        {
          "name": "Gamma γ", "value": 2
        }
      ]
    }      
  ]
}
```

We can extract particular sub items from a particular item in the `items` list by using `select` and array iterator in a filter pipeline:

```
$ jq '.items[] 
      | select(.name == "The first item")
      | .subitems[]
      | select(.value > 1)' structure.json
{
  "name": "baz",
  "value": 2
}
```

Filtering explained in steps:

1. `.items[]` – selects the `items` key of the outer object and applies the iteration operator `[]` to the array. This turns the array into a stream of item values which can be processed further, one by one.
2. `select(.name == "The first item")` – filters stream of item objects, selecting only those with a key `name` having the value `"The first item"`.
3. `.subitems[]` – selects the key `.subitems` of input values and explodes the arrays into separate values for further processing, using the iteration operator.
4. `select(.value > 1)` – selects only subitems with a value greather than 1.
5. Resulting in a single matching subitem object, which becomes the output.

Build the filtering pipeline up from the first to the fourth step, each time adding the next `|....`, to better understand how the data is transformed and flows.

If we alter the last `|...select(.value > 1)` matching filter, we can see that _multiple JSON objects_ will be output by jq:

```
$ jq '.items[] | select(.name == "The first item") | .subitems[] | select(.value > 0)' structure.json
{
  "name": "bar",
  "value": 1
}
{
  "name": "baz",
  "value": 2
}
```

And if wrapping this output in an outer array is desired:

```
$ jq '.items[] | select(.name == "The first item") | .subitems[] | select(.value > 0)' structure.json | jq -s
[
  {
    "name": "bar",
    "value": 1
  },
  {
    "name": "baz",
    "value": 2
  }
]
```

Notice how we actually use a shell pipe to send the output of the first jq invocation to a new one using slurp mode to wrap input objects in an array. (Don’t confuse shell pipes with jq filtering pipes when mixing the two !)