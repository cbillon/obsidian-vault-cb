---
link: https://blog.dochia.dev/blog/idempotency/
site: Dochia CLI Blog
excerpt: "Idempotency is not just an HTTP header or a key lookup. This article covers the failure cases that bite real APIs: different requests with the same key, concurrent retries, partial success, downstream uncertainty, response replay, expiry, and duplicate message handling."
twitter: https://twitter.com/@dochiadev
slurped: 2026-05-10T12:56
title: Idempotency Is Easy Until the Second Request Is Different | Dochia CLI Blog
tags:
  - idempotency
---

---

People talk about idempotency like it is a solved problem:

> Put an `Idempotency-Key` on the request. Store the response. Replay it on retry.

And yes, that is doable. For the happy path, it is even fairly small.

The client sends:

```
POST /payments
Idempotency-Key: abc-123
Content-Type: application/json
```

```
{
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR",
  "merchantReference": "invoice-7781"
}
```

The server checks whether it has seen `abc-123`. If not, it creates the payment. If yes, it returns the previous response.

That version survives the demo.

The part I contest is that this is the hard part. It is not. The hard part starts with the second request, because the second request is not always a clean replay of the first one.

Maybe it is a completed replay. Fine. Return the stored result.

Maybe it arrives while the first request is still running. Now your idempotency layer is part of your concurrency control.

Maybe the first request created a local payment but crashed before publishing an event. Now the local row and the external side effects are out of step.

Maybe the first request called a payment provider, the provider accepted it, and your process died before recording the result. Now your database cannot infer whether money moved.

Or maybe the second request has the same key and different content:

```
{
  "accountId": "acc_1",
  "amount": "100.00",
  "currency": "EUR",
  "merchantReference": "invoice-7781"
}
```

Same key. Different amount.

This is the case that makes idempotency interesting. Is it a retry? Is it a client bug? Is it a new operation? Should the server replay the old response, reject the request, or treat `(key + content)` as a new identity?

You can pick any of those policies if you document it clearly. But the server should have an opinion. Not necessarily my opinion, but a clear one.

My bias for side-effecting APIs is: same scoped key plus different canonical command should be a hard error. It catches client bugs early. A client that believes it is safely retrying a 10 EUR payment should not have the server silently interpret the second request as something else.

The cases that matter are the ones a replay cache does not explain:

- completed replay
- concurrent retry
- partial local success
- downstream unknown state
- same key with a different canonical command
- duplicate operation without a key
- retry after expiry
- retry after deploy, schema change, service hop, or region failover

If your design only handles completed same-command retries, it is a replay cache. That might be enough for some endpoints. But it is not the whole problem.

## Idempotency is about the effect

An operation is idempotent if applying it once or many times has the same intended effect.

That definition is simple. The word doing all the work is “effect”.

HTTP gives you method-level semantics. A `PUT /users/123/email` can be idempotent if sending the same representation repeatedly leaves the resource in the same state. A `DELETE /sessions/456` can be idempotent if deleting an already-deleted session still means “session does not exist”. Repeating the `DELETE` might return `404`; the effect can still be idempotent.

But your handler can still produce repeated side effects the business cares about: duplicate audit records, duplicate domain events, duplicate emails, duplicate provider calls, or duplicate metrics that affect billing or fraud logic.

`POST` is usually not idempotent by default, but it can be made idempotent if the server stores and enforces the right behavior. The key identifies a claimed operation. It does not define request equivalence, replay policy, or downstream deduplication.

A uniqueness constraint can prevent one class of duplicate. It does not, by itself, give the client a correct retry result.

For example, `unique(account_id, merchant_reference)` might prevent two payment rows, but if the retry gets a generic `500`, the client still does not know whether the payment succeeded. If the row exists but the response is different, or the event is published twice, or the ledger entry is duplicated, the operation is not idempotent in the way the caller cares about.

## What you need to remember

For `POST /payments`, the durable idempotency record needs to answer three questions:

1. Who owns this key?
2. What did the first command mean?
3. What outcome can be replayed?

In PostgreSQL-ish SQL, a minimal table might look like this:

