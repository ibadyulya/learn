# ACID and transactions — solutions

> 🌐 Russian version: [04-acid-transactions.answers.md](../ru/04-acid-transactions.answers.md)

---

## Question 1
A transaction is a logical unit of work of several operations applied **wholly or not
at all**. An example of trouble without atomicity: a money transfer — debited the
sender, but a failure before crediting the receiver. Without "all or nothing" the money
**vanishes**; a transaction rolls the debit back.

## Question 2
- **A (Atomicity)** — all or nothing; failed — full rollback.
- **C (Consistency)** — a transition between correct states without violating
  constraints/invariants.
- **I (Isolation)** — parallel transactions don't interfere (per the isolation level).
- **D (Durability)** — the committed survives a crash (written to disk).

## Question 3
The DB keeps an **undo log**: during the transaction it remembers how to undo each
change. On `ROLLBACK` (explicit or due to a crash/error) it applies the undo and returns
the data to the state **before BEGIN**. So a "half-done" state doesn't exist — from
outside you see either the result or nothing.

## Question 4
**Durability** — once committed, data isn't lost on a crash. **WAL**: changes are first
appended to a **journal on disk** and flushed (`fsync`), and only that counts as a
commit; the data pages themselves may apply later. You write to the journal **before**
the data so recovery is possible at any crash point: journal present → redo the
unwritten; no journal → the transaction wasn't considered committed.

## Question 5
- **C in ACID** — integrity **within one DB**: a transaction doesn't violate constraints
  and business rules.
- **C in CAP** — consistency **across nodes/replicas** in a distributed system: everyone
  sees the same fresh data.
Different things that happen to share a letter.

## Question 6
A **savepoint** is a point within a transaction you can roll back to (`ROLLBACK TO`)
**without cancelling the whole** transaction. Useful for a partial rollback: e.g., try an
optional step and roll back only it on error, keeping what's already done.

## Question 7
An ACID transaction is atomic **within one DB**. The DB and the broker are **two
different systems without a shared transaction**, so "write to the DB + publish an event"
can't be wrapped atomically — a crash between them loses/duplicates (the dual-write
problem). Instead of a simple transaction you use a **transactional outbox**, a **saga**
(a sequence with compensations), or **2PC**, plus **idempotency** on the consumer.
