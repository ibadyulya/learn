# Locking and MVCC

> 🌐 Russian version: [05-locking-mvcc.md](../ru/05-locking-mvcc.md)

How the DB implements isolation under the hood. Reads as a story; a continuation of
[isolation levels](./01-isolation-levels.md).

The through-line of the topic:

> **To keep parallel transactions from interfering, the DB has two tools: lock access
> (locks) or give each its own snapshot of the data (MVCC). Locks come in two kinds:
> pessimistic (forbid in advance) and optimistic (check at commit). MVCC is the key
> idea of modern DBs: readers don't block writers.**

---

## Pessimistic locking — "forbid in advance"

You assume a conflict is **likely**, so you **lock** the data while you work. Others
wait.
```sql
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;   -- took a row lock
-- until COMMIT/ROLLBACK, other transactions wait for this row
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;                                            -- lock released
```
Kinds: **shared (S, for reading)** — several readers at once; **exclusive (X, for
writing)** — only one. Good under **high contention** (many edits to one row), but lowers
concurrency — others wait.

## Optimistic locking — "check at commit"

You assume a conflict is **rare**: you work without a lock, and on write you **check**
whether someone changed the data meanwhile. Usually via a **version column**:
```sql
UPDATE items SET price = 200, version = version + 1
WHERE id = 1 AND version = 5;    -- 0 rows updated → someone got ahead → retry
```
If the version didn't match — a conflict, the transaction is **retried**. Good under
**low contention**: no lock overhead, but with frequent conflicts, many retries.

> Rule: **many conflicts → pessimistic; few → optimistic.**

---

## Deadlocks

Two transactions wait for each other's locks in a cycle: T1 holds A and waits for B, T2
holds B and waits for A — **nobody moves**. The DB **detects** the wait cycle and
**aborts** one transaction (the victim) with an error to break the deadlock; the app
retries it. Prevention: take locks **in one order**, keep transactions short.

---

## MVCC — multiversion concurrency control

The key idea Postgres and InnoDB stand on. Instead of locking reads, the DB keeps
**several versions** of a row. Each transaction sees a **snapshot** of the data as of its
moment.
```
The MVCC rule: readers don't block writers, writers don't block readers.
```
How: on a change a row isn't overwritten in place — a **new version** is created, and the
old one remains for transactions to which it's still "visible" (by start time/transaction
number). A reader takes the appropriate version from the chain without waiting for the
writer.

The benefit — **high concurrency**: a long `SELECT` doesn't slow an `UPDATE` and vice
versa. Read Committed and snapshot isolation (RR in Postgres) are built on MVCC. The
price — old versions must be **cleaned** (in Postgres — `VACUUM`).

---

## How this maps to isolation levels

- **Read Committed / Repeatable Read** usually on **MVCC** — a snapshot without locking
  reads (see [isolation-levels](./01-isolation-levels.md)).
- **Serializable** — either aggressive locks, or (Postgres) **SSI** over MVCC: it tracks
  conflicts and rolls back the dangerous transaction with a serialization error.

---

## Interview phrasing

> "The DB implements isolation via locks or MVCC. Pessimistic locks (`SELECT ... FOR
> UPDATE`, shared/exclusive) forbid access in advance — good under high contention, but
> lower concurrency. Optimistic ones (a version column, a check on UPDATE) assume rare
> conflicts — no locks, but retries on a collision. Mutual waiting for locks gives a
> deadlock — the DB detects the cycle and rolls back a victim; fixed by a single lock
> order and short transactions. MVCC — multiversioning: each transaction sees its own
> snapshot, readers don't block writers and vice versa; Postgres/InnoDB stand on it, the
> price is cleaning old versions (VACUUM)."

---

See [05-locking-mvcc.tasks.md](./05-locking-mvcc.tasks.md) — tasks. Solutions in
[05-locking-mvcc.answers.md](./05-locking-mvcc.answers.md).