```
create table idempotency_requests
(
    tenant_id       text        not null,
    operation_name  text        not null,
    idempotency_key text        not null,
    request_hash    text        not null,
    status          text        not null,
    response_status int,
    response_body   jsonb,
    resource_type   text,
    resource_id     text,
    error_code      text,
    created_at      timestamptz not null,
    updated_at      timestamptz not null,
    expires_at      timestamptz not null,
    locked_until    timestamptz,
    primary key (tenant_id, operation_name, idempotency_key)
);
```

The key is not globally unique unless you deliberately make it global. Usually it should not be. A broken client generating `abc-123` should only collide with itself, not with another tenant.

Scope might be tenant, user, account, merchant, API client, or some combination. Pick it deliberately.

The operation name prevents accidental reuse across different operations. A key used for `create_payment` should not automatically mean the same thing for `create_refund`.

The `request_hash` is the server’s memory of the first command. Without it, same key plus different body becomes ambiguous. You either replay the first response for a different command, or you execute a new operation under an old key. Both are bad if the client thinks it is retrying.

`IN_PROGRESS` is not an internal detail. A retry can arrive while the first request still owns execution.

The behavior needs to be explicit:

|Existing record|Same canonical command?|Suggested behavior|
|---|---|---|
|none|yes|insert `IN_PROGRESS` and execute|
|`COMPLETED`|yes|replay stored response or documented equivalent|
|any existing record|no|reject with idempotency conflict|
|`IN_PROGRESS`, fresh|yes|wait, return `202`, or return `409` + `Retry-After`|
|`IN_PROGRESS`, stale|yes|recover ownership; do not blindly execute again|
|`FAILED_REPLAYABLE`|yes|replay stored failure|
|`FAILED_RETRYABLE`|yes|allow retry according to policy|
|`UNKNOWN_REQUIRES_RECOVERY`|yes|trigger reconciliation or return pending/recovery status|
|expired/deleted|unknown|follow documented expiry behavior|

The response fields exist because idempotency is not just about preventing duplicate writes. The client needs an answer.

You can store the full response body, or store a reference to the created resource and reconstruct the response. Both choices are annoying in different ways.

Storing full responses gives faithful replay. It can also retain PII, signed URLs, one-time tokens, cardholder-related data, or fields you never intended to keep in a retry table.

Reconstructing from a resource reference saves space, but it can return a different representation if the resource changed after creation.

This is a contract decision. “Replay the creation response” and “return the current payment” are both valid API designs. They are not the same design.

## Same key, different command

This is the bug the idempotency layer should catch loudly.

First request:

```
{
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR",
  "merchantReference": "invoice-7781"
}
```

Second request:

```
{
  "accountId": "acc_1",
  "amount": "100.00",
  "currency": "EUR",
  "merchantReference": "invoice-7781"
}
```

Same `Idempotency-Key: abc-123`. Different amount.

Returning the original response anyway is simple. It also hides a serious client bug. The client asked for a 100 EUR payment and got back a 10 EUR payment. If the caller does not compare the response carefully, it may believe the 100 EUR payment succeeded.

That is not idempotency. That is reinterpretation.

For side-effecting APIs, a scoped key reused with a different canonical command should be a hard error, regardless of whether the first operation completed, failed, or is still running.

```
HTTP/1.1 409 Conflict
Content-Type: application/json
```

```
{
  "errorCode": "IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_REQUEST",
  "message": "This idempotency key was already used with a different request."
}
```

`409 Conflict` is a defensible default because the request conflicts with the server’s remembered meaning for that scoped key. Some APIs use `400` or `422`; the important part is a stable machine-readable error and no silent replay for a different command.

A common client bug looks like this:

```
bad:
  idempotencyKey = cartId

POST /payments amount=10.00 key=cart_123
POST /payments amount=15.00 key=cart_123

better:
  idempotencyKey = paymentAttemptId
```

The server should not guess which payment the cart key was supposed to represent.

You can design an API where `(key + content hash)` defines the operation identity. That is a valid policy. But then the key is no longer an idempotency key in the usual retry sense. It is part of a composite operation identifier. That needs to be obvious to the client.

The dangerous version is the middle ground, where the client thinks it is safely retrying one operation and the server silently interprets the second request as another.

