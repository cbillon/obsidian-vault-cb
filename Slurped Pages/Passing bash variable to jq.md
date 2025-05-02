---
link: https://stackoverflow.com/questions/40027395/passing-bash-variable-to-jq
byline: |-
  asiddasidd
          
              3,26122 gold badges1212 silver badges99 bronze badges
site: Stack Overflow
excerpt: >-
  I have written a script to retrieve certain value from file.json. It works if
  I provide the value to jq select, but the variable doesn't seem to work (or I
  don't know how to use it).


  #!/bin/sh


  #t...
slurped: 2024-11-01T09:37
title: Passing bash variable to jq
---


Consider also passing in the shell variable (EMAILID) as a jq variable (here also EMAILID, for the sake of illustration):

```
   projectID=$(jq -r --arg EMAILID "$EMAILID" '
        .resource[]
        | select(.username==$EMAILID) 
        | .id' file.json)
```


For the record, another possibility would be to use jq's `env` function for accessing environment variables. For example, consider this sequence of bash commands:

```
[email protected]  # not exported
EMAILID="$EMAILID" jq -n 'env.EMAILID'
```

The output is a JSON string:

```
"[email protected]"
```

### shell arrays

Unfortunately, shell arrays are a different kettle of fish. Here are two SO resources regarding the ingestion of such arrays:

[JQ - create JSON array using bash array with space](https://stackoverflow.com/questions/61588841/jq-create-json-array-using-bash-array-with-space)

[Convert bash array to json array and insert to file using jq](https://stackoverflow.com/questions/49184557/convert-bash-array-to-json-array-and-insert-to-file-using-jq)


I resolved this issue by escaping the inner double quotes

```
projectID=$(cat file.json | jq -r ".resource[] | select(.username==\"$EMAILID\") | .id")
```

	