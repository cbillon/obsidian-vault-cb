---
link: https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob
byline: Shrijith Venkatramana
site: DEV Community
date: 2025-09-21T19:45
excerpt: Hello, I'm Shrijith Venkatramana. I’m building LiveReview, a private AI code review tool that runs on...
twitter: https://twitter.com/@shrsv23
tags:
  - golang
  - postgresql
slurped: 2026-01-21T17:51
title: Turning PostgreSQL into a Robust Queue for Go Applications
---

PostgreSQL offers built-in features that make it a solid choice for handling queues in your applications. Instead of relying on external systems like RabbitMQ or Redis, you can use PostgreSQL's transactional guarantees and ACID compliance to manage tasks reliably. This approach simplifies your stack, especially if you're already using PostgreSQL for data storage.

In Go, integrating with PostgreSQL for queuing involves standard database operations with some optimizations for concurrency and efficiency. You'll handle enqueuing, dequeuing, and processing tasks while ensuring reliability. This guide walks through the process with practical examples.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#why-choose-postgresql-for-queuing-tasks)Why Choose PostgreSQL for Queuing Tasks?

Queues manage asynchronous tasks, like sending emails or processing uploads, without blocking your main application flow. PostgreSQL stands out because it supports row-level locking, notifications, and advisory locks, which help avoid common queue issues like race conditions.

Key advantages include:

- **Transactional safety**: Tasks commit or rollback atomically.
- **No extra infrastructure**: Reuse your existing database.
- **Durability**: Data persists even on crashes.

Compared to dedicated queues, PostgreSQL might not match ultra-high throughput, but it excels for moderate workloads. For instance, if your app processes hundreds of tasks per minute, this setup works well without added complexity.

|Feature|PostgreSQL Queue|Dedicated Queue (e.g., RabbitMQ)|
|---|---|---|
|Setup Complexity|Low (use existing DB)|Higher (separate service)|
|Durability|High (ACID)|Varies (configurable)|
|Cost|Included in DB|Additional|
|Scalability|Good for moderate loads|Excellent for high loads|

