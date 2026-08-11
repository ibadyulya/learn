# Reliability patterns

How to make delivery reliable: ack/nack, idempotency, transactional outbox,
retries + backoff, DLQ, rate limiting, crash recovery. A continuation of
[02-event-driven-messaging](./en.02-event-driven-messaging.md); Kafka/Rabbit
concepts — in [04-kafka-vs-rabbitmq](./en.04-kafka-vs-rabbitmq.md).

> 🌐 Russian version: [ru.03-reliability-patterns.md](./ru.03-reliability-patterns.md)

The through-line of the whole block:

> **In a distributed system the network is unreliable, and processes crash.
> Everything — ack, retries, idempotency, outbox, DLQ — is an answer to one
> brutal fact: you can't guarantee a message is delivered and processed exactly
> once. You don't fight this fact, you engineer around it.**

---

## ack/nack — how the broker knows a message was processed

The broker needs a way to learn whether a message arrived and was processed. This
is an **acknowledgement**. Mechanics: the broker hands out the message, but
**doesn't discard it right away** — it holds it "in suspense" (in-flight) and
waits:

- **ack** — "processed successfully" → only now does the broker delete / advance
  the pointer;
- **nack** — "couldn't" → the broker **returns it to the queue** (redelivers) or
  sends it to the DLQ.

Key: **a message stays alive until it's explicitly acknowledged** — insurance
against loss. The consumer goes silent (crashed, hung) → the broker, on
timeout/disconnect, decides "not processed" and **delivers again**.

### Where the "window" for duplicates is born (the answer to "double executions")

```
1. received the message
2. processed it (debited money)
3. sent the ack
```
Between 2 and 3 — a **gap**. Crash exactly here (money debited, ack didn't go
out) → the broker: "no ack arrived → not processed → I'll deliver again" →
debiting **a second time**. A duplicate.

Conversely, ack **before** processing (3 before 2): acknowledged, started,
crashed → the broker already discarded the message → **processing lost**. These
are the two semantics:

- **ack after processing** → at-least-once → risk of a **duplicate**;
- **ack before processing** → at-most-once → risk of a **loss**.

The gap between "did it" and "acknowledged it" is unavoidable. We usually don't
tolerate loss → we choose **ack after processing (at-least-once)** and **accept
duplicates** → we neutralize them with idempotency.

---

## Idempotency — neutralize the repeat

**An idempotent operation is one that leaves the system in the same state whether
run once or ten times.** A repeat adds nothing.

- "switch to OFF" — idempotent; "toggle the light" — not.
- `balance = 100` — idempotent; `balance -= 10` — **not** (5 repeats = −50).

Our duplicate from the ack gap is "debit 10" twice. The task is to make it so the
**second execution of the same message has no effect**. Three techniques:

**1. Idempotency key (deduplication).** The message has a unique `id` (or a
business key `payment_id`). The consumer keeps a record of processed ones:
```
seen id before?  yes → do nothing, just ack (a neutralized duplicate)
                 no → process, save id as done, ack
```
Subtlety (interviewers love it): **saving the id and the effect itself must be
atomic** (one DB transaction). Otherwise a new gap: processed, but didn't record
the id → crashed → the repeat processes again. "Debit money" and "mark the id
processed" commit together.

**2. Natural idempotency through state.** Design it as "set", not "change":
`UPDATE order SET status='paid'` is idempotent, `balance -= 10` is not.

**3. Unique DB constraints.** `INSERT` with a unique `order_id` → a repeat fails
on the constraint → we catch the duplicate error and ack. The database is the
deduplicator itself.

### The circle closes
```
at-least-once (ack after processing — no loss)
  + idempotency (dedup by id / state / constraint — no double effect)
  = effectively-once (externally "exactly once")
```
The full answer about "double executions": duplicates are inevitable because of
the gap between processing and ack → we make delivery at-least-once, processing
idempotent → together "exactly once", even though there's no honest exactly-once
at delivery.

---

## Transactional Outbox — the mirror gap on the producer side

**The dual-write problem.** The producer must do two things:
1. write the order to **its DB**;
2. publish an event to the **broker**.

These are **two different systems without a shared transaction** — you can't wrap
them atomically. A crash in the gap breaks everything:

- **DB → then broker**, crash between: the order is in the database, **the event
  wasn't sent** → email/warehouse never learned. **A lost event.**
