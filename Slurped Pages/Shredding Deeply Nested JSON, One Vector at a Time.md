---
link: https://duckdb.org/2023/03/03/json.html
byline: Laurens Kuiper
site: DuckDB
date: 2023-03-03T00:00
excerpt: We recently improved DuckDB's JSON extension so JSON files can be
  directly queried as if they were tables.
twitter: https://twitter.com/@lnkuiper
slurped: 2024-12-19T21:58
title: Shredding Deeply Nested JSON, One Vector at a Time
---

_TL;DR: We recently improved DuckDB's JSON extension so JSON files can be directly queried as if they were tables._

> We updated this blog post in December 2024 to reflect the changes in DuckDB's JSON syntax.

DuckDB has a [JSON extension](https://duckdb.org/docs/data/json/overview.html) that can be installed and loaded through SQL:

```
INSTALL 'json';
LOAD 'json';
```

The JSON extension supports various functions to create, read, and manipulate JSON strings. These functions are similar to the JSON functionality provided by other databases such as [PostgreSQL](https://www.postgresql.org/docs/current/functions-json.html) and [MySQL](https://dev.mysql.com/doc/refman/8.0/en/json.html). DuckDB uses [yyjson](https://github.com/ibireme/yyjson) internally to parse JSON, a high-performance JSON library written in ANSI C. Many thanks to the yyjson authors and contributors!

Besides these functions, DuckDB is now able to read JSON directly! This is done by automatically detecting the types and column names, then converting the values within the JSON to DuckDB's vectors. The automated schema detection dramatically simplifies working with JSON data and subsequent queries on DuckDB's vectors are significantly faster!

## [Reading JSON Automatically with DuckDB](https://duckdb.org/2023/03/03/json.html#reading-json-automatically-with-duckdb)

Since [version 0.7.0](https://duckdb.org/2023/02/13/announcing-duckdb-070.html), DuckDB supports JSON table functions. To demonstrate these, we will read [`todos.json`](https://duckdb.org/data/json/todos.json), a [fake TODO list](https://jsonplaceholder.typicode.com/todos) containing 200 fake TODO items (only the first two items are shown):

```
[
  {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  },
  {
    "userId": 1,
    "id": 2,
    "title": "quis ut nam facilis et officia qui",
    "completed": false
  },
  ...
]
```

Each TODO item is an entry in the JSON array, but in DuckDB, we'd like a table where each entry is a row. This is as easy as:

```
SELECT * FROM 'todos.json' LIMIT 5;
```

|userId|id|title|completed|
|---|---|---|---|
|1|1|delectus aut autem|false|
|1|2|quis ut nam facilis et officia qui|false|
|1|3|fugiat veniam minus|false|
|1|4|et porro tempora|true|
|1|5|laboriosam mollitia et enim quasi adipisci quia provident illum|false|

Finding out which user completed the most TODO items is as simple as:

```
SELECT userId, sum(completed::INTEGER) total_completed
FROM 'todos.json'
GROUP BY userId
ORDER BY total_completed DESC
LIMIT 1;
```

|userId|total_completed|
|---|---|
|5|12|

Under the hood, DuckDB recognizes the `.json` file extension in `'todos.json'`, and calls `read_json('todos.json')` instead. This function is similar to our `read_csv` function, which [automatically infers column names and types for CSV files](https://duckdb.org/docs/data/csv/auto_detection.html).

Like our other table functions, `read_json` supports reading multiple files by passing a list, e.g., `read_json(['file1.json', 'file2.json'])`, but also globbing, e.g., `read_json('file*.json')`. DuckDB will read multiple files in parallel.

## [Newline Delimited JSON](https://duckdb.org/2023/03/03/json.html#newline-delimited-json)

Not all JSON adheres to the format used in `todos.json`, which is an array of 'records'. Newline-delimited JSON, or [NDJSON](https://github.com/ndjson/ndjson-spec), stores each row on a new line. DuckDB also supports reading (and writing!) this format. First, let's write our TODO list as NDJSON:

```
COPY (SELECT * FROM 'todos.json') to 'todos2.json';
```

Again, DuckDB recognizes the `.json` suffix in the output file and automatically infers that we mean to use `(FORMAT JSON)`. The created file looks like this (only the first two records are shown):

```
{"userId":1,"id":1,"title":"delectus aut autem","completed":false}
{"userId":1,"id":2,"title":"quis ut nam facilis et officia qui","completed":false}
...
```

DuckDB can read this file in exactly the same way as the original one:

```
SELECT * FROM 'todos2.json';
```

If your JSON file is newline-delimited, DuckDB can parallelize reading. This can be specified by calling `read_ndjson` or passing the `records = true` parameter to `read_json`:

```
SELECT * FROM read_ndjson('todos2.json');
SELECT * FROM read_json('todos2.json', records = true);
```

You can also set `records = auto` to auto-detect whether the JSON file is newline-delimited.

## [Other JSON Formats](https://duckdb.org/2023/03/03/json.html#other-json-formats)

When the `read_json` function is used directly, the format of the JSON can be specified using the `format` parameter. This parameter defaults to `'auto'`, which tells DuckDB to infer what kind of JSON we are dealing with. The first `format` is `'array'`, while the second is `'nd'`. This can be specified like so:

```
SELECT * FROM read_json('todos.json', format = 'array');
SELECT * FROM read_json('todos2.json', format = 'nd');
```

Another supported format is `unstructured`. With this format, records are not required to be a JSON object but can also be a JSON array, string, or anything supported in JSON.

## [Manual Schemas](https://duckdb.org/2023/03/03/json.html#manual-schemas)

What you may also have noticed is the `auto_detect` parameter. This parameter tells DuckDB to infer the schema, i.e., determine the names and types of the returned columns. These can manually be specified like so:

```
SELECT *
FROM read_json(
    'todos.json',
    columns = {userId: 'INTEGER', id: 'INTEGER', title: 'VARCHAR', completed: 'BOOLEAN'},
    format = 'array'
);
```

You don't have to specify all fields, just the ones you're interested in:

```
SELECT *
FROM read_json(
    'todos.json',
    columns = {userId: 'INTEGER', completed: 'BOOLEAN'},
    format = 'array'
);
```

Now that we know how to use the new DuckDB JSON table functions let's dive into some analytics!

## [GitHub Archive Examples](https://duckdb.org/2023/03/03/json.html#github-archive-examples)

[GH Archive](https://www.gharchive.org/) is a project to record the public GitHub timeline, archive it, and make it easily accessible for further analysis. Every hour, a GZIP compressed, newline-delimited JSON file containing all public events on GitHub is uploaded. I downloaded a whole day (2023-02-08) of activity using `wget` and stored the 24 files (starting with [`2023-02-08-0.json.gz`](https://data.gharchive.org/2023-02-08-0.json.gz)) in a directory called `gharchive_gz`. You can get the full day's archive as [`gharchive-2023-02-08.zip`](https://blobs.duckdb.org/data/gharchive-2023-02-08.zip):

Keep in mind that the data is compressed:

Decompressed, one day's worth of GitHub activity amounts to more than 18 GB of JSON.

```
gunzip -dc gharchive_gz/* | wc -c
```

To get a feel of what the data looks like, we run the following query:

```
SELECT json_group_structure(json)
FROM (
    SELECT *
    FROM read_ndjson_objects('gharchive_gz/*.json.gz')
    LIMIT 2048
);
```

Here, we use our `read_ndjson_objects` function, which reads the JSON objects in the file as raw JSON, i.e., as strings. The query reads the first 2048 records of JSON from the JSON files `gharchive_gz` directory and describes the structure. You can also directly query the JSON files from GH Archive using DuckDB's [`httpfs` extension](https://duckdb.org/docs/extensions/httpfs/overview.html), but we will be querying the files multiple times, so it is better to download them in this case.

> Tip In the CLI client, you can use `.mode line` to make the output easier to read.

I formatted the result using [an online JSON formatter & validator](https://jsonformatter.curiousconcept.com/):

```
{
   "id":"VARCHAR",
   "type":"VARCHAR",
   "actor":{
      "id":"UBIGINT",
      "login":"VARCHAR",
      "display_login":"VARCHAR",
      "gravatar_id":"VARCHAR",
      "url":"VARCHAR",
      "avatar_url":"VARCHAR"
   },
   "repo":{
      "id":"UBIGINT",
      "name":"VARCHAR",
      "url":"VARCHAR"
   },
   "payload":{"..."},
   "public":"BOOLEAN",
   "created_at":"VARCHAR",
   "org":{
      "id":"UBIGINT",
      "login":"VARCHAR",
      "gravatar_id":"VARCHAR",
      "url":"VARCHAR",
      "avatar_url":"VARCHAR"
   }
}
```

I left `"payload"` out because it consists of deeply nested JSON, and its formatted structure takes up more than 1000 lines!

So, how many records are we dealing with exactly? Let's count it using DuckDB:

```
SELECT count(*) AS count
FROM 'gharchive_gz/*.json.gz';
```

|count|
|---|
|4434953|

That's around 4.4M daily events, which amounts to almost 200K events per hour. This query takes around 7.3 seconds on my laptop, a 2020 MacBook Pro with an M1 chip and 16 GB of memory. This is the time it takes to decompress the GZIP compression and parse every JSON record.

To see how much time is spent decompressing GZIP in the query, I also created a `gharchive` directory containing the same data but uncompressed. Running the same query on the uncompressed data takes around 5.4 seconds, almost 2 seconds faster. We got faster, but we also read more than 18 GB of data from storage, as opposed to 2.3 GB when it was compressed. So, this comparison really depends on the speed of your storage. I prefer to keep the data compressed.

As a side note, the speed of this query really shows how fast yyjson is!

So, what kind of events are in the GitHub data?

```
SELECT type, count(*) AS count
FROM 'gharchive_gz/*.json.gz'
GROUP BY type
ORDER BY count DESC;
```

|type|count|
|---|---|
|PushEvent|2359096|
|CreateEvent|624062|
|PullRequestEvent|366090|
|IssueCommentEvent|238660|
|WatchEvent|231486|
|DeleteEvent|154383|
|PullRequestReviewEvent|131107|
|IssuesEvent|88917|
|PullRequestReviewCommentEvent|79540|
|ForkEvent|64233|
|CommitCommentEvent|36823|
|ReleaseEvent|23004|
|MemberEvent|14872|
|PublicEvent|14500|
|GollumEvent|8180|

This query takes around 7.4 seconds, not much more than the `count(*)` query. So as we can see, data analysis is very fast once everything has been decompressed and parsed.

The most common event type is the [`PushEvent`](https://docs.github.com/en/developers/webhooks-and-events/events/github-event-types#pushevent), taking up more than half of all events, unsurprisingly, which is people pushing their committed code to GitHub. The least common event type is the [`GollumEvent`](https://docs.github.com/en/developers/webhooks-and-events/events/github-event-types#gollumevent), taking up less than 1% of all events, which is a creation or update of a wiki page.

If we want to analyze the same data multiple times, decompressing and parsing every time is redundant and slows down the analysis. Instead, we can create a DuckDB table like so:

```
CREATE TABLE events AS
    SELECT * EXCLUDE (payload)
    FROM 'gharchive_gz/*.json.gz';
```

This takes around 9 seconds if you're using an in-memory database. If you're using an on-disk database, this takes around 13 seconds and results in a database size of 444 MB. When using an on-disk database, DuckDB ensures the table is persistent and performs [all kinds of compression](https://duckdb.org/2022/10/28/lightweight-compression.html). Note that we have temporarily ignored the `payload` field using the convenient `EXCLUDE` clause.

To get a feel of what we've read, we can ask DuckDB to describe the table:

```
DESCRIBE SELECT * FROM events;
```

This gives us the following:

|cid|name|type|notnull|dflt_value|pk|
|---|---|---|---|---|---|
|0|id|BIGINT|false||false|
|1|type|VARCHAR|false||false|
|2|actor|STRUCT(id UBIGINT, login VARCHAR, display_login VARCHAR, gravatar_id VARCHAR, url VARCHAR, avatar_url VARCHAR)|false||false|
|3|repo|STRUCT(id UBIGINT, name VARCHAR, url VARCHAR)|false||false|
|4|public|BOOLEAN|false||false|
|5|created_at|TIMESTAMP|false||false|
|6|org|STRUCT(id UBIGINT, login VARCHAR, gravatar_id VARCHAR, url VARCHAR, avatar_url VARCHAR)|false||false|

As we can see, the `"actor"`, `"repo"` and `"org"` fields, which are JSON objects, have been converted to DuckDB structs. The `"id"` column was a string in the original JSON but has been converted to a `BIGINT` by DuckDB's automatic type detection. DuckDB can also detect a few different `DATE`/`TIMESTAMP` formats within JSON strings, as well as `TIME` and `UUID`.

Now that we created the table, we can analyze it like any other DuckDB table! Let's see how much activity there was in the [`duckdb/duckdb` GitHub repository](https://github.com/duckdb/duckdb) on this specific day:

```
SELECT type, count(*) AS count
FROM events
WHERE repo.name = 'duckdb/duckdb'
GROUP BY type
ORDER BY count DESC;
```

|type|count|
|---|---|
|PullRequestEvent|35|
|IssueCommentEvent|30|
|WatchEvent|29|
|PushEvent|15|
|PullRequestReviewEvent|14|
|IssuesEvent|9|
|PullRequestReviewCommentEvent|7|
|ForkEvent|3|

That's a lot of pull request activity! Note that this doesn't mean that 35 pull requests were opened on this day, activity within a pull request is also counted. If we [search through the pull requests for that day](https://github.com/duckdb/duckdb/pulls?q=is%3Apr+created%3A2023-02-08+), we see that there are only 15. This is more activity than normal because most of the DuckDB developers were busy fixing bugs for the 0.7.0 release.

Now, let's see who was the most active:

```
SELECT actor.login, count(*) AS count
FROM events
WHERE repo.name = 'duckdb/duckdb'
  AND type = 'PullRequestEvent'
GROUP BY actor.login
ORDER BY count desc
LIMIT 5;
```

|login|count|
|---|---|
|Mytherin|19|
|Mause|4|
|carlopi|3|
|Tmonster|2|
|lnkuiper|2|

As expected, Mark Raasveldt (Mytherin, co-founder of DuckDB Labs) was the most active! My activity (lnkuiper, software engineer at DuckDB Labs) also shows up.

## [Handling Inconsistent JSON Schemas](https://duckdb.org/2023/03/03/json.html#handling-inconsistent-json-schemas)

So far, we have ignored the `"payload"` of the events. We ignored it because the contents of this field are different based on the type of event. We can see how they differ with the following query:

```
SELECT json_group_structure(payload) AS structure
FROM (
    SELECT *
    FROM read_json(
        'gharchive_gz/*.json.gz',
        columns = {
            id: 'BIGINT',
            type: 'VARCHAR',
            actor: 'STRUCT(id UBIGINT,
                          login VARCHAR,
                          display_login VARCHAR,
                          gravatar_id VARCHAR,
                          url VARCHAR,
                          avatar_url VARCHAR)',
            repo: 'STRUCT(id UBIGINT, name VARCHAR, url VARCHAR)',
            payload: 'JSON',
            public: 'BOOLEAN',
            created_at: 'TIMESTAMP',
            org: 'STRUCT(id UBIGINT, login VARCHAR, gravatar_id VARCHAR, url VARCHAR, avatar_url VARCHAR)'
        },
        records = true
    )
    WHERE type = 'WatchEvent'
    LIMIT 2048
);
```

|structure|
|---|
|{"action":"VARCHAR"}|

The `"payload"` field is simple for events of type `WatchEvent`. However, if we change the type to `PullRequestEvent`, we get a JSON structure of more than 500 lines when formatted with a JSON formatter. We don't want to look through all those fields, so we cannot use our automatic schema detection, which will try to get them all. Instead, we can manually supply the structure of the fields we're interested in. DuckDB will skip reading the other fields. Another approach is to store the `"payload"` field as DuckDB's JSON data type and parse it at query time (see the example later in this post!).

I stripped down the JSON structure for the `"payload"` of events with the type `PullRequestEvent` to the things I'm actually interested in:

```
{
   "action":"VARCHAR",
   "number":"UBIGINT",
   "pull_request":{
      "url":"VARCHAR",
      "id":"UBIGINT",
      "title":"VARCHAR",
      "user":{
         "login":"VARCHAR",
         "id":"UBIGINT",
      },
      "body":"VARCHAR",
      "created_at":"TIMESTAMP",
      "updated_at":"TIMESTAMP",
      "assignee":{
         "login":"VARCHAR",
         "id":"UBIGINT",
      },
      "assignees":[
         {
            "login":"VARCHAR",
            "id":"UBIGINT",
         }
      ],
  }
}
```

This is technically not valid JSON because there are trailing commas. However, we try to [allow trailing commas wherever possible](https://duckdb.org/2022/05/04/friendlier-sql.html#trailing-commas) in DuckDB, including JSON!

We can now plug this into the `columns` parameter of `read_json`, but we need to convert it to a DuckDB type first. I'm lazy, so I prefer to let DuckDB do this for me:

```
SELECT typeof(
    json_transform('{}', '{
        "action":"VARCHAR",
        "number":"UBIGINT",
        "pull_request":{
          "url":"VARCHAR",
          "id":"UBIGINT",
          "title":"VARCHAR",
          "user":{
              "login":"VARCHAR",
              "id":"UBIGINT",
          },
          "body":"VARCHAR",
          "created_at":"TIMESTAMP",
          "updated_at":"TIMESTAMP",
          "assignee":{
              "login":"VARCHAR",
              "id":"UBIGINT",
          },
          "assignees":[
              {
                "login":"VARCHAR",
                "id":"UBIGINT",
              }
          ],
        }
    }')
);
```

This gives us back a DuckDB type that we can plug the type into our function! Note that because we are not auto-detecting the schema, we have to supply `timestampformat` to be able to parse the timestamps correctly. The key `"user"` must be surrounded by quotes because it is a reserved keyword in SQL:

```
CREATE OR REPLACE TABLE pr_events AS
    SELECT *
    FROM read_json(
        'gharchive_gz/*.json.gz',
        columns = {
            id: 'BIGINT',
            type: 'VARCHAR',
            actor: 'STRUCT(id UBIGINT,
                          login VARCHAR,
                          display_login VARCHAR,
                          gravatar_id VARCHAR,
                          url VARCHAR,
                          avatar_url VARCHAR)',
            repo: 'STRUCT(id UBIGINT, name VARCHAR, url VARCHAR)',
            payload: 'STRUCT(
                        action VARCHAR,
                        number UBIGINT,
                        pull_request STRUCT(
                          url VARCHAR,
                          id UBIGINT,
                          title VARCHAR,
                          "user" STRUCT(
                            login VARCHAR,
                            id UBIGINT
                          ),
                          body VARCHAR,
                          created_at TIMESTAMP,
                          updated_at TIMESTAMP,
                          assignee STRUCT(login VARCHAR, id UBIGINT),
                          assignees STRUCT(login VARCHAR, id UBIGINT)[]
                        )
                      )',
            public: 'BOOLEAN',
            created_at: 'TIMESTAMP',
            org: 'STRUCT(id UBIGINT, login VARCHAR, gravatar_id VARCHAR, url VARCHAR, avatar_url VARCHAR)'
        },
        format = 'newline_delimited',
        records = true,
        timestampformat = '%Y-%m-%dT%H:%M:%SZ'
    )
    WHERE type = 'PullRequestEvent';
```

This query completes in around 36 seconds with an on-disk database (resulting size is 478 MB) and 9 seconds with an in-memory database. If you don't care about preserving insertion order, you can speed the query up with this setting:

```
SET preserve_insertion_order = false;
```

With this setting, the query completes in around 27 seconds with an on-disk database and 8.5 seconds with an in-memory database. The difference between the on-disk and in-memory case is quite substantial here because DuckDB has to compress and persist much more data.

Now we can analyze pull request events! Let's see what the maximum number of assignees is:

```
SELECT max(length(payload.pull_request.assignees)) AS max_assignees
FROM pr_events;
```

|max_assignees|
|---|
|10|

That's a lot of people reviewing a single pull request!

We can check who was assigned the most:

```
WITH assignees AS (
    SELECT payload.pull_request.assignee.login AS assignee
    FROM pr_events
    UNION ALL
    SELECT unnest(payload.pull_request.assignees).login AS assignee
    FROM pr_events
)
SELECT assignee, count(*) AS count
FROM assignees
WHERE assignee NOT NULL
GROUP BY assignee
ORDER BY count DESC
LIMIT 5;
```

|assignee|count|
|---|---|
|poad|494|
|vinayakkulkarni|268|
|tmtmtmtm|198|
|fisker|98|
|icemac|84|

That's a lot of assignments – although I suspect there are duplicates in here.

## [Storing as JSON to Parse at Query Time](https://duckdb.org/2023/03/03/json.html#storing-as-json-to-parse-at-query-time)

Specifying the JSON schema of the `"payload"` field was helpful because it allowed us to directly analyze what is there, and subsequent queries are much faster. Still, it can also be quite cumbersome if the schema is complex. If you don't want to specify the schema of a field, you can set the type as `'JSON'`:

```
CREATE OR REPLACE TABLE pr_events AS
    SELECT *
    FROM read_json(
        'gharchive_gz/*.json.gz',
        columns = {
            id: 'BIGINT',
            type: 'VARCHAR',
            actor: 'STRUCT(id UBIGINT,
                          login VARCHAR,
                          display_login VARCHAR,
                          gravatar_id VARCHAR,
                          url VARCHAR,
                          avatar_url VARCHAR)',
            repo: 'STRUCT(id UBIGINT, name VARCHAR, url VARCHAR)',
            payload: 'JSON',
            public: 'BOOLEAN',
            created_at: 'TIMESTAMP',
            org: 'STRUCT(id UBIGINT, login VARCHAR, gravatar_id VARCHAR, url VARCHAR, avatar_url VARCHAR)'
        },
        format = 'newline_delimited',
        records = true,
        timestampformat = '%Y-%m-%dT%H:%M:%SZ'
    )
    WHERE type = 'PullRequestEvent';
```

This will load the `"payload"` field as a JSON string, and we can use DuckDB's JSON functions to analyze it when querying. For example:

```
SELECT DISTINCT payload->>'action' AS action, count(*) AS count
FROM pr_events
GROUP BY action
ORDER BY count DESC;
```

The `->>` arrow is short-hand for our `json_extract_string` function. Creating the entire `"payload"` field as a column with type `JSON` is not the most efficient way to get just the `"action"` field, but this example is just to show the flexibility of `read_json`. The query results in the following table:

|action|count|
|---|---|
|opened|189096|
|closed|174914|
|reopened|2080|

As we can see, only a few pull requests have been reopened.

## [Conclusion](https://duckdb.org/2023/03/03/json.html#conclusion)

DuckDB tries to be an easy-to-use tool that can read all kinds of data formats. In the 0.7.0 release, we have added support for reading JSON. JSON comes in many formats and all kinds of schemas. DuckDB's rich support for nested types (`LIST`, `STRUCT`) allows it to fully “shred” the JSON to a columnar format for more efficient analysis.
