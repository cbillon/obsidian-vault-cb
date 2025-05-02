---
link: https://spin.atomicobject.com/redis-postgresql/
byline: Chris Farber
site: Atomic Spin
date: 2021-02-04T13:00
excerpt: It's a common architecture to have PostgreSQL and Redis serve as the backbone of your app, but PostgreSQL may be capable of everything you use Redis for.
twitter: https://twitter.com/@atomicobject
slurped: 2025-02-17T16:22
title: Do You Really Need Redis? How to Get Away with Just PostgreSQL
tags:
  - postgresql
---

There’s a tried-and-true architecture that I’ve seen many times for supporting your web services and applications:

- PostgreSQL for data storage
- Redis for coordinating background job queues (and some limited atomic operations)

Redis is fantastic, but what if I told you that its most common use cases for this stack could actually be achieved using only PostgreSQL?

## Use Case 1: Job Queuing

Perhaps the most common use of Redis I’ve seen is to coordinate dispatching of jobs from your web service to a pool of background workers. The concept is that you’d like to record the desire for some background job to be performed (perhaps with some input data) and to ensure that only one of your many background workers will pick it up. Redis helps with this because it provides a rich set of atomic operations for its data structures.

But since the introduction of version 9.5, PostgreSQL has a `SKIP LOCKED` option for the `SELECT ... FOR ...`statement ([here’s the documentation](https://www.postgresql.org/docs/9.5/sql-select.html#SQL-FOR-UPDATE-SHARE)). When this option is specified, PostgreSQL will just ignore any rows that would require waiting for a lock to be released.

Consider this example from the perspective of a background worker:

```

BEGIN;

WITH job AS (
  SELECT
    id
  FROM
    jobs
  WHERE
    status = 'pending'
  LIMIT 1
  FOR UPDATE SKIP LOCKED
)
UPDATE
  jobs
SET
  status = 'running'
WHERE
  jobs.id = job.id
RETURNING
  jobs.*;
  
COMMIT;
```

By specifying `FOR UPDATE SKIP LOCKED`, a row-level lock is implicitly acquired for any rows returned from the `SELECT`. Further, because you specified `SKIP LOCKED`, there’s no chance of this statement blocking on another transaction. If there’s another job ready to be processed, it will be returned. There’s no concern about multiple workers running this command receiving the same row because of the row-level lock.

The biggest caveat for this technique is that, if you have a large number of workers trying to pull off this queue and a large number of jobs feeding them, they may spend some time stepping through jobs and trying to acquire a lock. In practice, most of the apps I’ve worked on have fewer than a dozen background workers, and the cost is not likely to be significant.

## Use Case 2: Application Locks

Let’s imagine that you have a synchronization routine with a third-party service, and you only want one instance of it running for any given user across all server processes. This is another common application I’ve seen for Redis: distributed locking.

PostgreSQL can achieve this as well using its [advisory locks](https://www.postgresql.org/docs/9.2/explicit-locking.html#ADVISORY-LOCKS). Advisory locks allow you to leverage the same locking engine PostgreSQL uses internally for your own application-defined purposes.

## Use Case 3: Pub/Sub

I saved the coolest example for last: pushing events to your active clients. For example, say you need to notify a user that they have a new message available to read. Or perhaps you’d like to stream data to the client as it becomes available. Typically, web sockets are the transport layer for these events while Redis serves as the Pub/Sub engine.

However, since version 9, PostgreSQL also provides this functionality via the [`LISTEN`](https://www.postgresql.org/docs/9.0/sql-listen.html) and [`NOTIFY`](https://www.postgresql.org/docs/9.0/sql-notify.html) statements. Any PostgreSQL client can subscribe (`LISTEN`) to a particular message channel, which is just an arbitrary string. When any other client sends a message (`NOTIFY`) on that channel, all other subscribed clients will be notified. Optionally, a small message can be attached.

If you happen to be using Rails and ActionCable, using PostgreSQL is even supported out of the box.

## Taking Full Advantage of PostgreSQL

Redis fundamentally fills a different niche than PostgreSQL and excels at things PostgreSQL doesn’t aspire to. Examples include caching data with TTLs and storing and manipulating ephemeral data.

However, PostgreSQL has a lot more capabilities than you may expect when you approach it from the perspective of just another SQL database or some mysterious entity that lives behind your ORM.

There’s a good chance that the things you’re using Redis for may actually be good tasks for PostgreSQL as well. It may be a worthy tradeoff to skip Redis and save on the operational costs and development complexity of relying on multiple data services.