---
link: https://fabianlee.org/2025/12/27/jq-resolving-cannot-index-array-with-string-errors-from-jq/
site: "Fabian Lee : Software Engineer"
excerpt: 'jq has a very powerful expression language for querying and
  transforming json.  But when dealing with complex data structures and
  responses in the real-world, well-tested happy path queries can fail. One
  common error is: jq: error (at <stdin>:1): Cannot index array with string
  "..." The root cause is that your query is attempting to pull ... jq:
  Resolving “Cannot index array with string” errors from jq'
twitter: "https://twitter.com/Fabian Lee : Software Engineer"
tags:
  - slurp/error
  - slurp/jq
  - slurp/json
slurped: 2026-08-30T10:40
title: "jq: Resolving “Cannot index array with string” errors from jq"
---

![](https://fabianlee.org/wp-content/uploads/2025/12/image-2.png)

[jq](https://jqlang.org/) has a very powerful expression language for querying and transforming json.  But when dealing with complex data structures and responses in the real-world, well-tested happy path queries can fail.

One common error is:

`jq: error (at <stdin>:1): Cannot index array with string "..."`

The root cause is that your query is attempting to pull a key from a dictionary, when in fact the JSON at that level is an array.

## Root Cause explained

Consider the simple JSON below that is a root level array of dictionary.  An error is thrown if the query attempts to pull a key from the root level array.

# INVALID - trying to pull by key when actual structure is array
$ echo '[{"id": 1},{"id":2}]' | jq '.id'
jq: error (at <stdin>:1): Cannot index array with string "id"

# use 'type' function to confirm root level is 'array', so needs int index
$ echo '[{"id": 1},{"id":2}]' | jq -r '. | type'
array

# valid - pull first array element using int
$ echo '[{"id": 1},{"id":2}]' | jq -c '.[0]'
{"id": 1}

## Candidate fixes

If you expect the JSON structure to be consistent, the easiest fix is to simply realize that level is an array type and change to integer indexing.

But in the real-world there are often corner cases that have to be considered, such as returned JSON from an API depending on permissions, circuit breaker responses, etc.  One way to avoid the error is use the [‘?’ operator](https://fabianlee.org/2025/12/27/jq-resolving-cannot-index-array-with-string-errors-from-jq/) to indicate the field is optional.

# use '?' to indicate optional field and avoid error, returns empty result
$ echo '[{"id": 1},{"id":2}]' | jq '.id?'

# can provide alternative to empty result
$ echo '[{"id": 1},{"id":2}]' | jq '.id? // 99'

Another way is to put a [conditional on](https://fabianlee.org/2025/12/27/jq-resolving-cannot-index-array-with-string-errors-from-jq/) the structure type and make the query accommodate both scenarios.  Notice the query is the same, just the JSON structure differs.

# if array, then pull .id from each element
$ echo '[{"id": 1},{"id":2}]' | jq -r '. | if type=="array" then .[].id | . else .id end'
1
2

# if dictionary, pull dictionary key
$ echo '{ "id": 1 }' | jq -r '. | if type=="array" then .[].id | . else .id end'
1

## Error “Cannot index object with number”

Conversely, if you try to pull an array element from a dictionary, you will see this error.

# INVALID - trying to pull by index when actual structure is dictionary
$ echo '{ "id": 1 }' | jq -r '.[0]'
jq: error (at <stdin>:1): Cannot index object with number

# type can be confirmed to be dictionary, not array
$ echo '{ "id": 1 }' | jq -r '. | type'
object

Once again, if the issue is human misunderstanding of the structure, then make sure you use a key at this level (instead of an array index).  If the problem is more nuanced and you want to mute the error, then use the optional operator as below OR use the type conditional as shown in the section above to handle both scenarios.

# workaround: error can be avoided by making optional with '?'
$ echo '{ "id": 1 }' | jq -r '.[0]?'

**REFERENCES**

[jq manual](https://jqlang.org/manual/)

[stackoverflow, array or object](https://stackoverflow.com/questions/64684980/jq-process-json-where-an-element-can-be-an-array-or-object)

**NOTES**

Alternative “//” can also return objects

$ echo '[{"id": 1},{"id":2}]' | jq '.[3] // {"id":99}' -c
{"id":99}