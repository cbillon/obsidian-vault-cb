---
link: https://www.softwareandbooz.com/introducing-pgnow-indexing
site: Software and Booz
date: 2025-04-09T19:54
excerpt: A few weeks ago I wrote about the introduction of pgNow, a free, cross-platform desktop application from Redgate that is designed to assist users who might lack extensive Postgres experience or a r…
slurped: 2026-02-21T10:59
title: "Introducing pgNow: Indexing"
tags:
  - postgresql
  - monitoring
---

A few weeks ago [I wrote about the introduction of pgNow](https://www.softwareandbooz.com/introducing-pgnow/), a free, cross-platform desktop application from Redgate that is designed to assist users who might lack extensive Postgres experience or a robust monitoring solution. It provides real-time data on queries, indexes, high-value configuration settings, and vacuuming setup and efficiency to help you pinpoint and tackle performance issues in your Postgres clusters.

In that first article, I shared how pgNow can be a lifesaver when you need immediate performance insights, highlighting features like query tuning and current activity monitoring. The tool’s ability to take periodic snapshots of query activity and spotlight active sessions has already been a significant help for early users.

Today, I wanted to look at another area of information that pgNow can help you explore during times of performance degradation or even as part of a regular database maintenance and hygiene: the Indexing tab.

([Download pgNow](https://rd.gt/3FlZB8k) today to follow along!)

## Unlocking the Power of Indexing

Database tuning is an ongoing need in any environment, whether as part of a performance incident or just regular maintenance. Proper indexing is one of the areas we all need to keep an eye on. Regardless of the relational database you use, a few things relating to indexes generally hold true:

- If a table is scanned a lot without using an index, there may be an opportunity to improve queries with the proper index
- If an index is rarely or never used to look up data, it’s likely a good candidate to drop and save the space and overhead of maintaining it
- Duplicate indexes with the identical leading column(s) may both be used by the planner (for whatever reason) but usually just one will do the job and provide better efficiency

And for PostgreSQL specifically:

- Index size, particularly given the various types of indexes that PostgreSQL supports, can be larger and more maintenance heavy then expected, which may present an opportunity to choose a better index type or filter it to reduce size and overhead.

Thankfully, and with no surprise, PostgreSQL provides many metrics that track table and index usage to help with this work through the `pg_stat_user_indexes`, `pg_statio_user_indexes`, and `pg_stat_user_tables` statistics views. Querying and organizing them on your own can be time consuming, especially when you’re in a jam. Relating the two sets of data together in a neat and clean way may be more challenging for inexperienced users.

Fortunately, the **Indexing** tab in pgNow provides most this information in a user-friendly way to get you started quickly.

## The Tables View

When you come do the Indexing tab, it might seem a bit odd that the first thing pgNow shows you is a list of all user tables in each database and not all indexes. It turns out that starting with the table usage statistics is a time-tested method of identifying tables that are large, heavily used, or both… **_and_** show that they have a lot of sequential scans. When a table has a significant number of sequential scans, that’s a leading indicator that an index might help the planner use the table data more efficiently. These are usually the tables where index work will have the most return on investment.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image.png?ssl=1)

It’s worth noting that some of the numbers in the following screenshots are pretty big. This is a sample application database that has been purposefully setup to demonstrate things like missing indexes and bad vacuuming configurations (we’ll talk about vacuuming in another article soon!). That also makes it pretty easy to spot some tables that could use some help. Convenient, right?!

Initially pgNow will sort this table by total tuples returned by sequential scans. This is a good way to spot the tables that have a lot of I/O that isn’t using an index, and therefore is an opportunity for improvement. Depending on your workload, you might want to sort by a few other metrics to draw out different things to attack.

### **Table Size**

It may seem obvious that your larger tables will often have the most I/O. That certainly isn’t a hard and fast rule, but probably worth exploring. However, what a lot of people new to Postgres might not realize is that tables can quickly bloat with dead tuples if vacuuming isn’t set up correctly. Therefore, sorting by table size descending (largest to smallest) may show some tables near the top that you wouldn’t expect to be that big.

For instance, when I first returned to using Postgres at a former company, we were investigating an issue with one of our core application functions becoming slow. During my investigation, I was shocked to find that the related table was over 5GB in size for a few hundred thousand rows of data that I knew should only be a few hundred MB at best. It turns out that we didn’t understand vacuum tuning at all and we just had a significant number of dead tuples.

### **Index Scans**

If you sort the tables by index scans ascending (smallest to largest), the tables that aren’t using any indexes to return data for queries become quickly evident. Now remember, Postgres often leans in favor of sequential scans for smaller tables unless you tune other settings correctly. So don’t be surprised if you have a table with a few thousand or tens of thousands of rows that has very low index usage. It doesn’t mean you can’t improve queries with indexes (again, there are other settings to tune which may help), but these might not be the tables you want to focus on for query tuning.

Instead, you want to look for tables that have a significant number of sequential scans but low index scans to focus on. Once a table has a few hundred MB of data (very general, gut recommendation) with very little index usage and disproportionate sequential scans, it’s time to investigate why the indexes aren’t being used as you expected. These tend to be the low hanging performance tuning opportunities that can be improved with appropriate indexes.

**Average tuples per sequential scan**

Finally, I want to draw your attention to the last column for table metrics, “Average tuples per sequential scan”. This is a final opportunity to find tables that would likely benefit from investigating query patterns and index improvements.

Sequential scans are a given in Postgres. You will never be able to eliminate all sequential scans on all tables. However, if you have a significant number of tuples being returned with every sequential scan and there are a lot of scans happening on that table, you’re likely wasting CPU cycles and disk I/O that could be used for other tasks.

In the sample data that you see in these screenshots, you’ll see two examples at the top of the table grid when I sort by average tuples per sequential scan – `rental` and `payment`.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image.jpg?ssl=1)

In this case I can see that the `payment` table returns more tuples for every sequential scan, however, there have only been three (3) sequential scans since statistics were last reset months ago. Instead, I can clearly see that this table essentially always uses the indexes to return tuples based on the current workload. (and yes, that’s on purpose and unlikely to happen in real-life applications this cleanly)

The second table, `rental`, is exactly the opposite. Although it also uses indexes ~34x more than sequential scans, those sequential scans are very costly. Since statistics have been reset, queries have returned nearly 9 trillion tuples by scanning the table, at an average of nearly 7 million tuples per scan. 😱🤯 Ummm… think that’s worth investigating a bit?

## Drilling down for further investigation

Once you’ve identified a table that might be used inefficiently, it’s time to see how indexes are being used (or not used) and any queries that reference that table. This is another helpful strength of pgNow.

Expanding any table row in reveals two more pieces of information; Indexes and Queries.

### Indexes

pgNow will list any indexes that exist on the selected table, showing the size of the index, how often it has been scanned (the good kind of scan), the last time it was used in a query, and some metrics about the tuples returned and how many pages in buffer or on disk were accessed through this index. Expanding any index row will display the index definition to see exactly what column(s) are in the index and what type of index it is.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image-1.jpg?ssl=1)

