---
link: https://wendelladriel.com/blog/welcome-to-the-state-machine-pattern
site: Wendell Adriel
excerpt: Learn what the State Machine Pattern is and how to apply it in your PHP
  applications with this in-depth article about it
twitter: https://twitter.com/wendell_adriel
tags:
  - slurp/backend
  - slurp/php
  - slurp/architecture
  - slurp/design-patterns
  - slurp/state-pattern
  - slurp/state-machine-pattern
slurped: 2025-09-13T19:37
title: Welcome to the State Machine Pattern
---

[

## Introduction

[site](https://wendelladriel.com/blog/welcome-to-the-state-machine-pattern#introduction)

Complex workflows often start as simple if/else statements and quickly transform into tangled logic that’s hard to read, test, and change. A **State Machine** gives that logic a clear shape: you define explicit states, the events that can happen, and the rules that allow (or forbid) moving from one state to another. The result is predictable, debuggable, and safe behavior.

[

## Why State Machines in PHP?

Because complex workflows don’t stay simple. What starts as a couple of flags and if/else checks quickly turns into contradictory states and hidden bugs. A **State Machine** turns that tangle into an explicit model: a finite set of states, the events that move you between them, and rules (guards) that make forbidden transitions impossible.

Here are some of the advantages of applying the **State Machine** pattern:

- **Clarity:** You can see every allowed transition in one place.
    
- **Safety:** Forbidden moves (e.g., refund before pay) are blocked by design.
    
- **Testability:** Transitions are pure, easy to unit test.
    
- **Observability:** Every state change can be logged and audited.
    

Let's check a really tiny example of how a **State Machine** would look like:

```
// The "tangle"
if ($order->paid && !$order->shipped && !$order->cancelled) {
    // ...
}
if ($order->shipped && $order->refunded) {
    // Impossible, but it can happen
}


// With a State Machine
enum OrderState
{
    case New;
    case Paid;
    case Shipped;
    case Cancelled;
    case Refunded;
}

$transitions = [
    OrderState::New => [
        'pay' => OrderState::Paid,
        'cancel' => OrderState::Cancelled
    ],
    OrderState::Paid => [
        'ship' => OrderState::Shipped,
        'refund' => OrderState::Refunded
    ],
    OrderState::Shipped => [
        // refund not allowed here by design
    ],
];
```

The **State Machine** pattern is a great fit when:

- You have 3+ mutually exclusive “phases/states” (paid/shipped/cancelled/refunded).
    
- Certain moves are forbidden or time-bound (e.g., cancel only before shipment).
    
- You need an audit trail or predictable side effects (emails, webhooks).
    

[

## Core Concepts


A **State Machine** is built from a few simple pieces.

A **State** is a named phase your object can be in (e.g., New, Paid, Shipped).

An **Event** is something that happens (pay, ship, cancel).

A **Transition** is the rule that says “from this state, when this event occurs, move to that state.”

A **Guard** is a boolean rule that must be `true` for the transition to happen (e.g., payment authorized).

**Actions** (or entry/exit effects) are the side effects you trigger when a transition succeeds, like sending an email or firing a webhook.

Keep the core of your **State Machine** deterministic: given the same current state and event, the next state is predictable. Put side effects at the edges so the transition decision stays pure, easy to test, and safe to run repeatedly.

[

## Design Before You Code

Resist the urge to jump straight into the code. Start with a tiny design doc and a sketch. Identify the **states as nouns** (Draft, Review, Published) and the **events as verbs** (submit, approve, publish). Write down the rules in plain language first. If two flags can’t be true at the same time, that’s a single state. If a “sometimes allowed” rule exists, that’s a **Guard** you’ll implement later.

Extract the invariants and make them explicit.

```
“Published articles must have a publish_at in the past”
“Only the author can update a Draft”
“You can’t schedule an article after it’s published”
```

These become **Guards** and **Preconditions**. Capture side effects too, but at the edges:

```
“On publish, send a notification”
“On update, clear scheduled jobs”
```

Decide if they must be idempotent (usually yes) and what to do on partial failures.

Let's check another quick example, other than related to orders. Imagine an **Article** workflow with well‑named states and events. Keep names short and unambiguous.

```
enum ArticleState
{
    case Draft;
    case InReview;
    case Scheduled;
    case Published;
    case Archived;
}

enum ArticleEvent
{
    case Submit;
    case Approve;
    case Schedule;
    case Publish;
    case Update;
    case Archive;
}

// Sample invariants/guards you’ll later implement:
final readonly class ArticleContext
{
    public function __construct(
        public int $authorId,
        public ?DateTimeImmutable $publishAt,
        public int $currentUserId,
        public bool $hasRequiredMeta,
    ) {}
}

// Plain-language rules you’ll turn into code:
// - Draft --Submit--> InReview       (guard: hasRequiredMeta)
// - InReview --Approve--> Scheduled  (guard: publishAt in future) OR --> Published (guard: publishAt null or <= now)
// - Scheduled --Publish--> Published (guard: now >= publishAt)
// - Draft/InReview --Update--> Draft (guard: currentUserId === authorId)
// - Published --Archive--> Archived  (terminal)
```

Decide how you’ll persist state and history. Most systems need the current state on the aggregate (article.state) and a transition log for audits and debugging. If concurrency is a concern, plan for optimistic locking with a version column. If timers are involved, note where you’ll schedule time‑based transitions (e.g., a queue or cron that emits Publish when `now >= publish_at`).

Finally, sanity‑check complexity. If you have more than 7–9 states or events, split the workflow into sub‑machines (e.g., `Editorial` vs `Lifecycle`) to keep each model understandable. Clear names, explicit invariants, and a small diagram will do more for reliability than any clever code you write later.

[

## Implementing A Simple State Machine

Keep your model explicit and type-safe. Use Enums for states and events so forbidden values can’t sneak in. For persistence, string-backed enums are practical: the database stores readable values while your code enjoys strict typing. Pass extra data as small, immutable value objects. A readonly context keeps your guards honest and your tests simple.

**Transitions** need a clear, fast lookup. For tiny **State Machines**, a match expression can work, but once you have more than a couple of transitions, a precomputed map gives you O(1) lookups and a single source of truth. A thin `Transition` class improves readability and tooling.

**Guards** and **Actions** are callables, where the **Guards** decide if a move is allowed and **Actions** perform side effects after a successful move. Keep the **Actions** at the edge so the decision logic stays pure and deterministic.

Let's go back to our Orders example, but now creating a full, yet simple example of a **State Machine** for it.

First we will start with our **States** and **Events**.

```
enum OrderState: string
{
    case New = 'new';
    case Paid = 'paid';
    case Shipped = 'shipped';
    case Cancelled = 'cancelled';
}

enum OrderEvent: string
{
    case Pay = 'pay';
    case Ship = 'ship';
    case Cancel = 'cancel';
}
```

Now, as said earlier, let's build a simple `Transition` class, that will handle the definition for a transition between states. For this, we will also need to create a "Context" class, that's going to be a simple DTO with the data needed for a transition.

```
final readonly class Transition
{
    /**
     * @param null|callable(OrderContext):bool $guard
     * @param null|callable(OrderContext):void $action
     */
    public function __construct(
        public OrderState $to,
        public ?callable $guard = null,
        public ?callable $action = null,
    ) {}
}

final readonly class OrderContext
{
    public function __construct(
        public int $orderId,
        public int $amountCents,
        public int $authorizedCents,
        public \DateTimeImmutable $now = new \DateTimeImmutable(),
    ) {}
}
```

Now that we have our **States**, **Events**, **Transition** and **Context** defined, let's glue all of them together with the implementation of our `Order` class.

```
final class Order
{
    /** @var array<string, array<string, Transition>> */
    private array $rules;

    public function __construct(
        public int $id,
        private OrderState $state = OrderState::New,
    ) {
        // Here we will precompute the rules for O(1) lookups
        $this->rules = $this->buildRules();
    }

    public function state(): OrderState
    {
        return $this->state;
    }

    public function apply(OrderEvent $event, OrderContext $context): void
    {
        $transition = $this->rules[$this->state->value][$event->value] ?? null;
        if (! $transition instanceof Transition) {
            throw new RuntimeException("Invalid transition: {$this->state->value} + {$event->value}");
        }

        if ($transition->guard && !($transition->guard)($context)) {
            throw new RuntimeException('Guard failed');
        }

        if ($transition->action) {
            // With this, we put the side effects on the edge 
            $transition->action($context);
        }

        $this->state = $transition->to;
    }

    /** @return array<string, array<string, Transition>> */
    private function buildRules(): array
    {
        return [
            OrderState::New->value => [
                OrderEvent::Pay->value => new Transition(
                    to: OrderState::Paid,
                    guard: fn(OrderContext $context) => $context->authorizedCents >= $context->amountCents,
                    action: fn(OrderContext $context) => $this->sendReceipt($context->orderId),
                ),
                OrderEvent::Cancel->value => new Transition(
                    to: OrderState::Cancelled,
                ),
            ],
            OrderState::Paid->value => [
                OrderEvent::Ship->value => new Transition(
                    to: OrderState::Shipped,
                    action: fn(OrderContext $context) => $this->notifyShipment($context->orderId),
                ),
            ],
            OrderState::Shipped->value => [], // Final state
            OrderState::Cancelled->value => [], // Final state
        ];
    }

    private function sendReceipt(int $orderId): void
    {
        // Send receipt
    }

    private function notifyShipment(int $orderId): void
    {
        // Notify about shipment
    }
}
```

With this implementation, you can see that besides being a simple one, it brings a lot of advantages for our codebase: Enums give you clarity, readonly value objects protect invariants, and a typed **Transition** map keeps rules centralized and fast. If the workflow continues to grow, you can split it into smaller **State Machines**, for example one focused in the **Payment** workflow and another one focused in the **Fulfilment** workflow.

[

## Guards, Actions, and Side Effects

**Guards** answer “may we transition?” and must be pure and fast, that means: no I/O, no API calls, no random checks hidden inside. Given the same state, event, and context, a guard should always return the same result.

**Actions** answer “what else happens if we do transition?” and should be idempotent, meaning that it should be safe to run twice without double-charging or double-emailing for example.

To keep your **State Machine** deterministic and easy to test, run side effects after the state change has been committed, not during the decision.

A practical way to enforce this separation is to make **Actions** to produce **Domain Events** instead of performing I/O directly. Your entity decides the next state and emits events into an outbox. A separate layer will then persist the state and dispatch the events to email/webhook/payment adapters with retries and idempotency.

Let's improve our implementation by adding the concept of using an outbox for **Domain Events**. The first things we will do is to create our events.

```
interface DomainEvent {}

final readonly class ReceiptToSend implements DomainEvent
{
    public function __construct(public int $orderId) {}
}

final readonly class ShipmentToNotify implements DomainEvent
{
    public function __construct(public int $orderId) {}
}
```

Now, let's update only the definition of our `Transition` class to reflect the usage of the **Domain Events**.

```
final readonly class Transition
{
    /**
     * @param null|callable(OrderContext):bool               $guard
     * @param null|callable(OrderContext):list<DomainEvent>  $action Produces events, no I/O here
     */
    public function __construct(
        public OrderState $to,
        public ?callable $guard = null,
        public ?callable $action = null,
    ) {}
}
```

It's time to update our `Order` class to use the **Domain Events** with the concept of using an outbox for them. Let's first add a new `$outbox` property to it:

```
final class Order
{
    /** @var list<DomainEvent> */
    private array $outbox = [];

    /** @var array<string, array<string, Transition>> */
    private array $rules;
    
    // ...
}
```

We also need to update the `buildRules()` method. Before it was setting the `action` directly as the methods for them, but now we want the `action` to be a list of **Domain Events**.

```
/** @return array<string, array<string, Transition>> */
private function buildRules(): array
{
    return [
        OrderState::New->value => [
            OrderEvent::Pay->value => new Transition(
                to: OrderState::Paid,
                guard: fn(OrderContext $context) => $context->authorizedCents >= $context->amountCents,
                action: fn(OrderContext $context) => [new ReceiptToSend($context->orderId)],
            ),
            OrderEvent::Cancel->value => new Transition(
                to: OrderState::Cancelled,
                action: fn(OrderContext $context) => [], // No side effects here
            ),
        ],
        OrderState::Paid->value => [
            OrderEvent::Ship->value => new Transition(
                to: OrderState::Shipped,
                action: fn(OrderContext $context) => [new ShipmentToNotify($context->orderId)],
            ),
        ],
        OrderState::Shipped->value => [], // Final state
        OrderState::Cancelled->value => [], // Final state
    ];
}
```

We will also need to update our `apply()` method to use the new `$outbox` property. The main difference here is that we apply the change of state in the `Order` instance and add all the needed **Domain Events** to the `$outbox` property, without executing the actions as we were doing before.

```
public function apply(OrderEvent $event, OrderContext $context): void
{
    $transition = $this->rules[$this->state->value][$event->value] ?? null;
    if (!$transition instanceof Transition) {
        throw new \RuntimeException("Invalid transition: {$this->state->value} + {$event->value}");
    }

    if ($transition->guard && !($transition->guard)($context)) {
        throw new \RuntimeException('Guard failed');
    }

    $this->state = $transition->to;
    if ($transition->action) {
        $events = ($transition->action)($context);
        foreach ($events as $event) {
            $this->outbox[] = $event;
        }
    }
}
```

Last but not least, we need a way to pull the **Domain Events** from the outbox and to clear it. For that let's implement a simple method in our `Order` class.

```
public function pullDomainEvents(): array
{
    $events = $this->outbox;
    $this->outbox = [];

    return $events;
}
```

And with that, we have our improved **State Machine** using **Domain Events**. Below, I'll show a pseudo-application flow of how it would work:

```
$order = new Order(id: 123);
$context = new OrderContext(
    orderId: 123,
    amountCents: 10_00,
    authorizedCents: 10_00
);

// Guard checks funds -> state becomes Paid
$order->apply(OrderEvent::Pay, $context);
$events = $order->pullDomainEvents(); // [ReceiptToSend(123)]

// STEP 1 -> Persist $order->state to DB
// STEP 2 -> Dispatch $events to be handled, making the handlers idempotent
```

This pattern keeps the decision logic pure and the side effects reliable. Persist the new state and enqueue the emitted events in the same transaction.

You can use a background worker to handle the events with retries. If a handler runs twice, idempotency will ensure that your customer doesn’t get charged or emailed twice.

[

## Testing State Machines

Treat your **State Machine** like a pure decision engine: given a State, Event, and Context, it either advances or refuses. That makes unit tests short, clear, and fast. Start with the happy path, then add tests for invalid transitions and failing guards. Assert the new state and the domain events were emitted. Make sure that you don’t hit the database or network in unit tests.

Here's a small example of a **Unit Test Suite** for the **State Machine** we built.

```
<?php

declare(strict_types=1);

use PHPUnit\Framework\TestCase;

final class OrderStateMachineTest extends TestCase
{
    public function test_happy_path_emits_events(): void
    {
        $order = new Order(id: 1);
        $context = new OrderContext(
            orderId: 1,
            amountCents: 1000,
            authorizedCents: 1000
        );

        $order->apply(OrderEvent::Pay, $context);
        $events = $order->pullDomainEvents();

        $this->assertCount(1, $events);
        $this->assertInstanceOf(ReceiptToSend::class, $events[0]);
        $this->assertSame(OrderState::Paid, $order->state());

        $order->apply(OrderEvent::Ship, $context);
        $events = $order->pullDomainEvents();

        $this->assertCount(1, $events);
        $this->assertInstanceOf(ShipmentToNotify::class, $events[0]);
        $this->assertSame(OrderState::Shipped, $order->state());
    }

    public function test_guard_blocks_payment_without_authorization(): void
    {
        $order = new Order(id: 2);
        $context = new OrderContext(
            orderId: 2,
            amountCents: 1000,
            authorizedCents: 500
        );

        $this->expectException(RuntimeException::class);
        $this->expectExceptionMessage('Guard failed');
        $order->apply(OrderEvent::Pay, $context);
    }

    public function test_invalid_transition_is_rejected(): void
    {
        $order = new Order(id: 3);
        $context = new OrderContext(
            orderId: 3,
            amountCents: 1000,
            authorizedCents: 1000
        );

        $this->expectException(RuntimeException::class);
        $order->apply(OrderEvent::Ship, $context);
    }
}
```

[

## State Machines and Observability

Make your state changes observable. At minimum, log transitions and measure latency so you can spot hotspots and failures. A tiny wrapper that logs success and failure does the job and keeps your entity clean.

For simplicity of this example, let's create a `OrderTransition` class that will wrap the `apply()` from our `Order` class and add logging capabilities.

```
<?php

declare(strict_types=1);

use Psr\Log\LoggerInterface;

final class OrderTransition
{
    public function __construct(private LoggerInterface $log) {}

    public function apply(Order $order, OrderEvent $event, OrderContext $context): void
    {
        $from = $order->state();
        $start = hrtime(true);

        try {
            $order->apply($event, $context);
            $to = $order->state();

            $this->log->info('order.transition.ok', [
                'order_id' => $order->id,
                'event' => $event->value,
                'from' => $from->value,
                'to' => $to->value,
                'ms' => (hrtime(true) - $start) / 1_000_000,
            ]);
        } catch (Throwable $exception) {
            $this->log->warning('order.transition.fail', [
                'order_id' => $order->id,
                'event' => $event->value,
                'from' => $from->value,
                'error' => $exception->getMessage(),
                'ms' => (hrtime(true) - $start) / 1_000_000,
            ]);

            throw $exception;
        }
    }
}
```

With this implementation, this is how our pseudo-application flow would look like.

```
$order = new Order(id: 123);
$context = new OrderContext(
    orderId: 123,
    amountCents: 10_00,
    authorizedCents: 10_00
);

$transition = new OrderTransition($logger);
// Guard checks funds -> state becomes Paid
$transition->apply($order, OrderEvent::Pay, $context);
$events = $order->pullDomainEvents(); // [ReceiptToSend(123)]

// STEP 1 -> Persist $order->state to DB
// STEP 2 -> Dispatch $events to be handled, making the handlers idempotent
```

[

## Documenting State Machines


You can keep the docs in sync with code by generating a diagram from your transition map. Expose a read-only view of the rules and render `Mermaid` syntax you can paste into a Markdown file or Pull Requests.

Let's first create a method in our `Order` class where we can get the map of our transitions.

```
/** @return array<string, array<string, Transition>> */
public function transitions(): array
{
    return $this->rules;
}
```

Now we can create a helper method to transform any map of transitions we have to a `Mermaid` syntax.

```
/**
 * @param array<string, array<string, Transition>>  $rules
 */
function toMermaid(array $rules): string
{
    $lines = ["stateDiagram-v2"];
    foreach ($rules as $from => $events) {
        foreach ($events as $event => $transition) {
            $to = $transition->to->value;
            $lines[] = "  {$from} --> {$to} : {$event}";
        }
    }
    return implode("\n", $lines)."\n";
}
```

And this is how you can use it for creating a file for your transitions.

```
$order = new Order(id: 42);
file_put_contents('order-state-machine.mmd', toMermaid($order->transitions()));
```

With this, your **State Machine** becomes a reliable and explainable part of the system. You can add a CI step that regenerates the diagram, so you'll never have an outdated one.

[

## Conclusion

**State Machines** turn messy conditionals into explicit, predictable workflows. You get logic that’s easier to read, test, and evolve. Modern PHP features like Enums and readonly value objects, and a small transition map can make the implementation straightforward and type‑safe.

Your next step is simple: pick one real workflow, sketch its states and events, write the invariants in plain language, and implement the minimal version you saw here. Add a few unit tests, log transitions for observability, and generate a `Mermaid` diagram to document the model. When side effects appear, emit **Domain Events** and deliver them via an outbox so retries are safe.