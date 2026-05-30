---
link: https://www.dbos.dev/blog/postgres-is-all-you-need-for-durable-execution
excerpt: Explaining the concept of a Postgres-backed durable execution system like DBOS and comparing it to external workflow orchestration systems like Temporal.
slurped: 2026-05-29T19:06
title: Postgres-backed Durable Workflow Execution | DBOS
tags:
  - postgresql
  - workflow
---

Durable workflows are a simple but powerful tool for building reliable programs. The idea is that as your program runs, you regularly checkpoint its progress to a database. That way, if your program ever crashes or fails, you can reload from the last checkpoint to recover it from its last completed step. You can think of this like saving in a video game: you regularly “save” your program’s progress so that if it crashes, you can “reload” it from its last checkpoint.

Most commonly, durable workflows are implemented via **external orchestration**. This is the pattern used by systems like Temporal, Airflow, and AWS Step Functions. In this model, durable programs are written as workflows of steps whose execution is coordinated by a central orchestrator.

When a client submits a workflow, the orchestrator creates a record for it in a data store then dispatches it to a worker for execution. Each time a worker completes a step, it sends the step’s outcome back to the orchestrator. The orchestrator checkpoints the output in its data store, then dispatches the next step. If a worker crashes or fails, the orchestrator dispatches its workflows to another worker, starting them from their last checkpointed step.

![External workflow orchestration system architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/6a0dbd9279898da496135035_f62a8f73.png)

In this blog post, we’ll argue that external orchestration is **fundamentally overcomplicated**. The core idea of durable workflows is to checkpoint program state in a database. But if durable workflows are about databases, then there’s no reason to have a separate orchestrator server. Instead, it’s simpler and more efficient to use the database itself as an orchestrator. To make this more concrete, we’ll focus specifically on building durable workflows on Postgres, because its popularity, [scalability](https://www.dbos.dev/blog/benchmarking-workflow-execution-scalability-on-postgres), and rich ecosystem make it an ideal choice.

In a Postgres-backed durable workflows system, application servers directly communicate with Postgres to execute workflows instead of going through a central orchestrator. A client submits a workflow for execution by creating an entry for it in a Postgres workflows table. Application servers poll the table for workflows to dequeue and execute. As a server executes a workflow, it checkpoints the output of each step to Postgres. If a server executing workflows crashes or fails, another server can recover its workflows from their checkpoints.

![Postgres-backed durable execution system architecture diagram](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/6a0dbd9279898da496135032_7bffa8dc.png)

This design renders a central orchestrator unnecessary because application servers can coordinate through Postgres. Instead of relying on a central orchestrator to dispatch workflows to workers, servers cooperatively dequeue workflows from a Postgres table, using mechanisms such as locking clauses to ensure each workflow is dequeued by exactly one worker. Instead of relying on an orchestrator to checkpoint step outputs, workers checkpoint steps to Postgres themselves. If multiple workers try to execute the same workflow simultaneously, Postgres database integrity constraints let them detect the duplicate work on checkpoint and back off.

Replacing a central orchestrator with Postgres (or another database) makes durable workflows fundamentally simpler. In particular, it means hard problems such as scalability, availability, observability, and security can be addressed using well-understood Postgres-native solutions.

### Scalability and Availability

The scalability and availability of a database-backed durable workflows system are fundamentally determined by the underlying database. The system can scale horizontally by adding more worker servers, so its maximum capacity is determined by how quickly the database can process workflows. Similarly, workers are fungible and can freely recover each other’s state, so the system is available as long as the underlying database is available.

When using Postgres specifically, this is beneficial because Postgres scalability and availability are well-studied problems with robust solutions. For scalability, a single Postgres server can vertically scale to handle [tens of thousands of workflows per second](https://www.dbos.dev/blog/benchmarking-workflow-execution-scalability-on-postgres), and further scaling can be achieved by using distributed (e.g., [CockroachDB](https://www.cockroachlabs.com/)) or sharded Postgres. For availability, Postgres supports streaming replication with automatic failover and managed offerings provide multi-AZ deployments with high-availability SLAs out of the box. As a result, the decades of engineering work and research that have gone into operating Postgres at scale can translate directly to operating durable workflows.

### Observability

When using Postgres-backed durable execution, workflows and their steps are checkpointed to Postgres tables. This means observability is built-in: you can scan those checkpoints to monitor workflows in real time and visualize workflow execution. 

Postgres excels at this because virtually any workflow observability query can be expressed in SQL. For example, here’s a query to find all workflows that errored in the last month:

![SQL Query to analyze durable workflow execution observability data](https://cdn.prod.website-files.com/672411cbf038560468c9e68f/6a0dbd9279898da49613502f_e0ebb3ff.png)

A query like this might seem obvious, but it’s hard to overstate how powerful this is. It’s only possible because Postgres’s relational model lets you express complex filtering and analytical operations declaratively in SQL, leveraging decades of query optimization research. Many systems with simpler data models, such as the key-value stores used by popular external orchestrators, have no such support. By storing workflow and step data in Postgres tables and augmenting them with secondary indexes for fast analytical queries, you get efficient observability from your durable execution “for free.”

### Reliability and Security

When using an external orchestrator for durable execution, both the orchestrator and its data store are single points of failure. Because they directly coordinate workflow execution, if either has downtime, the entire application becomes unavailable. Moreover, because they process and store workflow and step checkpoints, they likely have access to sensitive application data, meaning they must be hardened, access-controlled, and audited like any other piece of sensitive infrastructure. 

By contrast, the only point of failure in Postgres-backed durable execution is Postgres itself, and all workflow data is stored directly in Postgres and never transits any other system. If an application already depends on Postgres, adopting durable execution does not add any new points of failure to the system nor introduce new surface area to secure. Databases are already critical infrastructure, so it makes more sense to reuse them for orchestration than to add new critical infrastructure for it.

### Learn More

If you like building scalable, reliable systems, we’d love to hear from you. At DBOS, our goal is to make Postgres-backed durable execution as simple and performant as possible. Check it out:

- Quickstart: [https://docs.dbos.dev/quickstart](https://docs.dbos.dev/quickstart) 
- GitHub: [https://github.com/dbos-inc](https://github.com/dbos-inc)
- Discord community: [https://discord.gg/eMUHrvbu67](https://discord.gg/eMUHrvbu67)