See the [PostgreSQL documentation on transactions](https://www.postgresql.org/docs/current/transaction-iso.html) for more on ACID properties.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#preparing-your-postgresql-environment)Preparing Your PostgreSQL Environment

Start by ensuring PostgreSQL is installed and running. Use version 14 or later for better performance features like improved indexing.

Create a database for your queue:  

```
CREATE DATABASE task_queue;
```

 

Connect to it and enable necessary extensions if needed, though for basic queues, the core features suffice.

In Go, you'll need the `lib/pq` driver. Install it with:  

```
go get github.com/lib/pq
```

 

Set up your connection string, like `postgres://user:pass@localhost:5432/task_queue?sslmode=disable`. Always handle connections with a pool for efficiency.

Test your setup with a simple query to confirm connectivity.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#designing-the-queue-table-schema)Designing the Queue Table Schema

The core of your queue is a table to store tasks. Include columns for task ID, payload, status, and timestamps.

Here's a sample schema:  

```
CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    payload JSONB NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    attempts INTEGER DEFAULT 0,
    last_attempt_at TIMESTAMP WITH TIME ZONE
);

CREATE INDEX idx_status ON tasks (status);
```

 

- **payload**: Stores task data as JSONB for flexibility.
- **status**: Tracks states like 'pending', 'processing', 'failed', 'done'.
- **attempts**: Counts retries for failure handling.

The index on status speeds up queries for pending tasks. This design supports easy querying and updates.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#enqueuing-tasks-from-go-code)Enqueuing Tasks from Go Code

To add tasks, insert rows into the table. Use Go's `database/sql` package for this.

Here's a complete example:  

```
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "log"

    _ "github.com/lib/pq"
)

func main() {
    db, err := sql.Open("postgres", "postgres://user:pass@localhost:5432/task_queue?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    payload := map[string]string{"action": "send_email", "to": "user@example.com"}
    payloadBytes, _ := json.Marshal(payload)

    _, err = db.Exec("INSERT INTO tasks (payload) VALUES ($1)", payloadBytes)
    if err != nil {
        log.Fatal(err)
    }

    fmt.Println("Task enqueued successfully")
    // Output: Task enqueued successfully
}
```

 

This code connects, marshals a JSON payload, and inserts it. Run it after setting up the table; it should add a row without errors.

Wrap enqueuing in transactions if it depends on other operations.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#dequeuing-and-processing-tasks-efficiently)Dequeuing and Processing Tasks Efficiently

Dequeuing involves selecting a pending task, marking it as processing, and handling it. Use `FOR UPDATE SKIP LOCKED` to avoid blocking multiple workers.

Example worker code:  

```
package main

import (
    "database/sql"
    "encoding/json"
    "fmt"
    "log"
    "time"

    _ "github.com/lib/pq"
)

type Task struct {
    ID      int
    Payload map[string]string
}

func main() {
    db, err := sql.Open("postgres", "postgres://user:pass@localhost:5432/task_queue?sslmode=disable")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()

    for {
        tx, err := db.Begin()
        if err != nil {
            log.Println(err)
            time.Sleep(1 * time.Second)
            continue
        }

        var id int
        var payloadBytes []byte
        err = tx.QueryRow(`SELECT id, payload FROM tasks 
            WHERE status = 'pending' 
            ORDER BY created_at ASC 
            FOR UPDATE SKIP LOCKED 
            LIMIT 1`).Scan(&id, &payloadBytes)
        if err == sql.ErrNoRows {
            tx.Rollback()
            time.Sleep(1 * time.Second)
            continue
        } else if err != nil {
            tx.Rollback()
            log.Println(err)
            continue
        }

        _, err = tx.Exec("UPDATE tasks SET status = 'processing', last_attempt_at = CURRENT_TIMESTAMP WHERE id = $1", id)
        if err != nil {
            tx.Rollback()
            log.Println(err)
            continue
        }

        tx.Commit()

        var payload map[string]string
        json.Unmarshal(payloadBytes, &payload)
        // Process the task here, e.g., send email based on payload["action"]

        _, err = db.Exec("UPDATE tasks SET status = 'done' WHERE id = $1", id)
        if err != nil {
            log.Println(err)
        }

        fmt.Printf("Processed task %d\n", id)
        // Example output: Processed task 1
    }
}
```

 

This loop polls for tasks, locks one, processes it, and updates status. The `SKIP LOCKED` allows multiple workers to run concurrently without conflicts.

For more on row locking, check [PostgreSQL's SELECT FOR UPDATE docs](https://www.postgresql.org/docs/current/sql-select.html#SQL-FOR-UPDATE-SHARE).

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#implementing-retry-logic-for-failed-tasks)Implementing Retry Logic for Failed Tasks

Failures happen, so build in retries. Update attempts on error and reset status after max attempts.

Modify the dequeue code to handle failures:  

```
// Add this inside the processing block, after unmarshal

// Simulate processing
if payload["action"] == "send_email" {
    // Imagine email sending code here
    // If fails:
    err = fmt.Errorf("simulated failure")
}

if err != nil {
    attempts := 1 // Fetch actual attempts from query
    if attempts >= 3 { // Max retries
        _, err = db.Exec("UPDATE tasks SET status = 'failed' WHERE id = $1", id)
    } else {
        _, err = db.Exec("UPDATE tasks SET status = 'pending', attempts = attempts + 1 WHERE id = $1", id)
    }
    if err != nil {
        log.Println(err)
    }
} else {
    _, err = db.Exec("UPDATE tasks SET status = 'done' WHERE id = $1", id)
    if err != nil {
        log.Println(err)
    }
}
```

 

Query attempts in the select:  

```
SELECT id, payload, attempts FROM tasks ...
```

 

This retries up to 3 times, marking as failed afterward. Adjust based on your needs.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#scaling-with-multiple-workers-and-notifications)Scaling with Multiple Workers and Notifications

For higher throughput, run multiple Go workers. Each polls independently thanks to SKIP LOCKED.

To reduce polling overhead, use PostgreSQL's LISTEN/NOTIFY. Notify on enqueue, and workers listen for events.

Setup in PostgreSQL:  

```
-- On enqueue, after INSERT:
NOTIFY new_task;
```

 

In Go, use `pq.Listener`:  

```
import "github.com/lib/pq"

listener := pq.NewListener(connString, 10*time.Second, time.Minute, func(ev pq.ListenerEventType, err error) {})
err = listener.Listen("new_task")
if err != nil {
    log.Fatal(err)
}

for {
    select {
    case n := <-listener.Notify:
        // Dequeue and process
    case <-time.After(90 * time.Second):
        // Check anyway
    }
}
```

 

This wakes workers on new tasks, saving CPU. See [pq library docs](https://pkg.go.dev/github.com/lib/pq) for full listener usage.

## [](https://dev.to/shrsv/turning-postgresql-into-a-robust-queue-for-go-applications-1hob#optimizing-performance-for-larger-workloads)Optimizing Performance for Larger Workloads

Monitor query performance with EXPLAIN ANALYZE on your SELECTs.

Add vacuuming for the table:  

```
VACUUM ANALYZE tasks;
```

 

For very large queues, partition the table by status or date.

Use connection pooling in Go with `sql.DB` defaults, and set `MaxOpenConnections` to match your DB capacity.

Benchmark your setup: Insert 1000 tasks and measure processing time. Aim for under 10ms per task in low-load scenarios.

If queues grow too large, archive done tasks to a separate table periodically.

This PostgreSQL-based queue in Go provides a reliable, low-overhead solution for many applications. It handles failures gracefully and scales with your database. Experiment with these patterns in your projects to see where they fit best, and consider monitoring tools like pg_stat_statements for ongoing tweaks.

[![](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fwqw6gfklcvrrbqhq1rua.gif)](https://hexmos.com/livereview)