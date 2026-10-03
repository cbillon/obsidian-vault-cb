---
# Postgres AT TIME ZONE 'UTC' does NOT do what you think it does

---

_And why you may need to repeat `AT TIME ZONE 'UTC'` twice._

SUMMARY

- `AT TIME ZONE 'UTC'` converts the data type from `timestamptz` to `timestamp` ([example](https://onecompiler.com/postgresql/454fuzgeu?ref=tanin.nanakorn.com)).
- Now you are inadvertently using the `timestamp without time zone` (aka `timestamp`) data type, which is [markedly discouraged](https://wiki.postgresql.org/wiki/Don't_Do_This?ref=tanin.nanakorn.com#Don't_use_timestamp_\(without_time_zone\)_to_store_UTC_times).
- There are tons of footguns with `timestamp`.
- For example, the equality of `timestamp` and `timestamptz` will always be false..... unless your Postgres' default timezone is UTC. [Read more](https://news.ycombinator.com/item?id=49875982&ref=tanin.nanakorn.com).
- Adding a month with `+ INTERVAL '1 months'` is timezone-dependent. If your product operates in UTC, you must ensure the timezone is in UTC before adding a month with `<timestamptz_column> AT TIME ZONE 'UTC' + INTERVAL '1 months'`.... but now the result is a `timestamp`, not a `timestamptz`.
- To convert the above back to `timestamptz` in order to avoid issues with equality, you will need to invoke `AT TIME ZONE 'UTC` again. The final form is: `(<timestamptz_column> AT TIME ZONE 'UTC' + INTERVAL '1 months') AT TIME ZONE 'UTC'`

Postgres has these 2 timestamp types: `timestamp` and `timestamptz`.

`timestamp` doesn't contain the timezone information. Technically, it doesn't represent a time in the real world. Thinking about it deeply, when we say "5pm" in the real world, it actually means "5pm in our timezone". Even Postgres Wiki says: [Don't use timestamp without time zone](https://wiki.postgresql.org/wiki/Don't_Do_This?ref=tanin.nanakorn.com#Don't_use_timestamp_\(without_time_zone\)_to_store_UTC_times).

I already know this for quite a while. What I didn't realize is that I might not be able to completely avoid `timestamp without time zone`.

In many time-series products, you may be building a chart that shows the deltas from month to month and have a SQL like below:

```
SELECT
  a.month_start,
  (a.value - b.value) AS delta
FROM data a
JOIN data b
ON a.month_start = b.month_start + INTERVAL '1 months'
```

Now the first issue is: **adding months is actually timezone-dependent.**

For example, if your timezone is PT, and your product works in UTC, `2026-03-01 00:00:00+00` is equal to `2026-02-28 16:00:00-08`. Your machine's default setting is PT. This means Postgres will use `2026-02-28 16:00:00-08`, and `'2026-02-28 16:00:00-08'::timestamptz + INTERVAL '1 months'` will yield `2026-03-28 16:00:00-08`. That's not what we want.

Since we operate in UTC, we should convert the timestamp to UTC before adding a month. We modify the SQL to be:

```
SELECT
  a.month_start,
  (a.value - b.value) AS delta
FROM data a
JOIN data b
ON a.month_start = (b.month_start AT TIME ZONE 'UTC') + INTERVAL '1 months'
```

It turns out the SQL still doesn't produce the correct result because:

1. `'2026-02-28 16:00:00-08'::timestamptz AT TIME ZONE 'UTC'` will convert the data type from `timestamptz` (with time zone) to `timestamp` (_without_ time zone) and produces `2026-https://github.com/bluepuma77/traefik-best-practice/tree/main/docker-traefik-dnschallenge03-01 00:00:00` (Notice there's no `+00` at the end). [Example](https://onecompiler.com/postgresql/454fuzgeu?ref=tanin.nanakorn.com)
2. Comparing a `timestamp` and `timestamptz` will always result in `false`.
    1. It makes sense because `timestamp` is not a real-world point in time. The comparison is technically absurd.
    2. It gets even more confusing because, if you perform `EXTRACT(EPOCH FROM <timestamp>)`, they will both produce the same value.

In order to fix it, we have to convert `timestamp` back to `timestamptz` with another `AT TIME ZONE 'UTC'` like below:

```
ON a.month_start = ((b.month_start AT TIME ZONE 'UTC') + INTERVAL '1 months') AT TIME ZONE 'UTC'
```

Now the equality will work as you expect it to work.

[  
](https://bookofrevenue.com/blog)