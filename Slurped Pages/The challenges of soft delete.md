---
link: https://atlas9.dev/blog/soft-delete.html
site: atlas9
date: 2026-01-19T16:00
excerpt: "Exploring alternatives to the archived_at column pattern: triggers, application events, and WAL-based change data capture."
slurped: 2026-01-26T19:13
title: The challenges of soft delete
tags:
  - postgresql
---

Software projects often implement "soft delete", maybe with a `deleted` boolean or an `archived_at` timestamp column. If customers accidentally delete their data, they can recover it, which makes work easier for customer support teams. Perhaps archived records are even required for compliance or audit reasons.

I've run into some trouble with soft delete designs. I'll cover those, and ponder ideas for how I'd build this in the future.

Adding an `archived_at` column seems to ooze complexity out into queries, operations, and applications. Recovering deleted records does happen, but 99% of archived records are never going to be read.

So, the database tables will have a lot of dead data. Depending on access patterns, that might even be a significant amount of data. I've seen APIs that didn't work well with Terraform, so Terraform would delete + recreate records on every run, and over time that led to millions of dead rows. Your database can probably handle the extra bytes, and storage is fairly cheap, so it's not necessarily a problem, at first.

Hopefully, the project decided on a retention period in the beginning, and set up a periodic job to clean up those rows. Unfortunately, I'd bet that a significant percentage of projects did neither – it's really easy to ignore the archived data for a long time.

At some point, someone might want to restore a database backup. Hopefully that's for fun and profit and not because you lost the production database at 11 am. If your project is popular, you might have a giant database full of dead data that takes a long time to recreate from a dump file.

`archived_at` columns also complicate queries, operations, and application code. Applications need to make sure they always avoid the archived data that's sitting right next to the live data. Indexes need to be careful to avoid archived rows. Manual queries run for debugging or analytics are longer and more complicated. There's always a risk that archived data accidentally leaks in when it's not wanted. The complexity grows when there are mapping tables involved.

Migrations have to deal with archived data too. Migrations may involve more than just schema changes – perhaps you need to fix a mistake with default values, or add a new column and backfill values. Is that going to work on records from 2 years ago? I've done migrations where these questions were not trivial to answer.

Restoring an archived record is not always as simple as just running `SET archived_at = null` – creating a record may involve making calls to external systems as well. I've seen complex restoration code that was always a buggy, partial implementation of the "create" API endpoint. In the end, we removed the specialized restoration code and required all restoration to go through the standard APIs – that simplified the server implementation, and ensured that old data that had since become invalid, could not be restored incorrectly – it needs to pass the new validation rules.

I'm not a fan of the `archived_at` column approach. It's simple at first, but in my experience, it's full of pitfalls down the line.

Let's look at some alternatives (in PostgreSQL): application events, triggers, and logical replication.

All these approaches store archived data separately from live data – that may be a separate database table, a separate database, object storage, etc.

## Application level archiving

One team I worked with took the approach of emitting an event at the application layer when a record was deleted. The event was sent to SQS, and another service would archive that object to S3 (among other things).

This had a few big benefits:

- The primary database and application code were substantially simpler.
- Deleting a resource involved cleaning up resources in various external systems. Handling this in an async background system improved performance and reliability.
- The record and all its related records can be serialized to JSON in an application-friendly layout, rather than a serialized database table layout, so it's easier to work with.

The tradeoffs:

- It's more likely to have a bug in the application code, and indeed this happened more than once, which meant archived records were lost and manual cleanup of external resources was necessary.
- It's more infrastructure to understand and operate: multiple services, a message queue, etc.
- Archived objects in S3 were not easy to query – finding records to restore required extra tooling from the customer support teams.

## Triggers

A trigger can copy a row to an archive table before it's deleted. The archive table can be a single, generic table that stores JSON blobs:

```
CREATE TABLE archive (
    id UUID PRIMARY KEY,
    table_name TEXT NOT NULL,
    record_id TEXT NOT NULL,
    data JSONB NOT NULL,
    archived_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    caused_by_table TEXT,
    caused_by_id TEXT
);

CREATE INDEX idx_archive_table_record ON archive(table_name, record_id);
CREATE INDEX idx_archive_archived_at ON archive(archived_at);
```

The trigger function converts the deleted row to JSON:

```
CREATE OR REPLACE FUNCTION archive_on_delete()
RETURNS TRIGGER AS $$
BEGIN
    INSERT INTO archive (id, table_name, record_id, data)
    VALUES (
        gen_random_uuid(),
        TG_TABLE_NAME,
        OLD.id::TEXT,
        to_jsonb(OLD)
    );
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

Attach this trigger to any table you want to archive:

```
CREATE TRIGGER archive_users
    BEFORE DELETE ON users
    FOR EACH ROW EXECUTE FUNCTION archive_on_delete();

CREATE TRIGGER archive_documents
    BEFORE DELETE ON documents
    FOR EACH ROW EXECUTE FUNCTION archive_on_delete();
```

## Handling foreign key cascades

When a parent record is deleted, PostgreSQL cascades the delete to child records. These child deletes also fire triggers, but in the context of a cascade, you often want to know _why_ a record was deleted.

One approach is to use a session variable to track the root cause:

```
CREATE OR REPLACE FUNCTION archive_on_delete()
RETURNS TRIGGER AS $$
DECLARE
    cause_table TEXT;
    cause_id TEXT;