## Hash the command, not the bytes

Raw byte comparison is usually too strict for JSON APIs. These two bodies should normally be equivalent:

```
{
  "amount": "10.00",
  "currency": "EUR"
}
```

```
{
  "currency": "EUR",
  "amount": "10.00"
}
```

Field order and whitespace should not matter.

Defaults are less obvious:

```
{
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR"
}
```

versus:

```
{
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR",
  "channel": "web"
}
```

If `channel: "web"` is the server default, are these the same logical command? Maybe. Decide before hashing.

Unknown fields are another trap. Suppose your API ignores unknown JSON fields. If the first request includes `"foo": "bar"` and the second does not, do you consider them the same? If unknown fields are truly ignored, perhaps yes. If they might become meaningful after a deploy, perhaps no.

The practical rule is: hash the validated command, not the raw HTTP body.

A reasonable flow is:

1. Parse the request into a versioned request DTO or command.
2. Normalize values your API treats as equivalent: amounts, enum casing, default fields, timestamp precision.
3. Exclude transport-only metadata.
4. Include path parameters and operation name.
5. Include semantic headers if they affect the operation, such as API version.
6. If a header only affects response shape, such as `Prefer: return=minimal`, decide whether it belongs in the command hash, the replay contract, or neither.
7. Exclude `Authorization` and the idempotency key itself.
8. Serialize canonically.
9. Hash with a stable algorithm.

For the payment example, the fingerprint might include:

```
operation: create_payment
accountId: acc_1
amount: 10.00
currency: EUR
merchantReference: invoice-7781
channel: web
apiVersion: 2026-05-01
```

Be careful with amounts, timestamps, generated defaults, locale-sensitive formatting, and fields added during deploys. The request hash is a contract. If you change how it is computed, old retries can start looking different.

## The first insert decides who owns execution

Two identical requests hit two API instances at nearly the same time:

```
POST /payments
Idempotency-Key: abc-123
```

Same canonical command. Same tenant. Same endpoint.

This implementation is broken even if every single-threaded test passes:

```
existing = find_by_key(key)
if existing does not exist:
    create_payment()
    insert_idempotency_record()
```

Both requests can observe no existing row. Both can execute the side effect.

If there is no atomic insert or unique constraint on the scoped key, two instances can both decide they own execution.

The insert-first shape is:

```
insert into idempotency_requests (tenant_id,
                                  operation_name,
                                  idempotency_key,
                                  request_hash,
                                  status,
                                  created_at,
                                  updated_at,
                                  expires_at,
                                  locked_until)
values (:tenant_id,
        'create_payment',
        :idempotency_key,
        :request_hash,
        'IN_PROGRESS',
        now(),
        now(),
        now() + interval '24 hours',
        now() + interval '30 seconds') on conflict do nothing;
```

The exact syntax is database-specific. The important property is atomic ownership acquisition for `(tenant_id, operation_name, idempotency_key)`.

Then:

```
if rows_inserted == 1:
    this request owns execution
else:
    existing = load idempotency row

    if existing.request_hash != request_hash:
        return 409 IDEMPOTENCY_KEY_REUSED_WITH_DIFFERENT_REQUEST

    if existing.status == COMPLETED:
        return replay(existing.response_status, existing.response_body)

    if existing.status == IN_PROGRESS and existing.locked_until > now():
        return 202 or 409 + Retry-After

    if existing.status == IN_PROGRESS and existing.locked_until <= now():
        attempt recovery ownership
        # this must be atomic too

    if existing.status == UNKNOWN_REQUIRES_RECOVERY:
        trigger reconciliation or return pending/recovery response
```

Recovery ownership has to be acquired atomically too. Otherwise two retries can both decide the old owner is dead and both start recovery.

In the simple local case, the owner can create the payment and complete the idempotency record in one transaction:

```
begin transaction

insert idempotency row as IN_PROGRESS
insert payment row pay_789
insert outbox event PaymentCreated(pay_789)
update idempotency row:
  status = COMPLETED
  resource_type = payment
  resource_id = pay_789
  response_status = 201
  response_body = {...}

commit
```

That is the nice version: one database transaction covers the idempotency row, the business row, and the outbox event.