- **broker → then DB**, crash between: the event flew off to everyone, **the
  order isn't in the database** → an email about a non-existent order. **A phantom
  event.**

Ordering doesn't cure it. The solution — **remove the second system from the
dangerous moment**: write only to the DB, and there we know how to be atomic.

**Idea:** instead of "DB + broker" we do **one transaction into one DB**, writing
two things — the business data and the event into an `outbox` table:
```sql
BEGIN;
  INSERT INTO orders (...);              -- business data
  INSERT INTO outbox (event, payload);   -- the event "to be sent"
COMMIT;                                   -- atomic: either both or neither
```
The intermediate "order exists, event doesn't" no longer happens — the gap is
closed.

**Who publishes — the message relay** (publisher/forwarder), a separate process:
```
read unsent rows from outbox → publish to the broker → mark sent=true
```
The relay needn't be perfect: published but crashed before marking → on the next
lap it publishes **again** → a duplicate in the broker → caught by the
**idempotent consumer**. That is, the outbox gives at-least-once on the producer,
duplicates are quenched by idempotency on the consumer.

**Two ways to relay:**
- **Polling publisher** — a periodic `SELECT ... WHERE sent=false`. Simple, but
  latency and load on the DB.
- **CDC (Change Data Capture)** — the relay reads the DB's transaction log (WAL in
  Postgres) via **Debezium**, publishing instantly without polling. More
  efficient, but more complex.

### Two gaps — one principle

| Where the gap is | Between what | Cured by |
|---|---|---|
| **Producer** | "wrote to DB" ↔ "sent to broker" | **transactional outbox** |
| **Consumer** | "processed" ↔ "sent the ack" | **idempotency** |

Both — "reduce to a single atomic operation + at-least-once + idempotency".

### Important: the ack can't be made atomic either

A common mistake — thinking that on the consumer you can commit the processing
**together with the ack** to the broker. You can't: the ack is a message to the
**broker**, a separate system. "commit to DB + ack to broker" is **the same
dual-write** as on the producer.

What you *can* do atomically — only what's inside one DB:
```sql
BEGIN;
  UPDATE balance ...;                 -- business effect
  INSERT INTO processed (message_id); -- mark "id processed"  (table = inbox)
COMMIT;                                -- the ack is NOT part of this
```
The ack stays outside and is unreliable — and that's fine: crash before ack →
redelivery → dedup by id → just ack. **Idempotency exists precisely to cover the
non-atomic ack.**

Full symmetry: on both sides we atomically commit **only what's in the DB** (the
business action + metadata about the message: outbox / inbox), and we move
communication with the broker outside and make it at-least-once. Outbox and
idempotency are **one solution** for the outgoing and incoming gap.

**Kafka caveat (if they dig into exactly-once):** Kafka can do real transactions
"read → process → write → advance offset" atomically — but **only while
everything is inside Kafka** (Kafka-to-Kafka). As soon as a side effect is in an
**external DB** (debit money in Postgres) — dual-write again, Kafka's transaction
doesn't save you. "Exactly-once in Kafka" is about stream processing inside
Kafka, not about "debited money exactly once". For external effects — still
at-least-once + idempotency.

> Phrasing: "The dual-write problem: the DB and the broker are different systems
> without a shared transaction, so writing the order and publishing the event
> aren't atomic, and a crash between them either loses the event or sends a
> phantom one. Transactional outbox: the event is written to an outbox table in
> the same transaction as the business data, and a separate relay (polling or CDC
> via Debezium) publishes it and marks it sent. The relay is at-least-once,
> duplicates are quenched by an idempotent consumer. The same mirrored on the
> consumer: the ack isn't atomic either, so we atomically commit the effect + the
> processed message's id, and quench redelivery with dedup."

---

## Retries + backoff — repeat, but wisely

No ack arrived → the broker redelivers = a retry. The question is **how**. The
naive "crashed → retry immediately, in a loop" is a trap:

- **retry storm** — you bombard an already-down service with a barrage of
  retries, preventing it from getting up;
- you burn CPU/network spinning one failure a thousand times;
- if the service went down from overload — the retries **finish it off**, it
  never recovers.

So repeat **not immediately and not infinitely.**

