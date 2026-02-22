---
link: https://www.softwareandbooz.com/introducing-pgnow/
site: Software and Booz
date: 2025-03-10T14:09
excerpt: pgNow is a free, cross-platform desktop tool created by Redgate that helps you identify key performance metrics and configuration optimizations in your running Postgres instance. Available now as a…
slurped: 2026-02-21T10:55
title: "Introducing pgNow: A free, point-in-time diagnostic tool for PostgreSQL"
tags:
  - postgresql
  - monitoring
---

pgNow is a free, cross-platform desktop tool created by Redgate that helps you identify key performance metrics and configuration optimizations in your running Postgres instance. [Available now as a public preview application](https://rd.gt/3FlZB8k), it’s designed to help when you’re in a pinch and don’t have the Postgres experience or monitoring solution already in place to help identify why your server or database is experiencing a degradation in performance. Even in its current preview offering, pgNow is a helpful front-line tool for troubleshooting your Postgres cluster. And I couldn’t be more excited to share it with you.

Let me explain why.

## We’re using Postgres!?!

I’ve been stuck in those situations that feel helpless, particularly as a new user to Postgres, and I wished at the time that there was a tool like pgNow to help point me in the right direction.

In 2018, I took a new job after working with SQL Server for nearly 15 years. I _thought_ it was clear to the hiring company that I was coming as a Microsoft data platform professional to help solve big data problems… using SQL Server. I had literally just released my first Pluralsight course on Triggers and Functions for SQL Server. I was kind of “all in” as it were.

Imagine my surprise when I was handed a PostgreSQL connection string on the first day. I hadn’t connected to a Postgres database in nearly 15 years, and I was very confused about where all the SQL Server instances were hiding. It turns out that there weren’t any, as in the intervening time between accepting the job and starting, the team had released the updated application which replaced SQL Server with PostgreSQL. They had just forgotten to mention that change during the interview process. 😱😂

The next 6-12 months was a learning journey I hadn’t expected. But those experiences have allowed me to become an advocate for PostgreSQL and share my ever-growing knowledge with others in hopes of reducing the time it takes to get up and running on Postgres. (Grant Fritchey and I [just released a book](https://rd.gt/4ian4HR) with that exact purpose in mind!)

Trust me, it wasn’t without a lot of trial and ~~screaming~~ error.

## Learning SQL isn’t the main problem

It turns out that learning new ways of writing SQL isn’t the difficult part for most data professionals that transition to another platform like PostgreSQL. Instead, what I and so many new users to Postgres struggle with is knowing how to access actionable data that can point out problematic queries and server configuration issues.

It’s not that Postgres doesn’t provide enough metrics. To the contrary, Postgres comes with a large and growing set of metrics to help you figure out what’s happening in your cluster or database. The problem is that there are no native, visual tools to help coalesce all that data together into actionable information to improve a degradation in performance. You may suspect that a query is not performing well and is impacting other processes, but out of the box you have to know which views to query to begin putting the pieces together.

This, it turns out, is a feature, not a bug. There is a long history of code contributors and project maintainers providing users with all the necessary data, but not specifically deciding the best way to use that data to accomplish any particular task. As an open-source project, the assumption is that others will contribute tools that solve a specific problem with this data because there are usually many possible approaches.

And so, when I first started using Postgres again in 2018, I often felt stuck when our database would suddenly increase in IOPS, taking the system down, and no “easy” way to start to debug it. Before ChatGPT, the help I got was only as good as my Google-foo.

I didn’t know that one of our tables which tracked hundreds of thousands of IOT devices suffered from serious table bloat because the VACUUM settings hadn’t been adjusted over the years. As it turns out, updating all rows with a new value for “last updated” was quickly impacting vacuum and other processes. Without more experience, I didn’t know what I didn’t know.

Fortunately, I’m in a different spot now, having had the opportunity to invest thousands of hours learning about and teaching Postgres. I’m in a much better position to help when a database isn’t performing well or when configuration settings probably need to be tuned in a given situation.

But I know so many people that haven’t had that time but are still being asked to learn Postgres, and they feel stuck just like I did when it comes to troubleshooting an emerging database problem.

pgNow is a free, [cross-platform desktop tool](https://rd.gt/3FlZB8k) created by Redgate that is focused on doing one job well; connecting to a Postgres cluster, querying real-time data about queries, indexes, high-value configuration settings, and vacuuming to help you find the issue that’s happening right now. It’s not a full-fledged monitoring tool that is intended to track data over a long period of time. But in most high-pressure situations where you don’t already have an enterprise monitoring solution for Postgres in place, pgNow will help you triage the situation and find where to start looking.

Let’s take a brief look at the main features of pgNow, saving some “deep dive” examples for future blogs.

### Query Analysis

Most of the time we first catch wind of a performance problem when users start complaining or alerts go off. It’s natural to immediately want to know what’s happening on the system right now, and which queries may be causing some issues.

In pgNow, this information is found in the Workload tab.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-2.png?ssl=1)

Here you’ll find a table of query data from `pg_stat_statements`. The list can be filtered and sorted to find exactly what you want to start looking at first. Once you locate a query for further investigation, you can expand the row to get even more information.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image.png?ssl=1)

Not only can we see the query that was run, but which tables are referenced by that query, along with a lot of detailed table metrics. In this example, you can see that I have a table with lots of dead tuples and some custom vacuum configuration settings. Seeing that information here, right next to the query, helps to shed light on a potential factor contributing to the effected query.

After that, you’ll also notice that pgNow provides a generic execution plan when it is able to (stored procedures, for instance, won’t produce a generic plan). This may not be the actual execution plan that ran with the query. That can only be captured with the auto_explain extension and a lot of other setup work. But it may be helpful to get an initial idea of some problem areas like indexes not being used or a loop join being chosen over a hash join.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-1.png?ssl=1)

### But wait, there’s more!

That information alone is super helpful, especially if you’re not fully up to speed on the Postgres cumulative statistics system. But, when you’re in a pinch and need to figure out which query is actually causing the problem _right now_, the “cumulative” part of the cumulative statistics system is a problem. Notice how all the numbers on the default “All Time” view are cumulative. Just because one of these queries has taken a long time in total over the life of a running Postgres instance, doesn’t mean it’s the query hogging all the resources **_now_**_._

For that, pgNow has included the ability to take periodic snapshots of query activity so that you can compare over time which queries are actually doing work now. This is **hugely** helpful! Without a quick tool like pgNow to do this work for you, you’d have to start querying `pg_stat_statements` periodically, saving the cumulative values, and then query the deltas over time. Most point-and-click desktop tools don’t do this for you out of the box.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-2.jpg?ssl=1)

