---
link: https://bookofrevenue.com/blog/6aa8d8b292d31a00012a8642/identify-slow-postgres-query
tags:
  - postgre
---
****
A simple way to identify slow Postgres SQLs
I've just learned probably the simplest way to identify slow Postgres queries in your application.

Postgres has the built-in extension named `pg_stat_statements` that tracks the executed SQLs, their execution times, and other things.

First of all, you can enable the extension with: `CREATE EXTENSION pg_stat_statements;`. It's built-in, so no installation is needed.

Now you need to run 2 SQLs to be able to use this extension:

- `ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';` – I honestly don't know what it is. It's the shared library we have to preload. It's something we have to do.
- `ALTER SYSTEM SET "pg_stat_statements"."track" = 'all';` – this is to show the full SQLs in the `pg_stat_statements` table

There is another way to configure these 2 configs through command-line options, but I find using command-line option is complex when using PaaS like Render/Heroku.

Then, you let your application run for a bit, so `pg_stat_statements` can record some executed SQLs.

Then, you can query the `pg_stat_statements` table to see the execution times of the SQLs. Here's a convenient SQL to get you started:

```
SELECT
    calls,
    total_exec_time,
    mean_exec_time,
    query
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 5;
```

The above shows the top SQLs ranked by their largest average execution times.

Now you can see which SQL is slow and start creating an index for it.


I've just learned probably the simplest way to identify slow Postgres queries in your application.

Postgres has the built-in extension named `pg_stat_statements` that tracks the executed SQLs, their execution times, and other things.

First of all, you can enable the extension with: `CREATE EXTENSION pg_stat_statements;`. It's built-in, so no installation is needed.

Now you need to run 2 SQLs to be able to use this extension:

- `ALTER SYSTEM SET shared_preload_libraries = 'pg_stat_statements';` – I honestly don't know what it is. It's the shared library we have to preload. It's something we have to do.
- `ALTER SYSTEM SET "pg_stat_statements"."track" = 'all';` – this is to show the full SQLs in the `pg_stat_statements` table

There is another way to configure these 2 configs through command-line options, but I find using command-line option is complex when using PaaS like Render/Heroku.

Then, you let your application run for a bit, so `pg_stat_statements` can record some executed SQLs.

Then, you can query the `pg_stat_statements` table to see the execution times of the SQLs. Here's a convenient SQL to get you started:

```
SELECT
    calls,
    total_exec_time,
    mean_exec_time,
    query
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 5;
```

The above shows the top SQLs ranked by their largest average execution times.

Now you can see which SQL is slow and start creating an index for it.
