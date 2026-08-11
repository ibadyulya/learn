# Kafka vs RabbitMQ

The finale of the block: all the theory from
[02](./02-event-driven-messaging.en.md) and
[03](./03-reliability-patterns.en.md) maps onto two concrete technologies. The
main point — not "features", but **two different worlds by nature**; everything
follows from that nature.

> 🌐 Russian version: [04-kafka-vs-rabbitmq.md](./04-kafka-vs-rabbitmq.md)

> **RabbitMQ — a queue (a message delivered — it's gone). Kafka — a log (a
> message written to a file, it stays, readers move their own "bookmark").**

Metaphors: **a mailbox from which the letter is taken** (Rabbit) versus **a
newspaper everyone reads from the page they stopped on** (Kafka).

---

## RabbitMQ — a smart queue broker
The "deliver and forget" model:
- producer → queue; consumer took it, processed it, **ack** → the broker
  **deletes** the message, it physically disappears;
- it lives until eaten.

Routing (its strength): between the producer and the queues sits an **exchange**,
a smart router, distributing to queues by routing key/patterns. Hence **smart
broker / dumb consumer** — the logic is in the broker.

Our concepts:
- **several consumers on a queue** → **competing consumers**: each message goes
  to **one** of them (that's how you scale processing);
- **DLQ** is built in (dead-letter exchange);
- **order** — FIFO in the queue, competing consumers break it.

Good for: complex routing, command tasks for workers, work queues, RPC. The idea:
**"hand out tasks to workers".**

## Kafka — a distributed log
The "written — it stays" model:
- the producer **appends** to the end of a **topic** (an append-only log);
- a message **doesn't go away after being read**, it stays until **retention**
  (age/size); Kafka deletes **by time/size, not by the fact of being read**;
- a consumer holds an **offset** — a "bookmark"; reads → moves forward, nothing
  is deleted.

Consequences:
- **different consumers read the same thing independently** (each has its own
  offset);
- you can **re-read the past** — rewind the offset;
- **ack = advancing the offset**, the message isn't deleted.

**Partitions** — the mechanics of scale and order: a topic is cut into
partitions, messages are distributed by key (`user_id`):
- **parallelism** — partitions are read by different consumers (a **consumer
  group** divides the partitions among themselves);
- **order is guaranteed only within a partition**, not across the whole topic
  (related events → one partition by key). Between partitions there's no order —
  the price of parallelism.

**dumb broker / smart consumer** — the broker just stores the log, the logic is
on the consumer.

Good for: event streams, high throughput, many independent consumers, event
sourcing, analytics, history rewind. The idea: **"write a stream of events that
many read at their own pace".**

## Comparison

| | RabbitMQ | Kafka |
|---|---|---|
| Model | queue | log |
| Message fate | deleted after ack | stays until retention |
| Metaphor | mailbox | newspaper with a bookmark |
| Progress | ack deletes | offset, message remains |
| Readers | one of the competing consumers | each group reads everything independently |
| Re-read the past | can't (eaten) | can (rewind offset) |
| Order | FIFO in a queue (breaks) | strict within a partition |
| Scaling | competing consumers | partitions + consumer group |
| Routing | rich (exchange) | simple (topic/partition by key) |
| Smarts | smart broker / dumb consumer | dumb broker / smart consumer |
| Profile | commands to workers, RPC | event streams, streaming |

## How to choose (the trade-off from [01-foundations](./01-foundations.en.md))
- complex routing, hand out jobs to workers, each message eaten once →
  **RabbitMQ**;
- high throughput, many independent consumers, history rewind, event sourcing →
  **Kafka**.

> Phrasing: "RabbitMQ — a queue: delivered and deleted, rich routing through an
> exchange, competing consumers split messages — about handing out tasks to
> workers. Kafka — a distributed log: messages stay until retention, each
> consumer holds its own offset and reads independently, you can rewind history;
> order within a partition, parallelism through partitions and consumer groups.
> Rabbit — smart broker/dumb consumer, Kafka the opposite. Choose by task:
> commands to workers → Rabbit, event streams and many readers → Kafka."