BEGIN
    -- Check if we're in a cascade context
    cause_table := current_setting('archive.cause_table', true);
    cause_id := current_setting('archive.cause_id', true);

    -- If this is a top-level delete, set ourselves as the cause
    IF cause_table IS NULL THEN
        PERFORM set_config('archive.cause_table', TG_TABLE_NAME, true);
        PERFORM set_config('archive.cause_id', OLD.id::TEXT, true);
        cause_table := TG_TABLE_NAME;
        cause_id := OLD.id::TEXT;
    END IF;

    INSERT INTO archive (id, table_name, record_id, data, caused_by_table, caused_by_id)
    VALUES (
        gen_random_uuid(),
        TG_TABLE_NAME,
        OLD.id::TEXT,
        to_jsonb(OLD),
        cause_table,
        cause_id
    );
    RETURN OLD;
END;
$$ LANGUAGE plpgsql;
```

Now when you delete a user, you can see which archived documents were deleted because of that user:

```
SELECT * FROM archive
WHERE caused_by_table = 'users'
AND caused_by_id = '123e4567-e89b-12d3-a456-426614174000';
```

## Tradeoffs

Triggers add some overhead to deletes, and the archive table will grow. But:

- Live tables stay clean – no `archived_at` columns, no dead rows
- Cleaning up the archive table is trivial with `WHERE archived_at < NOW() - INTERVAL '90 days'`.
- Queries don't need to filter out archived records
- Indexes stay efficient
- Applications and migrations only deal with live data
- Backups of the main tables are smaller

The archive table can even live in a separate tablespace or be partitioned by time if it grows large.

## WAL-based Change Data Capture

PostgreSQL's write-ahead log (WAL) records every change to the database. Change data capture (CDC) tools can read the WAL and stream those changes to external systems. For archiving, you'd filter for DELETE events and write the deleted records to another datastore.

Debezium is the most well-known tool for this. It connects to PostgreSQL's logical replication slot, reads changes, and publishes them to Kafka. From there, a consumer writes the data wherever you want – S3, Elasticsearch, another database, etc.

```
PostgreSQL → Debezium → Kafka → Consumer → Archive Storage
```

For simpler setups, there are lighter-weight alternatives:

- **pgstream** – streams WAL changes directly to webhooks or message queues without Kafka
- **wal2json** – a PostgreSQL plugin that outputs WAL changes as JSON, which you can consume with a custom script
- **pg_recvlogical** – PostgreSQL's built-in tool for reading logical replication streams

## Operational complexity

The main downside is operational overhead. You're running additional services that need to be monitored, maintained, and made fault-tolerant. Debezium with Kafka is a significant infrastructure investment – Kafka alone requires careful tuning and monitoring.

The lighter-weight alternatives reduce this burden but shift reliability concerns to your custom code. If your consumer crashes or falls behind, you need to handle that gracefully.

## WAL retention and max_wal_size

A critical configuration is `max_wal_size` in PostgreSQL. The database retains WAL segments until all replication slots have consumed them. If your CDC consumer stops processing – due to a bug, network issue, or downstream failure – WAL segments accumulate on the primary.

If this continues unchecked, the primary database can run out of disk space and crash.

PostgreSQL 13+ has `max_slot_wal_keep_size` to limit how much WAL a slot can retain:

```
ALTER SYSTEM SET max_slot_wal_keep_size = '10GB';
```

If a slot falls too far behind, PostgreSQL invalidates it rather than filling the disk. This protects the primary but means your CDC pipeline loses data and needs to be re-synced from a snapshot.

You need monitoring and alerting on replication slot lag. If a slot starts falling behind, you want to know before it becomes a crisis.

## Tradeoffs

WAL-based CDC provides:

- Captures all changes without modifying application code or adding triggers
- Can stream to any destination (object storage, data warehouses, search indexes)
- The primary database has no additional query load – it just writes WAL as normal

But:

- Significant operational complexity, especially with Kafka-based setups
- Risk to primary database stability if consumers fall behind
- Schema changes require careful coordination between source and consumers
- More infrastructure to understand, deploy, and debug

This approach makes the most sense when you already have Kafka or similar infrastructure, or when you need to stream changes to multiple destinations beyond just archiving.

## Replica that doesn't process deletes

This is an idea I had never considered until I wrote this post – I haven't tested this, it's just an idea.

What if you kept a PostgreSQL replica (e.g. using logical replication) that just didn't process DELETE queries? Would it effectively accumulate records and updates without conflict over time?

One potential benefit of this is that the archive can be easily queried, so finding old data is simple.

Would the replica have _any_ information about deletes? Could it separate live from deleted data? Would you be able to find a record that was "deleted 2 hours ago in account 123" for a customer? Perhaps instead of ignoring DELETE queries entirely, you could have a specialized replica that transforms DELETE events into an `archived_at` column.

One potential pitfall here could be schema migrations – would the archive run into difficulty applying migrations over time?

Another downside might be cost – running a replica and keeping all that storage could have a non-trivial cost: it costs money and has operational overhead.

## Wrapping up

If I were starting a new project today and needed soft delete, I'd reach for the trigger-based approach first. It's simple to set up, keeps live tables clean, and doesn't require extra infrastructure. The archive table is easy to query when you need it, and easy to ignore when you don't.

If you have thoughts, comments, feedback, shoot me an email at `atlas9@eabuc.com`.