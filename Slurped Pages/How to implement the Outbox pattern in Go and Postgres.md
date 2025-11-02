---
link: https://packagemain.tech/p/how-to-implement-the-outbox-pattern-in-golang
byline: Alex Pliutau
site: packagemain.tech
date: 2025-09-16T09:57
excerpt: How and why to use the Outbox pattern to build a robust event-driven system.
slurped: 2025-11-02T18:13
title: How to implement the Outbox pattern in Go and Postgres
tags:
  - go
  - postgrest
  - patterns
---

I was at a [ContainerDays](https://www.containerdays.io/) conference recently and attended a great talk from [Nikolay Kuznetsov](https://www.linkedin.com/in/nkuznetsov/) about the Outbox pattern and resilient system design. It seemed like a powerful solution to a common problem in distributed systems, so I decided to dive deeper, and I want to share what I've learned and summarize for our readers in this post.

In modern, event-driven architectures, services often communicate asynchronously using a message broker. A typical flow looks like this: a service receives a request, updates its own database, and then publishes an event to notify other services about the change. Or these two actions happen in parallel.

Here's the problem: what happens if the database commit succeeds, but the subsequent call to the message broker fails? Maybe the broker is temporarily down, or there's a network glitch. Or what if the database is not available at that time? Or what if the program somehow crashes?

[

![](https://substackcdn.com/image/fetch/$s_!0TkS!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2d273d30-b9a1-42d7-b9b8-ef058160a0ca_2558x1062.png)

](https://substackcdn.com/image/fetch/$s_!0TkS!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2F2d273d30-b9a1-42d7-b9b8-ef058160a0ca_2558x1062.png)

You end up in an inconsistent state. Your local database has the new data, but the rest of the system never gets the notification. This is a serious issue because the database operation and the message publishing are not **atomic** - they don't succeed or fail as a single, indivisible unit.

This is where the Outbox pattern comes to the rescue! The core idea is simple but powerful: instead of directly publishing a message to the broker, you save the message to a dedicated "outbox" table within your local database. Crucially, you do this as part of the same database transaction as your business data changes.

[

![](https://substackcdn.com/image/fetch/$s_!gtns!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe3780b0c-aaeb-4919-b88c-0c241610e82e_2478x1216.png)

](https://substackcdn.com/image/fetch/$s_!gtns!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fe3780b0c-aaeb-4919-b88c-0c241610e82e_2478x1216.png)

This approach leverages the atomicity of database transactions to guarantee that either both the business data is saved and the event is queued for sending, or neither is. The message can't get lost.

To implement this, you need an outbox table in your database (or multiple ones). A typical schema might look something like this, storing the message content and its processing state.

```
-- Example schema for a PostgreSQL outbox table
CREATE TABLE outbox (
    id uuid PRIMARY KEY,
    topic varchar(255) NOT NULL,
    message jsonb NOT NULL,
    state varchar(50) NOT NULL DEFAULT 'pending', -- e.g., pending, processed
    created_at timestamptz NOT NULL DEFAULT now(),
    processed_at timestamptz
);
```

Of course, just putting messages in a database table doesn't send them. You need a separate background process, often called a Message Dispatcher or Relay. This worker's job is to:

- Periodically query the outbox table for new, unprocessed messages.
    
- Publish these messages to the actual message broker.
    
- Once the broker confirms receipt, update the message's record in the outbox table to mark it as processed.
    
- Handle errors and retries.
    

This process guarantees **at-least-once delivery**. A message might be sent more than once if the relay publishes it but fails before it can mark the record as processed. Because of this, your message consumers should always be designed to be **idempotent**, meaning they can safely process the same message multiple times without causing issues.

Let's look at a concrete example using Go, [pgx](https://github.com/jackc/pgx) for Postgres, and Google Cloud Pub/Sub. Imagine we have an **orders** service.

First, our business logic for creating an order will also create the outbox message within a single transaction.

```
// orders/main.go

package main

import (
	"context"
	"encoding/json"
	"log"
	"os"

	"github.com/google/uuid"
	"github.com/jackc/pgx/v5"
	"github.com/jackc/pgx/v5/pgxpool"
)

type Order struct {
	ID       uuid.UUID `json:"id"`
	Product  string    `json:"product"`
	Quantity int       `json:"quantity"`
}

type OrderCreatedEvent struct {
	OrderID uuid.UUID `json:"order_id"`
	Product string    `json:"product"`
}

// createOrderInTx creates an order and its corresponding outbox event atomically.
func createOrderInTx(ctx context.Context, tx pgx.Tx, order Order) error {
	// 1. Insert the order
	_, err := tx.Exec(ctx, "INSERT INTO orders (id, product, quantity) VALUES ($1, $2, $3)",
		order.ID, order.Product, order.Quantity)
	if err != nil {
		return err
	}
	log.Printf("Inserted order %s into database", order.ID)

	// 2. Prepare the event message for the outbox
	event := OrderCreatedEvent{
		OrderID: order.ID,
		Product: order.Product,
	}
	msg, err := json.Marshal(event)
	if err != nil {
		return err
	}

	// 3. Insert the event into the outbox table
	_, err = tx.Exec(ctx, "INSERT INTO outbox (topic, message) VALUES ($1, $2, $3)",
		"orders.created", msg)
	if err != nil {
		return err
	}
	log.Printf("Inserted outbox event for order %s", order.ID)

	return nil
}

func main() {
	ctx := context.Background()
	pool, err := pgxpool.New(ctx, os.Getenv("DATABASE_URL"))
	if err != nil {
		log.Fatalf("Unable to connect to database: %v", err)
	}
	defer pool.Close()

	tx, err := pool.Begin(ctx)
	if err != nil {
		log.Fatalf("Unable to begin transaction: %v", err)
	}
	defer tx.Rollback(ctx)

	newOrder := Order{
		ID:       uuid.New(),
		Product:  "Super Widget",
		Quantity: 10,
	}

	if err := createOrderInTx(ctx, tx, newOrder); err != nil {
		log.Fatalf("Failed to create order: %v", err)
	}

	if err := tx.Commit(ctx); err != nil {
		log.Fatalf("Failed to commit transaction: %v", err)
	}

	log.Println("Successfully created order and outbox event.")
}
```

Next, the Relay process polls the database, sends the message, and updates the state. Using `FOR UPDATE SKIP LOCKED` is a great way to allow multiple relay instances to run concurrently without processing the same message.

```
// relay/main.go
package main

import (
	"context"
	"log"
	"time"

	"cloud.google.com/go/pubsub"
	"github.com/google/uuid"
	"github.com/jackc/pgx/v5/pgxpool"
)

type OutboxMessage struct {
	ID      uuid.UUID
	Topic   string
	Message []byte
}

func processOutboxMessages(ctx context.Context, pool *pgxpool.Pool, pubsubClient *pubsub.Client) error {
	tx, err := pool.Begin(ctx)
	if err != nil {
		return err
	}
	defer tx.Rollback(ctx)

	// 1. Lock the next pending message so other relay instances don't grab it
	rows, err := tx.Query(ctx, `
        SELECT id, topic, message
        FROM outbox
        WHERE state = 'pending'
        ORDER BY created_at
        LIMIT 1
        FOR UPDATE SKIP LOCKED
    `)
	if err != nil {
		return err
	}
	defer rows.Close()

	// 2. If we found a message, publish it to Pub/Sub
	var msg OutboxMessage
	if rows.Next() {
		if err := rows.Scan(&msg.ID, &msg.Topic, &msg.Message); err != nil {
			return err
		}
	} else {
		// No new messages
		return nil
	}

	log.Printf("Publishing message %s to topic %s", msg.ID, msg.Topic)
	result := pubsubClient.Topic(msg.Topic).Publish(ctx, &pubsub.Message{
		Data: msg.Message,
	})
	_, err = result.Get(ctx)
	if err != nil {
		return err
	}

	// 3. Mark the message as processed
	_, err = tx.Exec(ctx, "UPDATE outbox SET state = 'processed', processed_at = now() WHERE id = $1", msg.ID)
	if err != nil {
		return err
	}
	log.Printf("Marked message %s as processed", msg.ID)

	return tx.Commit(ctx)
}

func main() {
	// TODO: initialize actual Postgres and Pubsub connections
	var (
		pool         *pgxpool.Pool
		pubsubClient *pubsub.Client
	)

	// feel free to use another interval
	ticker := time.NewTicker(1 * time.Second)
	defer ticker.Stop()
	for range ticker.C {
		if err := processOutboxMessages(context.Background(), pool, pubsubClient); err != nil {
			log.Printf("Error processing outbox: %v", err)
		}
	}
}
```

Why then we say that the message is sent at least once? That's because of the rare case when the message is sent, but something happened when updating the message status in the database, for example due to a service crash.

We touched this topic briefly in the [following post](https://packagemain.tech/p/real-time-database-change-tracking).

While polling is a simple and effective strategy, it can introduce some latency and be resource-intensive. For those using PostgreSQL, there's a more advanced, push-based alternative: logical replication.

Databases like Postgres maintain a **Write-Ahead Log (WAL)**, which is an append-only log of every change made to the database. Logical replication allows you to tap into this log and stream changes for specific tables as they happen.

Instead of your relay constantly asking the database "Anything new?", the database effectively tells your relay, "Hey, a new row was just inserted into the outbox table!". This can be more efficient and provide lower latency, though it adds some implementation complexity.

[

![](https://substackcdn.com/image/fetch/$s_!eXIM!,w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa7cd41d1-7535-497a-91e9-cfeb771eb79f_2362x1224.png)

](https://substackcdn.com/image/fetch/$s_!eXIM!,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fa7cd41d1-7535-497a-91e9-cfeb771eb79f_2362x1224.png)

In Go you can use [pglogrepl](https://github.com/jackc/pglogrepl) which is a Go library designed for interacting with PostgreSQL’s logical replication protocol.

To summarize, the Outbox Pattern ensures that a message was sent (e.g. to a queue) successfully at least once. With this pattern, instead of directly publishing a message to the queue, we store it in temporary outbox table first. We're wrapping the entity save and message storing in a single transaction so this operation is atomic. It will be published later through a background process called Message Relay.

While the concept is simple, it can be complicated to implement the actual solution and there multiple ways of doing so.

- [Source code](https://github.com/plutov/packagemain/tree/master/outbox)
    
- [Write-Ahead Logging (WAL)](https://www.postgresql.org/docs/current/wal-intro.html)
    
- [pglogrepl](https://github.com/jackc/pglogrepl)