External side effects change the shape. Holding a database transaction open while calling a provider is usually a bad idea. Committing before the provider call means your local state may say `IN_PROGRESS` while execution continues outside the transaction. If the process crashes there, a retry has to recover. This is where you need an operation state machine and a recovery worker, not just a request table.

Redis `SET NX EX` is often proposed as the whole solution. At best, it is an execution guard:

```
SET idempotency:tenant_1:create_payment:abc-123 value NX EX 30
```

It can reduce duplicate concurrent execution. It is not durable memory of the operation outcome. If the Redis lock expires while the provider call is still running, another request can enter. If the process dies after the provider succeeds but before storing the response, the lock does not help the retry know what happened. Redis locks also need fencing or durable ownership if they protect downstream resources.

Redis can be useful. It is not a substitute for remembering the operation outcome.

## The provider timeout is where your guarantee ends

The failure path that matters is not exotic:

1. API receives `POST /payments`.
2. It inserts an idempotency row as `IN_PROGRESS`.
3. It creates local payment `pay_789`.
4. It calls a downstream payment provider.
5. The provider receives the request and succeeds.
6. The API times out, crashes, or loses the provider response.
7. The client retries with the same key.

If the provider received your request and your process died before recording the result, your database cannot infer whether money moved.

A local state machine might look like this:

```
RECEIVED
LOCAL_PAYMENT_CREATED
PROVIDER_REQUEST_SENT
PROVIDER_CONFIRMED
COMPLETED
UNKNOWN_REQUIRES_RECOVERY
```

The retry behavior depends on the state.

If the retry finds `COMPLETED`, replay.

If it finds a fresh `PROVIDER_REQUEST_SENT`, return `202 Accepted`, `409 Conflict` with `Retry-After`, or block briefly and wait for completion. Pick one behavior and document it. Clients need to know whether to retry, poll, or wait.

If it finds stale `PROVIDER_REQUEST_SENT`, do not create `pay_790`. Do not call the provider with a new identity. Recover using the stable downstream operation ID:

```
payment id: pay_789
provider idempotency key: provider_payment_pay_789
```

A recovery worker or retrying request can then:

1. acquire recovery ownership for `pay_789`
2. query the provider by `provider_payment_pay_789`, if the provider supports it
3. if confirmed, mark the provider operation confirmed
4. mark the idempotency record `COMPLETED`
5. store or reconstruct the response
6. replay the response or return a documented final status
7. if the provider cannot answer, mark `UNKNOWN_REQUIRES_RECOVERY`

If the provider has no idempotency key and no query API, your system has an operational gap. You may still choose to accept it, but the local idempotency table is not protecting the external effect. It only prevents duplicate local request handling.

For payment-like operations, the client’s idempotency key is often not the exact key sent downstream. The downstream call needs a stable identity that survives retries, crashes, and reconciliation. Otherwise the second local attempt is just a second provider attempt.

I would avoid `425 Too Early` unless your API already has a specific reason to use it. Most clients will not handle it specially. `202 Accepted`, `409 Conflict` with `Retry-After`, or an operation-status endpoint are easier to explain.

## Replay is a contract, not a convenience

For a completed idempotent request, replaying the same status and body is the least surprising behavior:

```
HTTP/1.1 201 Created
Idempotent-Replayed: true
Content-Type: application/json
```

```
{
  "paymentId": "pay_789",
  "status": "PENDING",
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR",
  "merchantReference": "invoice-7781"
}
```

A custom response header such as `Idempotent-Replayed: true` can help debugging. I would not make clients depend on it.

Reconstructing responses from current resource state is tempting:

```
load payment pay_789
return current representation
```

But suppose the first response was:

```
{
  "paymentId": "pay_789",
  "status": "PENDING"
}
```

and the retry happens ten minutes later, after settlement:

```
{
  "paymentId": "pay_789",
  "status": "SETTLED"
}
```

That may be useful, but it is not a replay. It is a fresh read of the resource. If your API contract says idempotent retries return the original creation result, you need to store enough to do that.

Schema changes make this worse.

Version 2 response:

```
{
  "paymentId": "pay_789",
  "status": "PENDING"
}
```

Version 3 response:

