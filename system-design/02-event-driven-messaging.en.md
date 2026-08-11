# Event-driven and message queues

Why an event-based architecture is needed, who its participants are, and what
delivery guarantees exist. Builds on [01-foundations](./01-foundations.en.md);
next — how to make it reliable in
[03-reliability-patterns](./03-reliability-patterns.en.md).

> 🌐 Russian version: [02-event-driven-messaging.ru.md](./02-event-driven-messaging.ru.md)

---

## Why event-driven

The ordinary "direct" call: the orders service itself synchronously calls email,
analytics, warehouse over HTTP. What's bad:
- **Coupling** — orders must *know* about everyone it pokes; a new service →
  changes in the orders code.
- **Fragility** — email is down → order placement fails; a slow consumer slows
  everyone.
- **Spikes** — 10,000 orders/sec must be digested by all dependent services right
  now.

Event-driven flips this: the orders service **calls no one**, it **announces a
fact** — "order #42 created" — and throws an event into a **broker**. Whoever is
interested **subscribes themselves** and reads at their own pace.

Three benefits (to name in the interview):
- **Decoupling:** the sender doesn't know the receivers; you can add consumers
  without changing the producer.
- **Resilience:** a consumer went down → messages wait in the broker, placement
  doesn't suffer; it recovered → caught up.
- **Buffering (spike smoothing):** a burst turns into a queue, not a crash.

## Producer / broker / consumer

- **Producer** — publishes a message (the orders service).
- **Broker** — the intermediary: stores messages and hands them out (Kafka /
  RabbitMQ).
- **Consumer** — reads and processes (email, analytics).

The key thing — **the intermediary breaks the direct link in time and space**:
the producer and consumer needn't even be online at the same time. The producer
threw it and left, the consumer comes an hour later and picks it up. This
"decoupling in time" is not something a direct HTTP call gives you.

## Delivery semantics

As soon as there's a network between the parties — a message can be **lost** or
arrive **twice**. Hence three levels of guarantee (a mandatory question):

1. **At-most-once** — send and forget. **No duplicates, but loss is possible.**
   For things you don't mind losing: metrics, optional logs. "Better to skip than
   to repeat."
2. **At-least-once** — no processing acknowledgement → send again. **No losses,
   but duplicates are possible.** The workhorse, ~90% of systems. "Better to
   repeat than to lose."
3. **Exactly-once** — neither lost nor duplicated. The dream.

**The main thesis:**

> **True exactly-once, strictly speaking, does not exist in a distributed
> system.** Two parties over an unreliable network cannot with full guarantee
> agree "processed exactly once" — there's always a moment where one side doesn't
> know whether it reached the other.

What's called "exactly-once" is **at-least-once + idempotency**: we allow
duplicates at the *delivery* level (send until acknowledged), but make the
*processing* such that a repeat changes nothing. Externally "exactly once",
internally "delivered some number of times, effect applied once" — this is
**effectively-once**.

**The backbone formula** (unpacked by
[03-reliability-patterns](./03-reliability-patterns.en.md)):
```
at-least-once (delivery)  +  idempotency (processing)  =  effectively-once
```
