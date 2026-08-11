# Transaction isolation levels

The through-line of the topic:

> **A transaction wants to pretend it's alone in the database. But other
> transactions run in parallel and "leak" into it. The isolation level is the
> knob "how much of other people's leakage am I willing to tolerate in exchange
> for speed". Higher isolation → less leakage, but less parallelism.**

Self-check tasks — [01-isolation-levels.tasks.en.md](./01-isolation-levels.tasks.en.md),
solutions — [01-isolation-levels.answers.en.md](./01-isolation-levels.answers.en.md).

> 🌐 Russian version: [01-isolation-levels.ru.md](./01-isolation-levels.ru.md)

---

## Isolation — the "I" in ACID

ACID — a transaction's guarantees:
- **A — Atomicity:** all or nothing; failed halfway — full rollback.
- **C — Consistency:** a transaction moves the DB from one correct state to
  another, without violating constraints/foreign keys.
- **I — Isolation:** ← **our topic.** Parallel transactions don't interfere with
  each other; ideally each runs as if it's alone.
- **D — Durability:** committed — survives a crash (written to disk).

Isolation is the only letter about **what happens when there are many
transactions at once.** The rest are about a single one.

## The ideal — "as if in turns"

The gold standard of correctness — **serializability**: the result of parallel
execution = as if the transactions ran **strictly in turns**, one after another.
Then parallelism is "invisible", each lives in the illusion of solitude.

The problem: **it's expensive** — lots of locks or rollbacks, parallelism drops.
Hence the compromise: allow a bit of leakage for speed → a gradation of levels.
But to understand what exactly you're allowing, you need to know the leaks
themselves — the **anomalies**.

---

## Three anomalies — ways to "leak"

The core of the topic: levels are defined by **which anomalies they forbid.**
Three of them per the classic SQL standard, in increasing "scale of leakage".

### 1. Dirty read — read someone else's UNcommitted data
You read data another transaction changed but **hasn't committed yet**; and it
**rolls back** — so you worked with something that **never existed**.
```
T1: UPDATE balance SET amount=500 WHERE id=1;   -- NOT committed yet
T2:   SELECT amount FROM balance WHERE id=1;      -- reads 500 (dirty!)
T1: ROLLBACK;                                      -- reverted to 100
-- T2 decided based on 500, which didn't exist
```

### 2. Non-repeatable read — read a row twice, values differ
You read one **row** twice, getting **different** values, because another
transaction changed it between your reads and **committed**.
```
T1: SELECT amount FROM balance WHERE id=1;        -- 100
T2:   UPDATE balance SET amount=500 WHERE id=1; COMMIT;
T1: SELECT amount FROM balance WHERE id=1;        -- 500 (the same row!)
```
Difference from dirty: here T2 **committed** (the data is real). The problem —
one transaction sees an **inconsistent picture**: the ground shifted under its
feet.

### 3. Phantom read — the same query returned a DIFFERENT SET of rows
You run a query **with a condition** twice, and the second time there are
more/fewer rows, because another transaction **inserted/deleted** matching ones
and committed.
```
T1: SELECT * FROM orders WHERE amount > 100;      -- 3 rows
T2:   INSERT INTO orders (amount) VALUES (200); COMMIT;
T1: SELECT * FROM orders WHERE amount > 100;      -- 4 rows (phantom)
```
Difference from non-repeatable: there the **row's value** changed (`UPDATE`),
here — the **set of rows** (`INSERT`/`DELETE`).

| Anomaly | What leaked | Someone else's operation |
|---|---|---|
| **Dirty read** | an uncommitted value | an unfinished `UPDATE` |
| **Non-repeatable read** | a committed change to a **row** | a committed `UPDATE` |
| **Phantom read** | a committed change to a **set of rows** | a committed `INSERT`/`DELETE` |

The order "from dirtiest to subtlest" — the levels cut them off **in exactly this
order, bottom to top.**

---

## Four levels — each cuts off one more anomaly

### 1. Read Uncommitted — "I read everything, even drafts"
You see even other transactions' uncommitted changes. Protects against
**nothing**. Practically unused. (Postgres doesn't have it — it gives Read
Committed.)

### 2. Read Committed — "I read only committed data"
You see only committed data. ✅ removes **dirty read**; ❌ non-repeatable and
phantom remain. The reason: the guarantee is at the level of an **individual
query** — each `SELECT` sees a fresh snapshot as of its moment, between them the
world changes.
> **The default in PostgreSQL** (and Oracle, SQL Server).

### 3. Repeatable Read — "rows don't change under my feet the whole transaction"
The guarantee is raised from the query to the **whole transaction** — usually via
a **snapshot** (a "photograph" of the DB at the start). ✅ removes **dirty +
non-repeatable**; ❌ by the standard **phantom remains** (the rows you read are
frozen, but not new ones).
> **A gotcha for the interview.** **The default in MySQL (InnoDB)**, and its
> implementation (MVCC + gap locks) **in practice doesn't produce phantoms** —
> stricter than the standard. In **PostgreSQL** this level = snapshot isolation,
> it also cuts off ordinary phantoms. Say it like this: "by the SQL standard RR
> allows phantoms, but MySQL InnoDB and Postgres in practice mostly prevent
> them". Interviewers love the "standard vs real DB" distinction.

### 4. Serializable — "as if in turns"
The result = as under strictly sequential execution. ✅ removes **all three** and
any parallelism effects. The highest price: aggressive locks, or (Postgres —
**SSI, Serializable Snapshot Isolation**) conflict tracking and a **rollback** of
one transaction with a serialization error → it must be **retried**.

## Summary table

| Level | Dirty | Non-repeatable | Phantom |
|---|:---:|:---:|:---:|
| **Read Uncommitted** | ❌ | ❌ | ❌ |
| **Read Committed** | ✅ | ❌ | ❌ |
| **Repeatable Read** | ✅ | ✅ | ❌¹ |
| **Serializable** | ✅ | ✅ | ✅ |

✅ = anomaly forbidden. ¹ possible by the standard; in MySQL InnoDB and Postgres
in practice usually not. **The staircase:** each level forbids one more anomaly,
in the order dirty → non-repeatable → phantom.

## The trade-off
```
weaker (Read Uncommitted) ←—————————→ stricter (Serializable)
more parallelism, faster          less leakage, more correct
fewer locks/rollbacks             more locks / retries
```
Choose by task (like CAP): analytics/dashboards → **Read Committed** is enough;
finance/transfers → **Serializable** (prepare retries for a serialization
failure).

> Phrasing: "The isolation level is a compromise between correctness and
> parallelism. Three anomalies in increasing order: dirty read (someone else's
> uncommitted data), non-repeatable (a row changed between reads), phantom (the
> set of rows changed). Read Uncommitted protects against nothing, Read Committed
> removes dirty (Postgres default), Repeatable Read — also non-repeatable (MySQL
> default), Serializable — everything, as if in turns. By the standard RR allows
> phantoms, but MySQL and Postgres prevent them in practice. The stricter — the
> less parallelism and the more locks or rollbacks with retries."