```
{
  "id": "pay_789",
  "state": "PENDING",
  "createdAt": "2026-05-07T10:00:00Z"
}
```

If a generated client retries after a deploy, should it receive the stored v2 response or a reconstructed v3 response? Both can be defensible. They are different contracts.

A common compromise is to store:

```
resource_type = payment
resource_id = pay_789
response_status = 201
response_schema_version = v2
```

and store full response bodies only for endpoints where exact replay matters. If you store bodies, treat the idempotency table like sensitive data storage, not like a harmless cache.

## Your queue consumer has the same bug

HTTP gets most of the attention because the header is visible. A lot of duplicate side effects happen later, in consumers, outbox publishers, inbox processors, and notification workers.

Suppose the payment service publishes:

```
{
  "eventId": "evt_100",
  "type": "PaymentCreated",
  "paymentId": "pay_789",
  "accountId": "acc_1",
  "amount": "10.00",
  "currency": "EUR"
}
```

A consumer receives it twice. That should not send two emails, create two ledger entries, or notify a provider twice.

The dedupe key might be the event ID, message ID, operation ID, aggregate ID plus version, or a business key such as `ledger_payment_pay_789`. The right answer depends on the side effect.

A consumer inbox table might be:

```
consumer_inbox

- consumer_name
- message_id
- status
- processed_at
- error_code

unique(consumer_name, message_id)
```

But marking the message processed is not trivial.

If you mark it processed before sending the email and then crash, the retry skips the email forever. If you send the email before marking it processed and then crash, the retry may send it again. The usual answer is to make the side effect durable before sending it: insert an email notification row with a unique key, then have a sender process that row.

Ledger entries often have a natural idempotency key:

```
unique(ledger_entry_type, source_payment_id)
```

Processing `PaymentCreated(pay_789)` twice attempts to create the same ledger entry twice, and the second attempt resolves to the existing entry.

Many production queue integrations are effectively at-least-once from the consumer’s point of view. Even when the broker advertises stronger delivery semantics, your business side effects still need deduplication. Exactly-once delivery is not exactly-once business effect. The latter usually comes from durable operation IDs, unique constraints, idempotent writes, and recovery paths.

Outbox/inbox is the usual shape:

```
same database transaction:
  insert payment row pay_789
  insert outbox event PaymentCreated(pay_789)

publisher:
  reads unpublished outbox event
  publishes event with eventId
  marks outbox event published

consumer:
  deduplicates by eventId or business operation key
  writes side effect behind a unique constraint
```

Idempotency prevents some duplicates. It does not remove poison messages, broken providers, dead-letter handling, or recovery work.

## Expiry is part of the API contract

Idempotency records cannot usually live forever.

If the server promises a 24-hour idempotency window, then a retry after 25 hours may create a new operation. That may be acceptable. It may also surprise clients that queue retries for days. The replay window is a product/API decision, not just a cleanup setting.

A completed record might be:

```
created_at: 2026-05-07T10:00:00Z
expires_at: 2026-05-08T10:00:00Z
status: COMPLETED
```

After expiry, you might delete the response body but retain metadata longer:

```
idempotency_key
scope
operation_name
request_hash
resource_id
created_at
expires_at
```

That supports diagnostics without retaining sensitive response payloads.

Stale `IN_PROGRESS` needs separate handling:

```
status: IN_PROGRESS
resource_id: pay_789
updated_at: 2026-05-07T10:00:00Z
locked_until: 2026-05-07T10:00:30Z
now: 2026-05-07T10:45:00Z
```

A retry that sees this should not blindly execute again. It should acquire recovery ownership, inspect `pay_789`, query downstream if needed, and move the operation to `COMPLETED`, `FAILED_RETRYABLE`, or `UNKNOWN_REQUIRES_RECOVERY`.

Cleanup jobs should not remove in-progress records just because they are old. An old in-progress row may mean a stuck worker, a process crash, or an operation waiting for reconciliation. Deleting it can allow a duplicate side effect.

Bad cleanup:

```
delete
from idempotency_requests
where expires_at < now();
```

Better options include deleting in small batches, partitioning by `expires_at`, dropping old time partitions after the replay window, and keeping separate retention policies for response bodies and metadata.

