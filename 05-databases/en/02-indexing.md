# Indexing

> 🌐 Russian version: [02-indexing.md](../ru/02-indexing.md)

How a database finds rows fast and why an index isn't a free "make it faster". Reads
as a story.

The through-line of the topic:

> **An index is a separate sorted structure (usually a B-tree) that lets you find the
> needed rows without scanning the whole table. It speeds up reads at the cost of
> slower writes and extra space. The key — understand when an index is actually used
> and when the DB does a full scan anyway.**

---

## The problem: the full scan

Without an index, `WHERE email = 'x'` makes the DB read **every row** of the table —
that's **O(n)**, slow on millions of rows. An index turns the search into **O(log n)**:
like an alphabetical index in a book — you don't flip through it all, you jump via a
sorted reference to the right page.

---

## B-tree — the main index type

By default an index is a **balanced tree (B-tree)**: keys are stored **sorted**,
search/insert in logarithmic time. Why it's universal: it supports not only equality
but also **ranges and sorting** (`>`, `<`, `BETWEEN`, `ORDER BY`, the prefix `LIKE
'abc%'`) — because the data is already ordered.

A **hash index** — equality only (`=`), can't do ranges; faster on strict `=`, but used
less often.

---

## Composite indexes and the leftmost-prefix rule

An index on several columns `(a, b, c)` is sorted first by `a`, then `b`, then `c`.
Hence the **leftmost-prefix rule**: it works for conditions on `a`, `(a, b)`,
`(a, b, c)` — but **not** on `b` or `c` apart from `a`.
```
INDEX(country, city)
WHERE country='US'                 → the index works
WHERE country='US' AND city='NY'   → works
WHERE city='NY'                    → the index does NOT help (no leading country)
```
The column order in an index is an important design decision.

---

## Covering index

If the index contains **all** the columns a query needs, the DB answers **straight
from the index** without looking at the table — an **index-only scan**. Very fast.
```sql
INDEX(user_id, status)
SELECT status FROM orders WHERE user_id = 1;   -- everything is in the index
```

---

## The cost of indexes

An index isn't free:
- **slows writes** — every `INSERT/UPDATE/DELETE` must also update all indexes;
- **takes space** on disk/in memory;
- unused indexes nobody hits are pure downside.
So you index **for real queries**, not "just in case on every column".

---

## When an index is NOT used

A frequent question — why an index exists yet the scan is still full:
- **a function/expression on the column**: `WHERE LOWER(email)=...` — an index on
  `email` doesn't fit (you need a functional index);
- **low selectivity**: a column with 2 values (`gender`) — cheaper to just scan;
- **a leading wildcard**: `LIKE '%abc'` — you can't jump through the sorted tree;
- **a type mismatch**: comparing a string with a number → coercion breaks the index;
- the planner decided a scan is **cheaper** (a small table).

To see what actually happens — `EXPLAIN` (see
[query-optimization](./06-query-optimization.md)).

---

## Interview phrasing

> "An index is a sorted structure (usually a B-tree) turning search from an O(n) full
> scan into O(log n). A B-tree is good because it does equality, ranges, and sorting,
> since the data is ordered; a hash does only `=`. A composite index works by the
> leftmost-prefix rule — the condition must start with the leading column. A covering
> index answers straight from the index (index-only). The cost is slower writes and
> space, so I index for real queries. An index won't fire on a function over the
> column, low selectivity, a leading `%` in LIKE, or a type mismatch — I check with
> EXPLAIN."

---

See [02-indexing.tasks.md](./02-indexing.tasks.md) — tasks. Solutions in
[02-indexing.answers.md](./02-indexing.answers.md).
