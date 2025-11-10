---
link: https://brojonat.com/posts/window-functions/
byline: Jon
site: Jon Brown's Webpage
date: 2024-08-19T01:00
excerpt: |-
  Hello friends! Today&rsquo;s post is going to be about window functions (specifically with Postgres).
  Here&rsquo;s the docs and tutorial.
tags:
  - postgresql
  - window-function
slurped: 2025-11-08T11:41
title: Window Functions
---

Hello friends! Today’s post is going to be about window functions (specifically with Postgres).

Here’s the [docs](https://www.postgresql.org/docs/current/functions-window.html) and [tutorial](https://www.postgresql.org/docs/current/tutorial-window.html). Give them a read. Like most Postgres docs, they’re quite good.

What is a window function? Well, you can think of it like a convolution: there’s a “window” (i.e., a kernel), and each row gets a new value by applying some aggregate function to all the rows in the window. Unlike traditional aggregate functions, all the rows are preserved. Row order is also important when it comes to window functions; the aggregator “knows” which rows have already been handled. This allows for some pretty powerful functions. Let’s jump into it.

### Setting up a Schema[⌗](https://brojonat.com/posts/window-functions/#setting-up-a-schema)

Here’s the schema we’re working with (it’s just a bunch of “things” that belong to groups; each “thing” has an associated value):

```
CREATE TABLE things (
	id SERIAL PRIMARY KEY,
	item VARCHAR(255),
	collection VARCHAR(255),
	value INT
);

INSERT INTO things (item, collection, value)
VALUES
    ("queen", "chess", 9),
    ("rook", "chess", 5),
    ("knight", "chess", 3),
    ("bishop", "chess", 3),
    ("pawn", "chess", 1),
    ("chase", "banks", 3000),
    ("bofa", "banks", 2000),
    ("wells fargo", "banks", 45),
    ("citi", "banks", 1000),
    ("tacoma", "cars", 50000),
    ("prius", "cars", 2000),
    ("lighning", "cars", 25000),
    ("viper", "cars", 35000),
    ("mini", "cars", 10000),
    ("astrovan", "cars", 500),
    ("walnut", "wood", 20),
    ("oak", "wood", 15),
    ("wenge", "wood", 45),
    ("maple", "wood", 17),
    ("pine", "wood", 12),
    ("mdf", "wood", 3);

SELECT * FROM things;
```

```
id|item       |collection|value|
--+-----------+----------+-----+
 1|queen      |chess     |    9|
 2|rook       |chess     |    5|
 3|knight     |chess     |    3|
 4|bishop     |chess     |    3|
 5|pawn       |chess     |    1|
...continues on...
```

### Why Window Functions?[⌗](https://brojonat.com/posts/window-functions/#why-window-functions)

There’s many cases when you want to do something _like_ a `GROUP BY`, but the fact that the traditional aggregation reduces the number of rows output is undesirable. Yes, you can get around this with a subquery or other gymnastics, but it’s nice that window functions make it trivial. Plus window functions _tend_ to be more performant than their subquery counterparts, and they unlock some pretty cool use cases.

### Simple Window Function[⌗](https://brojonat.com/posts/window-functions/#simple-window-function)

Many vanilla aggregate functions can be used as window functions. A window function call always contains an `OVER` clause directly following the window function’s name and argument(s). This is what syntactically distinguishes it from a normal function or non-window aggregate. It’s worth noting that you can omit the `PARTITION BY` part of the clause and all rows will be treated as a single partition. Here’s an example window function that computes the max value for each collection:

```
SELECT *, MAX(value) OVER (PARTITION BY collection) AS "group max"
FROM things
ORDER BY "group max" DESC, value DESC;
```

```
id|item       |collection|value|group max|
--+-----------+----------+-----+---------+
10|tacoma     |cars      |50000|    50000|
13|viper      |cars      |35000|    50000|
12|lighning   |cars      |25000|    50000|
14|mini       |cars      |10000|    50000|
11|prius      |cars      | 2000|    50000|
15|astrovan   |cars      |  500|    50000|
 6|chase      |banks     | 3000|     3000|
 7|bofa       |banks     | 2000|     3000|
 9|citi       |banks     | 1000|     3000|
 8|wells fargo|banks     |   45|     3000|
18|wenge      |wood      |   45|       45|
16|walnut     |wood      |   20|       45|
19|maple      |wood      |   17|       45|
17|oak        |wood      |   15|       45|
20|pine       |wood      |   12|       45|
21|mdf        |wood      |    3|       45|
 1|queen      |chess     |    9|        9|
 2|rook       |chess     |    5|        9|
 4|bishop     |chess     |    3|        9|
 3|knight     |chess     |    3|        9|
 5|pawn       |chess     |    1|        9|
```

Pretty straightforward. Ok, lets move on to something more interesting.

### Useful Window Functions[⌗](https://brojonat.com/posts/window-functions/#useful-window-functions)

Something to keep in mind is that the querier “knows” which rows have come before and go after the current row. As a result, there’s lots of useful window functions, especially when it comes to getting a rows position in the overall _distribution_ of the group:

- `ROW_NUMBER`: Returns the number of the current row within its partition, counting from 1.
- `RANK`: Returns the rank of the current row, with gaps; that is, the row_number of the first row in its peer group.
- `DENSE_RANK`: Returns the rank of the current row, without gaps; this function effectively counts peer groups.
- `PERCENT_RANK`: Returns the relative rank of the current row, that is (rank - 1) / (total partition rows - 1). The value thus ranges from 0 to 1 inclusive.
- `CUME_DIST`: Returns the cumulative distribution, that is (number of partition rows preceding or peers with current row) / (total partition rows). The value thus ranges from 1/N to 1.
- `NTILE`: Returns an integer ranging from 1 to the argument value, dividing the partition as equally as possible.

There’s also some neat functions for working with adjacent rows (i.e., computing derivatives):

- `LAG`: Returns value evaluated at the row that is offset rows before the current row within the partition; if there is no such row, instead returns default (which must be of a type compatible with value). Both offset and default are evaluated with respect to the current row. If omitted, offset defaults to 1 and default to `NULL`.
- `LEAD`: Returns value evaluated at the row that is offset rows after the current row within the partition; if there is no such row, instead returns default (which must be of a type compatible with value). Both offset and default are evaluated with respect to the current row. If omitted, offset defaults to 1 and default to `NULL`.
- `FIRST_VALUE`: Returns value evaluated at the row that is the first row of the window frame.
- `LAST_VALUE`: Returns value evaluated at the row that is the last row of the window frame.
- `NTH_VALUE`: Returns value evaluated at the row that is the n’th row of the window frame (counting from 1); returns NULL if there is no such row.

So, the window “knows” the order of rows, but how? This is where it gets a _little_ intricate. You can specify an `ORDER BY` clause in the `OVER` clause (N.B.: no relation to the `ORDER BY` clause of the query itself). Rows that are not distinct when considering only the `ORDER BY` columns are said to be peers. The four ranking functions (including `CUME_DIST`) are defined so that they give the same answer for all rows of a peer group. Here’s what `ORDER BY` looks like in action when we get the value ranking (higher value is better rank) of each row within its collection:

```
SELECT
	id,
	item,
	collection,
	value,
	MAX(value) OVER (PARTITION BY collection) AS "max value",
	RANK() OVER (PARTITION BY collection ORDER BY value DESC) AS "rank"
FROM things
ORDER BY "max value" DESC, "rank";
```

```
id|item       |collection|value|max value|rank|
--+-----------+----------+-----+---------+----+
10|tacoma     |cars      |50000|    50000|   1|
13|viper      |cars      |35000|    50000|   2|
12|lighning   |cars      |25000|    50000|   3|
14|mini       |cars      |10000|    50000|   4|
11|prius      |cars      | 2000|    50000|   5|
15|astrovan   |cars      |  500|    50000|   6|
 6|chase      |banks     | 3000|     3000|   1|
 7|bofa       |banks     | 2000|     3000|   2|
 9|citi       |banks     | 1000|     3000|   3|
 8|wells fargo|banks     |   45|     3000|   4|
18|wenge      |wood      |   45|       45|   1|
16|walnut     |wood      |   20|       45|   2|
19|maple      |wood      |   17|       45|   3|
17|oak        |wood      |   15|       45|   4|
20|pine       |wood      |   12|       45|   5|
21|mdf        |wood      |    3|       45|   6|
 1|queen      |chess     |    9|        9|   1|
 2|rook       |chess     |    5|        9|   2|
 3|knight     |chess     |    3|        9|   3|
 4|bishop     |chess     |    3|        9|   3|
 5|pawn       |chess     |    1|        9|   5|
```

You can also see an example here of the behavior for a “peer group” as mentioned above: both the knight and bishop have a rank of 3. See the details of `RANK` vs. `DENSE RANK` for how the presence of peer groups effects subsequent rows.

Also note that the `ORDER BY` clause has an optional `NULLS FIRST | NULLS LAST` component, which can be useful depending on what you’re trying to do.

### What is a Window Frame?[⌗](https://brojonat.com/posts/window-functions/#what-is-a-window-frame)

The rows considered by a window function are those of the “virtual table” produced by the query’s `FROM` clause as filtered by its `WHERE`, `GROUP BY`, and `HAVING` clauses if any. A window function has a “partition”, which is all rows in the window. There is also a “window frame”: for each row, there is a set of rows within its partition called its window frame. **Some window functions act only on the rows of the window frame, rather than of the whole partition**. By default, if `ORDER BY` is supplied then the frame consists of **all rows from the start of the partition up through the current row, plus any following rows that are equal to the current row** according to the `ORDER BY` clause. When `ORDER BY` is omitted the default frame consists of all rows in the partition. Let’s use this behavior to get the cumulative sum of values by collection ordered by `value` (again, note the chess collection behavior):

```
SELECT
	id,
	item,
	collection,
	value,
	SUM(value) OVER (PARTITION BY collection ORDER BY value ASC) AS "sum"
FROM things
ORDER BY "collection", "sum" ASC;
```

```
id|item       |collection|value|sum   |
--+-----------+----------+-----+------+
 8|wells fargo|banks     |   45|    45|
 9|citi       |banks     | 1000|  1045|
 7|bofa       |banks     | 2000|  3045|
 6|chase      |banks     | 3000|  6045|
15|astrovan   |cars      |  500|   500|
11|prius      |cars      | 2000|  2500|
14|mini       |cars      |10000| 12500|
12|lighning   |cars      |25000| 37500|
13|viper      |cars      |35000| 72500|
10|tacoma     |cars      |50000|122500|
 5|pawn       |chess     |    1|     1|
 3|knight     |chess     |    3|     7|
 4|bishop     |chess     |    3|     7|
 2|rook       |chess     |    5|    12|
 1|queen      |chess     |    9|    21|
21|mdf        |wood      |    3|     3|
20|pine       |wood      |   12|    15|
17|oak        |wood      |   15|    30|
19|maple      |wood      |   17|    47|
16|walnut     |wood      |   20|    67|
18|wenge      |wood      |   45|   112|
```

Note that if you need to filter based on the result of a window function, you’ll need to use a subquery like:

```
SELECT * FROM (
    SELECT *, MAX(value) OVER (PARTITION BY collection) AS "group max"
    FROM things
    ORDER BY "group max" DESC, value DESC
) WHERE "group max" > 30000;
```

You can also specify your own window frame! If you need the whole window frame, you can simply omit the `ORDER BY`, or you can use `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`. The syntax of the window frame specification is **one of the following**:

```
{ RANGE | ROWS | GROUPS } frame_start [ frame_exclusion ]
{ RANGE | ROWS | GROUPS } BETWEEN frame_start AND frame_end [ frame_exclusion ]
```

where `frame_start` and frame_end can be one of

```
UNBOUNDED PRECEDING
offset PRECEDING
CURRENT ROW
offset FOLLOWING
UNBOUNDED FOLLOWING
```

and `frame_exclusion` can be one of

```
EXCLUDE CURRENT ROW
EXCLUDE GROUP
EXCLUDE TIES
EXCLUDE NO OTHERS
```

The frame can be specified in `RANGE`, `ROWS` or `GROUPS` mode. In `RANGE` or `GROUPS` mode, a frame_start of `CURRENT ROW` means the frame starts with the current row’s first peer row (a row that the window’s `ORDER BY` clause sorts as equivalent to the current row), while a frame_end of CURRENT ROW means the frame ends with the current row’s last peer row. In `ROWS` mode, `CURRENT ROW` simply means the current row.

In the `offset PRECEDING` and `offset FOLLOWING` frame options, the offset must be an expression not containing any variables, aggregate functions, or window functions. The meaning of the offset depends on the frame mode:

- In `ROWS` mode, the offset must yield a non-null, non-negative integer, and the option means that the frame starts or ends the specified number of rows before or after the current row.
    
- In `GROUPS` mode, the offset again must yield a non-null, non-negative integer, and the option means that the frame starts or ends the specified number of peer groups before or after the current row’s peer group, where a peer group is a set of rows that are equivalent in the `ORDER BY` ordering. (There must be an `ORDER BY` clause in the window definition to use `GROUPS` mode.)
    
- In `RANGE` mode, these options require that the `ORDER BY` clause specify exactly one column. The offset specifies the maximum difference between the value of that column in the current row and its value in preceding or following rows of the frame. The data type of the offset expression varies depending on the data type of the ordering column. For numeric ordering columns it is typically of the same type as the ordering column, but for datetime ordering columns it is an interval. For example, if the ordering column is of type date or timestamp, one could write `RANGE BETWEEN '1 day' PRECEDING AND '10 days' FOLLOWING`. The offset is still required to be non-null and non-negative, though the meaning of “non-negative” depends on its data type.
    

Note that the distance to the end of the frame is limited by the distance to the end of the partition, so that for rows near the partition ends the frame might contain fewer rows than elsewhere.

The frame_exclusion option allows rows around the current row to be excluded from the frame, even if they would be included according to the frame start and frame end options. `EXCLUDE CURRENT ROW` excludes the current row from the frame. `EXCLUDE GROUP` excludes the current row and its ordering peers from the frame. `EXCLUDE TIES` excludes any peers of the current row from the frame, but not the current row itself. `EXCLUDE NO OTHERS` simply specifies explicitly the default behavior of not excluding the current row or its peers.

### Example: Find the median,[⌗](https://brojonat.com/posts/window-functions/#example-find-the-median)

Let’s find the median in each collection. This is a notoriously annoying thing to do in SQL without the use of window functions (and obviously `percentile_cont`). Here it is:

```
SELECT collection, AVG(value) AS "median"
FROM (
	SELECT
		id, item, collection, value,
    	ROW_NUMBER() OVER (PARTITION BY collection ORDER BY value ASC) AS "row_id",
    	COUNT(1) OVER (PARTITION BY collection) AS "count"
	FROM things
)
WHERE "row_id" BETWEEN "count"/2.0 and "count"/2.0 + 1
GROUP BY "collection"
```

```
collection|median |
----------+-------+
banks     | 1500.0|
cars      |17500.0|
chess     |    3.0|
wood      |   16.0|
```

It’s…not too bad? The only clever thing is the fact that you can use `WHERE row_id BETWEEN "count"/2.0 AND "count"/2.0+1` to handle both even/odd number of row cases. In the odd case, this is a single row; in the even case, you’ll get two rows. The `AVG` gives you the correct value of the median in either case.