There are a few things you can determine with this information which may be helpful in your tuning investigation.

**Unused indexes**

pgNow doesn’t  specifically call out unused indexes because it requires more information over a longer timeframe to credibly suggest that an index may be unused. However, if you see indexes that have no recorded index scans, dig a little deeper.

- Based on the definition of the index, do you expect it to be used?
- Was it used in the past but isn’t used now because of a query or application change?
- Is it unused because it’s a duplicate index?

Again, you know your database and application best. Rely on that information to help you identify and track indexes that might be unused.

**Index Shared Buffer Usage**

Recall that `shared_buffers` is the configurable amount of RAM used to store pages of data for faster access. Both table data pages and index pages are both stored here as much as possible to ensure the fastest data access and modification possible. If your indexes are large and competing for space with other table and index data pages, it can result in many index scans requiring data from outside of shared buffers which is less efficient. Postgres gives us this information in the `pg_statio_user_indexes` view.

From the secondary index view under each table, we can get an idea of how often the pages of index information are available in `shared_buffers` and how often Postgres has to load an index page from disk to look things up. Remember, all database platforms tend to perform best when the data they need most often is available in memory (`shared_buffers` for PostgreSQL).

Just as with data pages, when an index is accessed and the data is available in `shared_buffers`, PostgreSQL calls that a hit (`idx_blks_hit`). When the page had to be accessed from disk first, PostgreSQL records that as a read (`idx_blks_read`). The ratio of these two numbers gives us an indication of how efficient Postgres can access the index data.

Looking at the same two indexes that are highlighted in the image above, we can see that the `rental_rental_period_idx` index appears to be held in memory most of the time and very efficient at a >99% cache hit ratio. The `rental_store_id_idx` index, however, only finds index pages in memory 38% of the time and often has to pull/read pages from disk. Over the lifetime of these cumulative statistics, this index on `store_id` has read nearly 6 billion tuples from the index. If we increase `shared_buffers`, we’ll likely find that more of this index can stay in memory longer, improving the overall query efficiency.

