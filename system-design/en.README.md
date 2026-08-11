# System Design — navigation

The system design area. One topic = one file, read in numeric order. The overall
prep plan — in [../en.PLAN.md](../en.PLAN.md).

> 🌐 Russian version: [ru.README.md](./ru.README.md)

The through-line of the area: **the network is unreliable, nodes fail. Almost
every pattern is an answer to "you can't guarantee delivery/processing exactly
once".** The interview meta-rule: reason about **trade-offs**, start your answer
with "it depends on…".

---

## Block A — Messaging & reliability ✅

| # | File | About | Status |
|---|---|---|---|
| 01 | [foundations](./en.01-foundations.md) | distributed systems, CAP/PACELC, trade-off thinking | ✅ |
| 02 | [event-driven-messaging](./en.02-event-driven-messaging.md) | why event-driven, producer/broker/consumer, delivery semantics | ✅ |
| 03 | [reliability-patterns](./en.03-reliability-patterns.md) | ack/nack, idempotency, outbox, retries+backoff, DLQ, rate limiting, crash recovery | ✅ |
| 04 | [kafka-vs-rabbitmq](./en.04-kafka-vs-rabbitmq.md) | log vs queue, offset/partitions vs exchange/competing consumers | ✅ |

The backbone of the block: `at-least-once (delivery) + idempotency (processing) =
effectively-once`. Honest exactly-once at the delivery level does not exist.

## Block B — Scalability & infrastructure

| # | File | About | Status |
|---|---|---|---|
| 05 | [scaling](./en.05-scaling.md) | vertical/horizontal Node, cluster/PM2, stateless, worker_threads | ✅ |
| 06 | [scaling-data](./en.06-scaling-data.md) | load balancing (L4/L7), replication (leader/follower, lag), sharding (shard key) | ✅ |

Still planned (files appear as topics are covered):
- caching and CDN (strategies, invalidation);
- idempotent HTTP APIs, saga/2PC (block C).

## Block C — misc ⏳

- idempotent HTTP APIs (Idempotency-Key on REST);
- consistency between services (saga, 2PC vs eventual).

> Adjacent: transactions and **isolation levels** — in a separate `databases/`
> module (that's about a single DB, not distribution). Replication/sharding live
> on the boundary — we cover them once and cross-reference.
