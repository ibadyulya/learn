# Query optimization — solutions

> 🌐 Russian version: [06-query-optimization.answers.md](../ru/06-query-optimization.answers.md)

---

## Question 1
The planner turns **declarative** SQL into an **execution plan**: it picks indexes, the
order and method of joins, sorts. The choice is **cost-based**, estimated from table
**statistics** (row counts, value distribution). Stale statistics give wrong estimates →
the planner picks a **bad plan** (e.g., a nested loop where a hash join is needed).
`ANALYZE` fixes it.

## Question 2
- **`EXPLAIN`** — shows the **estimated** plan and estimates **without running** the
  query.
- **`EXPLAIN ANALYZE`** — **actually runs** it and shows the **real** time and row counts
  (comparable to the estimate).
A **`Seq Scan`** of a big table = the DB reads it **whole** (no/unused index) — a common
source of slowness.

## Question 3
1. A **Seq Scan** of a big table where an Index Scan was expected.
2. A **discrepancy** between estimated and actual rows (stale statistics).
3. Expensive **sorts/joins** (a nested loop on large volumes, a big Sort); the "fattest"
   node by time.

## Question 4
**N+1** — the app does 1 query for a list + one query per **each** element (e.g., for
related data in a loop), instead of one JOIN/batch. The result — hundreds of queries
instead of one. Remove it: a **JOIN**/eager loading (`include`/`join` in the ORM) or
batching (**DataLoader**: collect ids and one `WHERE id IN (...)`).

## Question 5
`SELECT *` pulls **all** columns, even unneeded ones: extra I/O and traffic, and it
**prevents index-only** (a covering index — the index lacks all columns, so it must go to
the table). Better select **only the needed** fields — less data and a chance at an
index-only scan.

## Question 6
The order: (1) **find** the slow query (logs/APM); (2) `EXPLAIN ANALYZE` — **where** the
bottleneck is; (3) a hypothesis (index/rewrite/denormalize); (4) change and **measure
again**, compare. "Indexes at random" is bad: extra indexes **slow writes** and take
space, and the problem may not be the index (N+1, statistics, `SELECT *`) — without
measuring you fix blindly.

## Question 7
A large **OFFSET** makes the DB **read and discard** all 100000 rows before the needed 20
— the further the page, the slower (grows linearly). The fix — **cursor-based
pagination**: `WHERE created_at < :lastSeen ORDER BY created_at DESC LIMIT 20` — jump
through the index from the cursor, without paging through everything up to it.
