# Query optimization

> 🌐 Russian version: [06-query-optimization.md](../ru/06-query-optimization.md)

Why a query is slow and how to fix it systematically, not by guessing. Reads as a
story.

The through-line of the topic:

> **SQL is declarative — you say "what" to get, and how to execute it is decided by
> the **planner**. A slow query is almost always a bad plan. The tool — `EXPLAIN`: it
> shows the plan; you read it, find the bottleneck (full scan, expensive JOIN, N+1)
> and fix it with an index or a rewrite.**

---

## The planner: from "what" to "how"

You write `SELECT ... WHERE ...`, and the DBMS builds an **execution plan**: which index
to use, in what order to join tables, how to sort. The choice is **cost-based**: the
planner estimates options by **statistics** (how many rows, what distribution) and picks
the cheapest. So: stale statistics → a bad plan.

---

## EXPLAIN — see the plan

`EXPLAIN` shows the **estimated** plan; `EXPLAIN ANALYZE` also **actually** runs it,
giving real time and row counts.
```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE user_id = 42;
```
What to look for:
- **Seq Scan** (full scan) of a big table where you expect an Index Scan → no/unused
  index;
- a **large discrepancy** between estimate and actual row counts → stale statistics
  (`ANALYZE`);
- expensive **sorts**, **nested loops** on large volumes;
- the bottleneck — the "fattest" plan node by time/rows.

---

## Join strategies (briefly)

How the DB does a JOIN:
- **Nested loop** — for each row of one table, look in the other; good when one table is
  small and there's an index.
- **Hash join** — build a hash table on one side; good for large sets without sorting.
- **Merge join** — merge two sorted sides; good when the data is already ordered (an
  index).
The planner chooses itself; your job is to give it indexes and fresh statistics.

---

## Typical causes of slow queries and fixes

- **No index** for `WHERE`/`JOIN`/`ORDER BY` → add one (see [indexing](./02-indexing.md));
  check it's actually used.
- **N+1 queries** (from the app/ORM) — instead of one JOIN you hit the DB in a loop (see
  [GraphQL/DataLoader](../../04-backend/en/07-graphql.md)) → batch/join.
- **`SELECT *`** — pulling extra columns, prevents index-only; select what's needed.
- **A function on a column** in `WHERE` — breaks the index (`LOWER(email)`); a functional
  index or different storage.
- **A huge OFFSET** in pagination — switch to cursor-based (see
  [REST](../../04-backend/en/03-rest-api-design.md)).
- **Stale statistics** → `ANALYZE`.

---

## The order of actions

1. Found a slow query (slow-query logs, APM).
2. `EXPLAIN ANALYZE` — where the bottleneck is.
3. A hypothesis: an index? a rewrite? denormalize?
4. Changed → `EXPLAIN ANALYZE` again, compared.
Not "add indexes at random", but **measure → change → recheck**.

---

## Interview phrasing

> "SQL is declarative, the plan is built by a cost-based planner from statistics, so a
> slow query is usually a bad plan. I diagnose with `EXPLAIN ANALYZE`: I look for a Seq
> Scan of a big table, a discrepancy between estimate and actual, expensive sorts/joins.
> Common causes — no suitable index, N+1 from the app, `SELECT *`, a function on a
> column, a huge OFFSET, stale statistics. I fix precisely: an index for the query,
> batching instead of N+1, cursor pagination, `ANALYZE`. I work in a measure → change →
> recheck loop, not adding indexes at random."

---

See [06-query-optimization.tasks.md](./06-query-optimization.tasks.md) — tasks.
Solutions in [06-query-optimization.answers.md](./06-query-optimization.answers.md).
