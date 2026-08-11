# Foundations: distributed systems and trade-offs

The bedrock under all of system design. Read first — these ideas underpin
[02-event-driven-messaging](./02-event-driven-messaging.md),
[03-reliability-patterns](./03-reliability-patterns.md) and
[04-kafka-vs-rabbitmq](./04-kafka-vs-rabbitmq.md).

> 🌐 Russian version: [01-foundations.md](../ru/01-foundations.md)

The interview meta-rule: system design tests the **ability to reason about
trade-offs**, not knowledge of a specific technology. The right answer almost
always starts with "it depends on…" — and you have to be able to say *on what
exactly*.

---

## What "distributed system" means

It's when work is spread across **several machines communicating over a
network**. The moment a network appears, so does its fundamental flaw: **the
network is unreliable and can break.** Packets get lost, nodes drop off, part of
the cluster stops seeing another part. This is a **network partition** — not "if"
but "when": over time it always happens.

From this "the network will eventually break" grows the main trade-off.

## The CAP theorem — the famous "three pillars"

Three properties, of which you can't hold all at once:

- **C — Consistency:** any reading node sees **the freshest** data. Once written,
  anyone immediately reads the new value, not a stale one.
- **A — Availability:** the system **always responds**, even if some nodes are
  down. It doesn't hang, doesn't error — it responds.
- **P — Partition tolerance:** the system **keeps working even when the network
  between nodes has broken** and nodes can't see each other.

The theorem: you can't have all three at once; during a break you pick two. But
in practice the **choice isn't of three but of two**, and that distinguishes the
one who understood from the one who crammed:

**P is not optional.** You can't "opt out of partition tolerance" — the network
*will* break regardless of your wishes. So you always hold `P`, and the real
choice happens **at the moment of a break** between `C` and `A`:

- **CP system:** the network broke → better to **refuse/wait** than serve
  possibly stale data. Consistency > availability. (A bank balance: better an
  error than a wrong amount.)
- **AP system:** the network broke → better to **respond with what I have** (even
  if slightly stale) than refuse. Availability > consistency. (A social feed:
  better old likes than a blank screen.)

> Phrasing: "CAP is not a choice of three. Partition tolerance is mandatory,
> because the network breaks regardless of our wishes. The real choice is between
> Consistency and Availability at the moment of a network break: refuse for
> correctness or respond for availability."

**Honest footnote (if they dig):** CAP is criticized for being crude — the
properties aren't binary, and the trade-off matters not only during a break. The
extension **PACELC**: *if* Partition — choose A vs C; *else* (network is fine) —
choose **Latency vs Consistency**. Even without breaks, strict consistency
(waiting for all replicas) costs speed. **Consistency versus latency exists
always, not only during an outage.**

## How this maps onto queues

In Kafka/RabbitMQ you turn exactly these knobs by hand:
- **Reliability vs latency:** wait for a write to disk and to all replicas
  (reliable but slow) or acknowledge immediately (fast, but on a crash you can
  lose data) — the `acks` setting.
- **Order vs parallelism:** strict order (one thread) or scale to many threads
  (order is lost).
- **Strength of the delivery guarantee vs throughput:** exactly-once is more
  expensive than at-least-once.

Hence the order of the conversation: **trade-offs first, technology second.** Not
"I know Kafka", but "what are the requirements for consistency, order, losses —
and *based on that* I pick this".