Replay count is mostly capacity planning. Different-body reuse, stale `IN_PROGRESS` rows, expired retries, and unknown states are the metrics that find bugs.

```
idempotency.replay.count
idempotency.conflict.different_request.count
idempotency.in_progress.age.max
idempotency.expired_retry.count
idempotency.unknown_state.count
```

## Failure replay is a policy decision

The dangerous mistake is treating every failure as either “safe to retry” or “completed”.

Pure syntactic validation failures usually do not need idempotency storage. If the JSON is malformed or a required field is missing, repeating the request will fail again.

Business rejections are different. If the decision depends on mutable state, such as balance, inventory, account status, or fraud rules, decide whether the first decision is binding for that idempotency key or whether the client must retry with a new key.

A deterministic rejection might be replayable:

```
{
  "errorCode": "INSUFFICIENT_FUNDS",
  "message": "The account has insufficient funds for this payment."
}
```

But if the account balance changes five seconds later, replaying that rejection may or may not be what your API intends.

Authentication failures should not create idempotency records. For authorization failures, be careful: a retry must still resolve to the same scope/principal that created the original record. Do not let one caller use another caller’s idempotency key to discover whether an operation happened. Whether later permission changes block replay of an already completed authorized operation is a product and security decision.

Rate limits usually should not be recorded as completed idempotent outcomes. A retry later might be allowed.

Server error before side effects can often allow retry. Server error after side effects is dangerous. If you created the payment but failed to serialize the response, the retry should not create another payment. If you called a provider and lost the response, the retry needs recovery state, not optimism.

A practical internal status set might be:

```
IN_PROGRESS
COMPLETED
FAILED_REPLAYABLE
FAILED_RETRYABLE
UNKNOWN_REQUIRES_RECOVERY
EXPIRED
```

Do not expose every internal state directly. But internally, pretending every failure is either “done” or “not done” makes recovery harder.

## When one transaction cannot cover the operation

The useful distinction is not monolith versus microservices. It is whether one durable transaction can cover the operation.

If one database transaction can cover the idempotency row, payment row, and outbox record, the local part is straightforward:

```
insert idempotency row
insert payment row
insert outbox event
mark idempotency completed
commit
```

The publisher can retry outbox delivery. Consumers deduplicate by event ID or business operation key. The local write path is much easier to reason about.

When side effects cross boundaries, every boundary that can repeat work needs its own duplicate-suppression rule.

An upstream API accepting `Idempotency-Key: abc-123` can prevent duplicate HTTP payment creation requests at the edge. It does not automatically prevent duplicate ledger entries, duplicate notifications, duplicate provider calls, or duplicate read-model updates.

A better model is to maintain stable operation identities:

```
client idempotency key: abc-123
payment operation id: payop_456
payment id: pay_789
ledger entry id: ledger_payment_pay_789
email dedupe key: receipt_payment_pay_789
provider idempotency key: provider_payment_pay_789
```

The names do not matter. The point is that each side effect has a durable identity appropriate to that side effect.

In active-active multi-region deployments, a region-local idempotency table only protects retries that land in the same region. You either need to route all requests for the same scoped key to a home region, use a strongly consistent shared store for idempotency records, or rely on downstream business constraints that survive cross-region races. Async replication alone can allow two regions to accept the same key before either sees the other write.

For high-throughput APIs, the idempotency table can become a hot path. Response bodies can become expensive. Cleanup can compete with traffic. Partition by tenant, hash, or time if needed. Know your replay window. Do not make a global table the bottleneck unless the duplicate harm justifies it.

## When not to build a general idempotency layer

The cost is not the header. The cost is the durable memory and recovery behavior behind it.

Do not build a payment-grade idempotency layer for an admin action where a duplicate is harmless and visible.

For read-only operations, idempotency keys usually add noise.

If a duplicate analytics event costs almost nothing and can be corrected downstream, a heavy idempotency table may be the wrong trade.

For some operations, a business key is better than a random key:

```
unique(account_id, merchant_reference)
```

If the business rule is “there can be only one payment per merchant reference per account,” that constraint catches duplicates even when the client retries with a new random key by mistake. Random idempotency keys only help when the client reuses the same key for retries.

