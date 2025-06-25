---
link: https://megamorf.gitlab.io/cheat-sheets/jq/
excerpt: jq is a lightweight and flexible command-line JSON processorExample datafile.json[  {
slurped: 2025-06-05T16:32
title: Cheat Sheet - jq - Seb's IT blog
tags:
  - jq
  - tips
---

jq is a lightweight and flexible command-line JSON processor

## Example data

**file.json**

```
[
  {
    "_id": "5c1abfdac8a85098a79cb618",
    "index": 0,
    "created_at": "2016-10-21T20:51:16.000+08:00",
    "guid": "e7fba244-16cd-427e-ba97-1bcf57205317",
    "isActive": true,
    "balance": "$3,745.35",
    "picture": "http://placehold.it/32x32",
    "age": 36,
    "eyeColor": "blue",
    "name": {
      "first": "Lena",
      "last": "Mullen"
    },
    "company": "CORMORAN",
    "email": "lena.mullen@cormoran.net",
    "tags": [
      "consequat",
      "ea",
      "velit",
      "amet",
      "non"
    ],
    "friends": [
      {
        "id": 0,
        "name": "Ray Nelson"
      },
      {
        "id": 1,
        "name": "Barnett Barron"
      },
      {
        "id": 2,
        "name": "Megan Simpson"
      }
    ]
  },
  {
    "_id": "5c1abfda37fdec50ed48cbd7",
    "index": 1,
    "created_at": "2016-10-22T08:09:16.000+08:00",
    "guid": "e12bef5d-49db-4acc-889b-6aac31b91c0d",
    "isActive": true,
    "balance": "$3,319.15",
    "picture": "http://placehold.it/32x32",
    "age": 20,
    "eyeColor": "blue",
    "name": {
      "first": "Martha",
      "last": "Summers"
    },
    "company": "EXOTERIC",
    "email": "martha.summers@exoteric.info",
    "tags": [
      "aliqua",
      "exercitation",
      "id",
      "cupidatat",
      "excepteur"
    ],
    "friends": [
      {
        "id": 0,
        "name": "Travis Harper"
      },
      {
        "id": 1,
        "name": "Mable Jordan"
      },
      {
        "id": 2,
        "name": "Durham Curry"
      }
    ]
  },
  {
    "_id": "5c1abfda3834725b63588532",
    "index": 2,
    "created_at": "2016-10-23T09:19:55.000+08:00",
    "guid": "8949115d-13f9-4ab3-bc8a-41ad21a3aeb6",
    "isActive": false,
    "balance": "$2,913.66",
    "picture": "http://placehold.it/32x32",
    "age": 28,
    "eyeColor": "blue",
    "name": {
      "first": "Luna",
      "last": "Page"
    },
    "company": "STELAECOR",
    "email": "luna.page@stelaecor.org",
    "tags": [
      "irure",
      "et",
      "aute",
      "reprehenderit",
      "et"
    ],
    "friends": [
      {
        "id": 0,
        "name": "Minnie Crane"
      },
      {
        "id": 1,
        "name": "Klein Kelley"
      },
      {
        "id": 2,
        "name": "Cassie Gibbs"
      }
    ]
  }
]
```

## Generic commands

Output a JSON file, in pretty-print format:

Pretty-print API output:

```
curl example.org/api/v1/users | jq .
```

Output all elements from arrays (or all key-value pairs from objects) in a JSON file:

Read JSON objects from a file into an array, and output it (inverse of `jq .[]`):

Output the first element in a JSON file:

Output the value of a given key of the first element in a JSON text from stdin:

```
cat file.json | jq .[0].key_name
```

Select multiple properties from the first element:

```
jq '.[0] | { _id, email }' file.json
```

Output the value of a given key of each element in a JSON text from stdin:

```
cat file.json | jq 'map(.key_name)'
```

Filter, map, reduce:

```
cat file.json | jq '.[] | select(.age == 36)'
cat file.json | jq 'map({ _id, email })'
cat file.json | jq 'reduce .[] as $item (0; . + $item.age)'
```

Minify json

```
jq -c . < file.json
jq -c . file.json
```

Remove certain properties:

```
# delete the guid property from each item and the id from every element in its friends property
cat file.json | jq '.[] |= del(.guid, .friends[].id)'
```

## Escape and condense JSON

This can be useful for templating - e.g. let’s say we have a file with the following contents `{"data": "_TEMPLATE_"}` and the string `_TEMPLATE_` should be replaced. Input is a pretty (indented and line breaks) JSON object. To insert that we need to remove line breaks and escape it:

```
JSON_CONDENSED_ESCAPED=$(jq -c '. | @json' < pretty_input.json)
sed "s#_TEMPLATE_DATA_#${JSON_CONDENSED_ESCAPED}#" output.json
```

## Select item in time range

Source: [Stackoverflow](https://stackoverflow.com/questions/40210276/how-to-select-a-date-range-from-a-json-string-by-using-jq/40211953#40211953)

```
# Use `jq` to select the objects in the array whose .created_at
# property value falls between "2016-10-21:T20:51" and "2016-10-22T08:09"
# and return them as an array (effectively a sub-array of the input).
# To solve the problem as originally stated, simply pass "2016-10-21"
# and "2016-10-22" instead.
cat file.json | jq --arg s '2016-10-21T20:51' --arg e '2016-10-22T08:09' '
    map(select(.created_at | . >= $s and . <= $e + "z"))'
```

- Arguments `--arg s '2016-10-21T20:51'` and `--arg e '2016-10-22T08:09'` define variables `$s` (start of date+time range) and `$e` (end of date+time range) respectively, for use inside the `jq` script.
    
- Function `map()` applies the enclosed expression to all the elements of the input array and outputs the results as an array, too.
    
- Function `select()` accepts a filtering expression: every input object is evaluated against the enclosed expression, and the input object is only passed out if the expression evaluates to a “truthy” value.
    
- Expression `.created_at | . >= $s and . <= $e + "z"` accesses each input object’s `created_at` property and sends its value to the comparison expression, which performs _lexical_ comparison, which - due to the formatting of the date+time strings - amounts to _chronological_ comparison.
    
    - Note the trailing `"z"` appended to the range endpoint, to ensure that it matches all date+time strings in the JSON string that _prefix-match_ the endpoint; e.g., endpoint `2016-10-22T08:09` should match `2016-10-22T08:09:01` as well as `2016-10-22T08:59`.
        
    - This lexical approach allows you to specify as many components from the beginning as desired in order to narrow or widen the date range; e.g. `--arg s '2016-10-01' --arg e '2016-10-31'` would match all entries for the entire month of October 2016.
        

## Return query with custom format

```
# Get last backup timestamp
docker exec -i restic-backups restic snapshots --host ${HOST} latest --json | jq -r 'max_by(.time) | .time | sub("[.][0-9]+"; "") | sub("Z"; "+00:00") | def parseDate(date): date | capture("(?<no_tz>.*)(?<tz_sgn>[-+])(?<tz_hr>\\d{2}):(?<tz_min>\\d{2})$") | (.no_tz + "Z" | fromdateiso8601) - (.tz_sgn + "60" | tonumber) * ((.tz_hr | tonumber) * 60 + (.tz_min | tonumber)); parseDate(.) | "restic_last_snapshot_ts \(.)"' >> ${TEMP_FILE}
# Get last backup size in bytes and files count
docker exec -i restic-backups restic stats --host ${HOST} latest --json | jq -r '"restic_stats_total_size_bytes \(.total_size)\nrestic_stats_total_file_count \(.total_file_count)"' >> ${TEMP_FILE}
```

## Log valid json from the shell with jq

jq will take care of all the necessary escaping and always produce valid single line JSON-structured messages, regardless of message payload.

```
# Accept stdin
echo -n "foo bar" | jq -Rsc '{"@timestamp": now|strftime("%Y-%m-%dT%H:%M:%S%z"), "message":.}'

# Pass arguments
jq --arg m "$*" -nc '{"@timestamp": now|strftime("%Y-%m-%dT%H:%M:%S%z"), "message":$m}'
```

[https://stegard.net/2021/07/how-to-make-a-shell-script-log-json-messages/](https://stegard.net/2021/07/how-to-make-a-shell-script-log-json-messages/)