And honestly, if you’re seeing `shared_buffers` issues with your actively used indexes, it’s likely that you’re having the same issue with your data. Look at both ratios and track it over time as you increase `shared_buffers` given recommendations elsewhere, either in the **Settings** tab of pgNow, in documentation, in [my thoughts from this older post](https://www.softwareandbooz.com/postgresql-to-sql-server-the-first-four-settings/), or even in the configuration chapter in the book [Grant Fritchey and I wrote about learning PostgreSQL](https://rd.gt/4ian4HR) (it’s available for free as a PDF at that link!)

### Queries

Finally, when you inspect the details of a table in pgNow on the indexing tab, you’ll see a list of parameterized queries from `pg_stat_statements` that reference the table in focus.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image-4.jpg?ssl=1)

Showing this information in context of the table and the indexes is a helpful way to focus on queries that might be ripe for optimization. I tend to look at these queries by average time per execution to get a sense of which queries typically take a long time compared to the others and have more than just a few executions.

We can then expand a query to see:

- The parameterized version of the query as saved in `pg_stat_statements`
- A list of tables that are referenced in the query
- A generic execution plan when available

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image-2.jpg?ssl=1)

In my opinion, there are two interesting things to take away from this section of the indexing view.

**Last vacuum and analyze**

A healthy PostgreSQL server is configured in a way that maintains table information on a regular basis. If the autovacuum and autoanalyze thresholds are not configured correctly ([check this blog post again](https://www.softwareandbooz.com/postgresql-to-sql-server-the-first-four-settings/) or get the [free PDF of our book](https://rd.gt/4ian4HR)), then dead tuples aren’t cleaned up which leads to table bloat and more overall I/O to satisfy queries. Also, if the data isn’t analyzed on a regular basis, then table statistics get out of date and the planner might start to choose inefficient plans based on outdated information. If you’re having a problem with specific queries and you notice that the tables involved, especially larger tables, haven’t been analyzed or vacuumed recently (that’s a relative term and different for each workload/dataset), then that’s a good place to start.

Furthermore, pgNow prompts you if the table has custom values set for autovacuum and autoanalyzer to clue you in to investigate further. If the custom settings are significantly delaying these processes from running on a regular basis, you may need to revisit them. Maybe they were good thresholds when your application had less data, but as the database has grown they need to be tuned further.

**Generic query plan**

Finally, as an initial attempt to provide some insights into what the query plan may look like, pgNow will display a generic plan for queries when it can. This is not the actual query plan that you need to use EXPLAIN ANALYZE to get. pgNow cannot get that plan without knowing the actual values or pulling the plan out of the logs if you had `autoexplain` configured. Because of the permissions and configuration required, pgNow can’t get that data.

However, asking Postgres to provide a “best guess” plan based on generic parameters at least gives you a starting point to see what might be happening.

- In general, are the indexes used that I expect to be used?
- Are there any sequential scans on tables that I would expect an index to be used for?
- Is there an inefficient join strategy based on the estimated row/cost values?

Again, this isn’t the real plan, but it can be helpful just to have a beginning idea of what the planner thinks is a reasonable plan based on the statistics it has access to at the time.

## All Indexes View

On the **Indexing** tab there is one more way to view index data. If you want to look more wholistically at all of the indexes in your database, you can select the “All Indexes” radio button next to the **Refresh** button. I primarily see this as a helpful view to see which indexes are used the most, find unused indexes, spot overall efficiency in `shared_buffers` usage, and to get an idea of how big your various indexes are.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/04/image-3.jpg?ssl=1)

There are different reasons to look at the indexing data from each view. As with all things tech related, **_it depends_** what your need or goal is when accessing this data. Thankfully, pgNow gives you the options either way.

## Download the Preview and provide feedback

Database maintenance and performance tuning are constant tasks for DBAs and developers. PostgreSQL provides a plethora of information and metrics to help identify areas for improvement, especially when there is an ongoing issue.

Having the ability to quickly look more wholistically at the table and index metrics within your database can help you quickly identify inefficiencies in your indexing, server memory (`shared_buffers`) configuration, and overall maintenance task performance. pgNow is a helpful tool to bring this data alive for you when you’re not able to quickly query the data for yourself… or you just like having a simple UI to help you.

Give pgNow a try and [download the preview today](https://rd.gt/3FlZB8k) and share your feedback with the team through the “Give Feedback” button. Your input will help improve the tool and make it even more valuable for the Postgres community. Download pgNow today and take advantage of its powerful features to optimize your databases.