For other operations, change the resource model:

```
PUT /accounts/acc_1/settings/default-currency
```

```
{
  "currency": "EUR"
}
```

Repeating that request leaves the setting as EUR. You still need to think about side effects, but the operation shape is helping you.

Client-generated keys are useful when the client can identify a retry of the same operation. Properly generated random keys are usually enough; timestamp-only keys, counters, and keys derived from sensitive data are not. Scope the key to the caller and operation, for example `(tenant_id, operation_name, idempotency_key)`, so a bad client only collides with itself. If clients generate a new key on every attempt, you need a business key or a server-created operation resource.

Use the amount of harm caused by duplicate side effects, the likelihood of retries, and the difficulty of detecting duplicates after the fact to decide how much machinery you need.

If duplicates move money, notify humans, call providers, consume scarce inventory, or corrupt accounting, spend the design effort. If duplicates are harmless, rare, and easy to clean up, use a smaller mechanism.

## Failure modes worth testing

Here are tests I would rather see than a dozen happy-path unit tests.

### Same key, same canonical command, completed

First request creates the payment:

```
POST /payments
Idempotency-Key: abc-123
```

returns:

```
201 Created
```

with `paymentId = pay_789`.

Second request with the same canonical command and key returns the same stored result or documented equivalent. It does not create `pay_790`. It does not publish a second `PaymentCreated` event.

### Same key, different canonical command

First request:

```
{
  "amount": "10.00",
  "currency": "EUR"
}
```

Second request:

```
{
  "amount": "100.00",
  "currency": "EUR"
}
```

Same key.

Expected behavior: reject with a stable machine-readable idempotency conflict. Log and count it.

### Two concurrent identical requests

Start two requests at the same time with the same key and same command.

Expected behavior: one wins execution. The other sees `IN_PROGRESS`, waits and replays, or returns a retry-later response. The side effect executes once.

If this test passes without a unique constraint or atomic insert, be suspicious of the test.

### Timeout after downstream success

Simulate provider success and then crash before the client receives the response.

Expected behavior: the retry should not call the provider with a new operation identity. It should find local completed state, query provider idempotent state, or move into recovery.

### Duplicate message from a queue

Deliver `PaymentCreated(pay_789)` twice.

Expected behavior: one ledger entry, one email notification, one provider notification. If the first attempt fails halfway through, the retry should complete missing durable work without duplicating completed work.

### Expired or stale state

Retry after the idempotency record expired. Retry while the record is stale `IN_PROGRESS`. Retry after response schema changed. Retry from another region if your deployment allows it.

These are not exotic cases. They are the normal edges of retrying over networks.

## Checklist before shipping

- Reject same scoped key plus different canonical command.
- Use a unique constraint or atomic insert on the scoped key.
- Hash the validated command, not raw JSON bytes.
- Treat `IN_PROGRESS` as API-visible behavior.
- Define fresh, stale, completed, retryable failure, replayable failure, and unknown states.
- Store enough response data to satisfy your replay contract.
- Make downstream calls idempotent too, or have reconciliation.
- Use outbox/inbox patterns where events and queues are involved.
- Do not mark messages processed before their durable side effects exist.
- Define the idempotency window as part of the API contract.
- Retain metadata separately from sensitive response bodies if needed.
- Test concurrent duplicates, timeout after downstream success, partial failure, expiry, and schema-change replay.
- Monitor different-body reuse, stale `IN_PROGRESS`, expired retries, unknown states, and replay rates.

## The second request is not a repeat until proven

The easy version of idempotency remembers that a key was seen.

The useful version remembers what the key meant.

For `POST /payments`, that means remembering the scoped operation, the canonical command, the execution state, the resulting resource or response, the expiry window, and enough failure state to avoid turning uncertainty into duplicate side effects.

The second request may be a retry. It may be a different operation wearing the same key. It may be racing the first request. It may arrive after the provider succeeded but your process failed. It may arrive after your cleanup job deleted the only memory of what happened.

The server has to prove which case it is.

The key is not the guarantee. The guarantee is that the server remembers the first operation precisely enough to replay it, reject a mismatch, or recover instead of guessing