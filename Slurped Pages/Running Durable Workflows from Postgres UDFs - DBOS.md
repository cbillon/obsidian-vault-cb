---
link: https://www.dbos.dev/blog/running-durable-workflows-from-postgres-udfs
excerpt: Using the Postgres workflows client from DBOS to enqueue durable workflows and send messages to them directly via a SQL function.
slurped: 2026-05-29T19:32
title: Running Durable Workflows from Postgres UDFs | DBOS
tags:
  - postgresql
  - workflow
  - architecture
---

One of the reasons we chose to build durable workflows on Postgres was because of the richness of its ecosystem. Postgres is the most popular database in the world, but it isn’t just a database: it can process incoming data with triggers, schedule tasks with pg_cron, integrate other data sources via foreign data wrappers, and much more.

To make it easier to integrate workflows with the Postgres ecosystem, we just released a [PostgreSQL workflows client.](https://docs.dbos.dev/explanations/system-tables#dbosenqueue_workflow) This lets you enqueue workflows (or send messages to them) directly from Postgres by calling a SQL function. Here’s what that looks like:

![Workflows as code via Postgres UDF client example](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/69c2b3e8560aaaa016dc5507_aa2a7d6e.png)

One important use case is to enqueue a workflow from a database trigger. For example, let’s say you want to use a workflow to process each new row in a table. You can create a Postgres trigger that runs on each insert into the table and starts a workflow processing the newly inserted row. This guarantees that exactly one workflow is launched to process each new row of the table:

![Invoking a durable workflow via a Postgres trigger](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/69c2b3e8560aaaa016dc550d_abab38f9.png)

Alternatively, you can enqueue a workflow as part of a larger database transaction, for example as part of a transactional outbox pattern. This way, you guarantee that the workflow enqueue is done atomically with your other database updates:

![Invoking a durable workflow via a Postgres SQL transaction](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/69c2b3e8560aaaa016dc5504_c97224f5.png)

### How to Enqueue Durable Workflows via Postgres UDFs

Enqueueing workflows from Postgres is powered by a user-defined function that’s created as part of your application’s system database. Here’s what it looks like, in all its gory detail:

![DBOS durable workflow defined within a Postgres UDF](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/69c2b3e8560aaaa016dc550a_6d8f20a8.png)

This looks complex, so let’s break it down. Essentially, this defines an enqueue_workflow function you can call from a SQL statement. The function takes in two required arguments (workflow name and queue name) and a number of optional arguments, such as workflow inputs or workflow ID. When it’s called, the function:

1. Validates and normalizes its arguments
2. Coerces the workflow inputs into a JSON format that the workflow execution library can parse
3. Enqueues the workflow by inserting a new row (containing all the input arguments) into the workflow_status table 

This works because [workflows are just database rows](https://www.dbos.dev/blog/why-workflows-should-be-postgres-rows). Once a new row defining a new workflow is inserted, the workflow is ready to go: a server can dequeue it, parse its inputs from JSON, and execute it.

A similar function allows sending messages to workflows from Postgres:

![Sending a message to a durable workflow via a PostgreSQL UDF](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/69c2b3e8560aaaa016dc5510_2b2685b3.png)

This function takes in a workflow ID and a message (and optionally a topic and idempotency key) and inserts the message as a new row into the notifications table for that workflow and topic. The workflow can later receive and process the message by consuming it from the notifications table.

### Learn More

To learn more about how to manage workflows from Postgres, check out the [documentation](https://docs.dbos.dev/explanations/portable-workflows).

If you like making systems reliable, we’d love to hear from you. At DBOS, our goal is to make durable workflows as easy to work with as possible. Check it out:

- Quickstart: [https://docs.dbos.dev/quickstart](https://docs.dbos.dev/quickstart) 
- GitHub: [https://github.com/dbos-inc](https://github.com/dbos-inc)
- Discord community: [https://discord.gg/eMUHrvbu67](https://discord.gg/eMUHrvbu67)