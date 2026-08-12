# ACID and transactions

> 🌐 Russian version: [04-acid-transactions.md](../ru/04-acid-transactions.md)

What a transaction is and what guarantees make it reliable. Reads as a story; the
foundation under [isolation levels](./01-isolation-levels.md).

The through-line of the topic:

> **A transaction is a group of operations executed on an "all or nothing" basis.
> ACID is the four guarantees that turn a set of queries into a reliable unit:
> atomicity, consistency, isolation, durability. Isolation is about concurrency (a
> separate topic), the other three are about a single transaction.**

---

## What a transaction is

A logical unit of work of several operations that must apply **wholly or not at all**.
The classic — a money transfer: debit one and credit another. If there's a failure
after the debit, you can't leave the money "evaporated".
```sql
BEGIN;
  UPDATE accounts SET balance = balance - 100 WHERE id = 1;
  UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;   -- either both updates apply, or ROLLBACK cancels everything
```

---

## A — Atomicity

**All or nothing.** The transaction applies wholly; if it fails midway, a **ROLLBACK**
returns the DB to the state before the start. There's no "half-done" (debited but not
credited). The mechanism — a journal (undo log): the DB remembers how to undo the
unfinished.

## C — Consistency

A transaction moves the DB from one **correct** state to another, without violating
**integrity rules**: constraints, foreign keys, uniqueness, business invariants. If the
result would break a rule, the transaction rolls back. (This is the "C" in ACID — not to
be confused with the "C" in CAP, which is about replica consistency.)

## I — Isolation

Parallel transactions don't interfere; ideally each runs as if alone. In practice it's
governed by **isolation levels** (a separate large topic, see
[isolation-levels](./01-isolation-levels.md)): the stricter, the fewer "leaks", but less
concurrency. Implemented via locking or MVCC (see
[locking-mvcc](./05-locking-mvcc.md)).

## D — Durability

Once a transaction is **committed**, it survives a crash/reboot/power loss: the data is
written to durable storage. The mechanism — a **WAL (write-ahead log)**: changes are
first written to a journal on disk (**before** applying to the data itself), with
`fsync`. A crash after commit — recovery from the journal.

---

## Savepoints — partial rollback

Within a transaction you can set a **savepoint** and roll back to it without cancelling
the whole transaction:
```sql
BEGIN;
  INSERT ...;
  SAVEPOINT sp1;
  INSERT ...;           -- if there's an error here —
  ROLLBACK TO sp1;      -- roll back only to sp1, the first INSERT survives
COMMIT;
```

---

## Distributed transactions — hard

ACID is great **within one DB**. As soon as an operation spans **two systems** (DB +
broker, two microservices), there's no simple transaction — you get the dual-write
problem (see [reliability-patterns](../../06-system-design/en/03-reliability-patterns.md))
and the saga/2PC patterns (see [saga-2pc](../../06-system-design/en/09-saga-2pc.md)). In
distribution you pay for atomicity with consistency/complexity.

---

## Interview phrasing

> "A transaction is a group of operations, all or nothing, ACID gives it four
> guarantees. Atomicity — wholly or rolled back, no half-done (an undo log).
> Consistency — don't violate constraints/invariants, else roll back. Isolation —
> parallel transactions don't interfere, governed by isolation levels (locking/MVCC).
> Durability — the committed survives a crash thanks to WAL (write to the journal
> before the data, with fsync). Savepoints give partial rollback. Within one DB ACID
> works; for an operation across two systems there's no simple transaction — there
> it's saga/2PC and idempotency."

---

See [04-acid-transactions.tasks.md](./04-acid-transactions.tasks.md) — tasks.
Solutions in [04-acid-transactions.answers.md](./04-acid-transactions.answers.md).
