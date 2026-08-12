# Saga and 2PC (distributed transactions)

> 🌐 Russian version: [09-saga-2pc.md](../ru/09-saga-2pc.md)

How to achieve consistency for an operation across several services when there's no
shared transaction. Reads as a story; a continuation of
[ACID transactions](../../05-databases/en/04-acid-transactions.md) and
[reliability-patterns](./03-reliability-patterns.md).

The through-line of the topic:

> **An ACID transaction is atomic only within one DB. When an operation spans several
> services/DBs, there's no simple transaction. Two answers: 2PC — strict atomicity via
> a coordinator, but blocking and fragile; saga — a chain of local transactions with
> compensations, giving eventual consistency. In microservices it's almost always
> saga.**

---

## The problem

The "place an order" operation = debit in payments + reserve in the warehouse + create
the order. These are **three services, three DBs, no shared transaction** — you can't
wrap them atomically (the dual-write problem). A crash midway leaves the system in an
inconsistent state. You need a way to coordinate a distributed operation.

---

## 2PC — two-phase commit

Strict atomicity via a **coordinator** and two phases:
1. **Prepare (voting)** — the coordinator asks each participant: "ready to commit?" Each
   **reserves** the changes and answers yes/no, but **doesn't commit**.
2. **Commit/Abort** — if **all** said yes → the coordinator tells everyone to commit; if
   even one said no → everyone rolls back.

It gives true atomicity across nodes, but the downsides are heavy:
- **blocking** — participants hold resources locked between phases; if the coordinator
  crashes midway → participants **hang** in uncertainty;
- **the coordinator is a single point of failure**;
- **slow**, scales poorly.
So in microservices 2PC is used rarely.

---

## Saga — a chain with compensations

Instead of one atomic transaction — a **sequence of local transactions**, one per
service. Each step commits **its own** DB and triggers the next. If a step fails,
**compensating transactions** run, undoing the previous steps **semantically** (not a
rollback, but a reverse action).
```
payment OK → warehouse reserve OK → order creation FAIL
   ↓ compensations in reverse order:
cancel the reserve ← refund the payment
```
The result — **eventual consistency**: the system reaches a consistent state, but not
instantly, and visible intermediate "half-states" are possible (an order "in progress").

**Compensation ≠ rollback:** the money is already debited and committed — you can't "roll
it back", only **make a refund** (a new transaction). So steps are designed reversible and
**idempotent** (see [reliability-patterns](./03-reliability-patterns.md)).

---

## Orchestration vs choreography

Two ways to wire the saga steps:
- **Orchestration** — a central **orchestrator** conducts: explicitly calls steps and
  compensations. The logic is in one place (the whole process is visible), but the
  orchestrator is a node of complexity.
- **Choreography** — no center: each service **listens for an event** and publishes its
  own (see [event-driven-messaging](./02-event-driven-messaging.md)). Looser coupling, but
  the process is "smeared" — harder to understand and debug.

---

## When to use what

- **2PC** — rarely, where strict atomicity is needed and participants support it (some
  DBs/brokers), and the scale is small.
- **Saga** — the standard for microservices: long business processes across services where
  eventual consistency is tolerable and fault isolation is mandatory.

---

## Interview phrasing

> "ACID is atomic within one DB; for an operation across several services there's no
> simple transaction. 2PC — a coordinator + two phases (prepare-voting, then
> commit/abort): it gives atomicity but is blocking, the coordinator is a SPOF, it's
> slow, so it's rare in microservices. Saga — a sequence of local transactions, each
> commits its own DB and triggers the next; on failure, compensations run (a semantic
> undo, not a rollback — e.g., a refund), the result is eventual consistency. Steps are
> wired by orchestration (a central conductor) or choreography (events). I make steps
> idempotent. By default I take saga."

---

See [09-saga-2pc.tasks.md](./09-saga-2pc.tasks.md) — tasks. Solutions in
[09-saga-2pc.answers.md](./09-saga-2pc.answers.md).