This functionality alone is worth its weight in gold if you don’t have a lot of Postgres experience or other monitoring solutions set up for your applications and infrastructure. Alternatively, you can use the “Live” view and let pgNow periodically take snapshots for you to get a sense of where the load is coming from.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-1.jpg?ssl=1)

The beauty is that each of these views display the same information, allowing you to dig a little deeper into what’s happening with your current query performance.

## Current Activity

The second tab in pgNow is currently labeled “Sessions” and it shows you the information available from pg_stat_activity. These are the current open sessions, whether they are active or idle. Like the Workload tab, the grid of data can be filtered numerous ways.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-3.jpg?ssl=1)

In this example, taken from an active demo of the Bluebox database, you’ll notice that I have a lot of sessions that are active and waiting on BufferIO. This database is experiencing some major slowdowns right now and seeing this information quickly and clearly points to an IO issue, letting me know that I am probably over saturating the disks and need to do some further investigation.

Opening the details of any session provides both the query that is currently running on that connection (or last ran if the session is idle) and whether there are any blocking processes to deal with.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-4.jpg?ssl=1)

## PostgreSQL Configuration

For now, I want to touch on just one more tab in pgNow, leaving the rest for another blog post (but please explore all of the tabs on your own!).

The “Settings” tab provides a list of configuration checks and recommendations for your currently running Postgres cluster. Although Postgres has well over 300 possible configuration settings out of the box, we’ve worked hard to highlight the configuration options that are most often in need of some specific attention based on your workload. If you’ve ever heard a version of my “PostgreSQL for the SQL Server Developer and DBA” talk (or blog), the list in pgNow highlights the configurations I discuss often and a few others that the community generally encourages users to modify.

[![](data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iMSIgaGVpZ2h0PSIxIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciPjwvc3ZnPg==)](https://i0.wp.com/www.softwareandbooz.com/wp-content/uploads/2025/03/image-5.jpg?ssl=1)

Each setting will show a colored area indicating if the value is generally within recommended ranges or if it’s a setting that might need more attention. In this example screenshot, you can see that `random_page_cost` is still using the default value of 4 and could use some thoughtful attention.

## Download the Preview and provide feedback

As I said, pgNow is a new offering from Redgate, currently in public preview. It’s being provided with the understanding that although it already provides a huge amount of troubleshooting value, there’s more work to be done.

I’d like to encourage you to [download pgNow](https://rd.gt/3FlZB8k), try it on a few of your active databases, and provide feedback to the team through the big, blue “Give Feedback” button. Any feedback you provide will help to improve pgNow as we work towards a stable, 1.0 release. And, if you’re willing to talk with the team, I’d also encourage you to include your name and email when submitting the feedback form. The teams at Redgate are great to work with and eager to understand user requirements and ways to improve the product.

[Download pgNow today](https://rd.gt/3FlZB8k)!
