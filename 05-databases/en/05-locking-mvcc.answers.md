# Locking and MVCC — solutions

> 🌐 Russian version: [05-locking-mvcc.answers.md](../ru/05-locking-mvcc.answers.md)

---

## Question 1
- **Pessimistic** — assumes a conflict is **likely**: **locks** the data right away
  (`FOR UPDATE`), others wait. "Forbid first".
- **Optimistic** — assumes a conflict is **rare**: works without a lock and **checks** on
  write whether someone changed the data. "Act, verify at the end".

## Question 2
Keep a `version` column. On update require the version to be the one you read:
```sql
UPDATE items SET price=200, version=version+1 WHERE id=1 AND version=5;
```
If someone changed the row meanwhile (version is already 6), the condition won't match →
**0 rows updated** → a conflict. The transaction is **retried** (re-read the current data
and try again).

## Question 3
- **High contention** (many transactions edit the same rows) → **pessimistic**:
  optimistic retries would be too frequent.
- **Low contention** (conflicts are rare) → **optimistic**: no lock overhead, an
  occasional retry.
It depends on the **conflict probability** and the cost of waiting vs retrying.

## Question 4
A **deadlock** is a cyclic wait: T1 holds A and waits for B, T2 holds B and waits for A —
both stuck forever. The DB **detects the cycle** in the wait graph and **aborts one**
transaction (the victim) with a deadlock error, breaking the impasse; the app retries it.
Lower the likelihood: take locks **in one order**, keep transactions **short**, lock
less.

## Question 5
**MVCC (multiversion concurrency control)** — instead of locking reads, the DB keeps
**several versions** of a row, and each transaction sees a **consistent snapshot** of the
data as of its moment. The main rule: **readers don't block writers, and writers don't
block readers** → high concurrency (a long SELECT doesn't hinder an UPDATE and vice
versa).

## Question 6
On a change the row **isn't overwritten in place** — a **new version** is created, and
the old one remains while it's "visible" to transactions that started earlier (by
transaction number/time). A reader takes the appropriate version from the chain. The
price — **accumulation of old versions** (bloat); in Postgres **`VACUUM`** cleans them,
reclaiming space and updating statistics.

## Question 7
- **Read Committed / Repeatable Read** are implemented via an **MVCC snapshot** — reads
  without locks; RR takes a snapshot for the whole transaction.
- **Serializable** — either aggressive **locks**, or (Postgres) **SSI** over MVCC: it
  tracks dangerous intersections and **rolls back** one transaction with a serialization
  error (which must be retried).
So the isolation level is the policy, and locks/MVCC are the mechanisms implementing it.
