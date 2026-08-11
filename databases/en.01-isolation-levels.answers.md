# Isolation levels — solutions

> 🌐 Russian version: [ru.01-isolation-levels.answers.md](./ru.01-isolation-levels.answers.md)

---

## Question 1
**Non-repeatable read.** T1 reads **the same row** (id=7) twice and gets
different values (100 → 150), because T2 did an `UPDATE` between the reads and
**committed**. The data is real (not a draft) — but the picture inside T1 is
inconsistent. Disappears at **Repeatable Read** and above.

## Question 2
**Dirty read.** T2 read `qty=0`, which T1 hadn't yet **committed**, and then T1
did a `ROLLBACK` — the value 0 never existed. The minimum level where it
disappears — **Read Committed** (only committed data is visible).

## Question 3
The difference is in the **status of the other transaction's data**:
- **dirty read** — you read **uncommitted** data (a draft) that may roll back →
  you work with something that may not exist at all;
- **non-repeatable read** — you read **committed** data (it's real), but the
  row's value **changed between your two reads** → inconsistency within one
  transaction.

In short: dirty — "someone else's unfinished"; non-repeatable — "someone else's
finished, but already different".

## Question 4
- **non-repeatable** — the **value of an existing row** changes (someone else's
  `UPDATE`): re-read the same row — a different value;
- **phantom** — the **set of rows** matching the condition changes (someone
  else's `INSERT` or `DELETE`): re-run the same query with `WHERE` — a different
  number of rows.

Non-repeatable is about an `UPDATE` of one row, phantom about rows
appearing/disappearing.

## Question 5
- (a) "don't read uncommitted" → **Read Committed** (removes dirty read);
- (b) "read a row twice and get the same" → **Repeatable Read** (removes
  non-repeatable read).

## Question 6
**PostgreSQL by default — Read Committed. MySQL (InnoDB) by default — Repeatable
Read.** About phantoms: by the SQL standard they're possible at Repeatable Read,
**but InnoDB at its default RR prevents them in practice** (an MVCC snapshot + gap
locks). So formally "yes, the standard allows it", in fact "MySQL at its default
mostly doesn't produce phantoms". It's precisely this "standard vs
implementation" distinction that's being tested.

## Question 7
Serializable must guarantee a result "as if the transactions ran in turns". This
costs parallelism: the DBMS either aggressively **locks** (transactions wait for
each other), or (PostgreSQL — **SSI**) tracks conflicts and, at the first sign of
danger, **rolls back** one transaction with a serialization error
(`serialization failure`).

So an application on Serializable **must be able to catch this error and retry**
the transaction — at Read Committed there's no error class like this, no retries
needed. (A link to the retries + backoff topic from system-design.)