**Backoff** — with each failure we wait longer. Usually **exponential** (the
pause doubles): 1s → 2s → 4s → 8s → 16s. Didn't recover in a second — give it a
longer breather; the exponent quickly spreads attempts out in time.

**Jitter** — a random spread added to the pause. Without it, 10,000 messages that
went down at once wait exactly 1s, 2s… and **repeat in a synchronized volley**
(**thundering herd**). Jitter (`4s ± random`) smears the retries out evenly. Rule:
**backoff spreads attempts in time, jitter keeps them from synchronizing.**

**Attempt limit (max retries).** Failures come in two kinds — a key distinction:
- **transient:** the network blinked, overload, a deadlock — it'll pass on its
  own → **a retry makes sense**;
- **permanent:** broken JSON, a reference to a non-existent user, a bug in the
  handler — **a retry is useless**, it'll always fail. This is a **poison
  message**.

Without a limit a poison message gets stuck forever (delivered → crashed →
redelivered → …), **blocking the queue**: one broken JSON stops the whole
pipeline. Where to put it → DLQ.

## DLQ — Dead Letter Queue

**A separate queue where a message goes after exhausting all retries** (a "dead
letter" — a postal term: undeliverable, set aside).
```
fails → retry (backoff) → ... → N attempts exhausted
   → moved to the DLQ and out of the main queue
   → the main queue is unblocked, the rest go on
```
What for (name three):
1. **unblock the pipeline** — the poison is off the road, healthy ones go;
2. **don't lose the problem** — it sits in the DLQ, you can study the cause;
3. **replay** — after a fix, return it to the main queue and process it again.

DLQ = **quarantine**: not in the main flow (doesn't get in the way), not in the
trash (not lost). We hang an **alert** — landed in the DLQ → "come take a look",
often a symptom of a bug/a downed dependency.

> Phrasing: "Retry with exponential backoff and jitter — otherwise a retry storm
> finishes off the failing service, and a thundering herd hits in volleys.
> Retries aren't infinite: a transient failure passes on its own, a permanent one
> (poison message) is pointless to retry and it'll block the queue. After N
> attempts → DLQ (quarantine): the main flow is unblocked, the message isn't
> lost, you can study it and replay after a fix. On the DLQ — an alert."

---

## Rate limiting — capping the rate

Protect the system by not letting it process/send faster than a set pace. What
for: not to swamp an external API with limits; not to let one client eat up
resources (protection against abuse/DDoS); keep your own DB within comfortable
bounds.

**Token bucket.** Tokens drip into the bucket at a constant rate (10/sec), the
capacity is limited (100). A request **takes a token**: there is one → passes,
none → throttled. The trick — **it allows bursts**: a full bucket accumulated → a
volley of 100 requests passes at once. "Average rate is capped, a short burst is
allowed."

**Leaky bucket.** Requests pour in from the top, **leak out the bottom strictly
at a constant rate**; overflow is dropped. The trick — **it smooths the output
into a perfectly even stream**, bursts on the output never happen.

The difference: **a token bucket allows bursts (you spend what you accumulated),
a leaky bucket levels into a constant stream.** The API can survive a burst →
token bucket; you need a strictly even pace → leaky bucket.

## Crash recovery — what happens on a crash

Not a separate pattern, but a stress-test of "what if it crashes right here". If
you understood ack/idempotency/outbox — the answer is already there.

The principle: **an unacknowledged message (no ack) isn't lost** — the broker
holds it in-flight and **redelivers** it after a crash. A walkthrough by crash
point:

| Crashed… | What happens to the message | Result |
|---|---|---|
| before processing | untouched, no ack | redelivery, from scratch — fine |
| in the middle | the DB transaction rolled back, no ack | redelivery, fresh from a clean slate — fine |
| after commit, before ack | the effect + id are committed, no ack | redelivery → dedup by id → ack — fine |

The middle row — why **DB transactionality** saves you: uncommitted processing
rolls back by itself, "half-processed" states don't happen (all or nothing). DB
atomicity + redelivery + idempotency = work isn't lost and isn't duplicated.

> Phrasing: "Crash recovery rests on the fact that an unacknowledged message
> isn't lost — the broker redelivers after a crash. Unfinished processing rolls
> back with the DB transaction, half-processed states don't occur. Redelivery is
> quenched by idempotency. At any crash point: either we start over, or we
> recognize a duplicate — but we don't lose it and don't execute it twice."
