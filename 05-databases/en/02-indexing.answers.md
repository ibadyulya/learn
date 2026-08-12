# Indexing — solutions

> 🌐 Russian version: [02-indexing.answers.md](../ru/02-indexing.answers.md)

---

## Question 1
An index is a sorted reference that lets you find rows **without scanning the whole
table**. It changes search complexity from **O(n)** (full scan) to **O(log n)** —
because the data in the index is **ordered**, and you can jump through the tree like an
alphabetical index rather than reading everything in a row.

## Question 2
A **B-tree** stores keys sorted → it does **equality, ranges, and sorting** (`>`, `<`,
`BETWEEN`, `ORDER BY`, the prefix `LIKE 'abc%'`). A **hash** stores by hash → only
strict **equality** (`=`), no ranges or sorting. A hash is slightly faster on `=`, but
a B-tree is more universal — hence the default.

## Question 3
**No**, for `WHERE city='NY'` the index `(country, city)` won't work. It's sorted
**first by country**, then city — like a phone book by surname, then first name.
Searching by one city without the country = flipping through the whole book. The
**leftmost-prefix rule**: a composite index helps only if the condition includes the
**leading** columns in order — `country`, `(country, city)`, but not `city` on its own.

## Question 4
A **covering index** contains all the columns a query needs (both in `WHERE` and
`SELECT`). Then the DB takes the answer **straight from the index** without touching the
table itself (**index-only scan**). Faster because it saves the **extra trip to the
table** for the row via the found pointer.

## Question 5
Costs: (1) **slower writes** — every INSERT/UPDATE/DELETE also updates all indexes;
(2) **space** on disk/in memory. "An index on every column" is bad because it bloats
writes and storage, while most of those indexes are **used by no one** — pure overhead
with no benefit. You index for **real** queries.

## Question 6
`LOWER(email)` is a **function on the column**: the index is built on `email` values,
but the compared thing is a function result — the DB can't match it against the tree and
does a scan. Fix: a **functional index** `INDEX(LOWER(email))`, or store email already
lowercased and search by that.

## Question 7
- **low selectivity** — a column with a couple of values (`gender`, `is_active`): a scan
  is cheaper than jumps;
- **a leading wildcard** — `LIKE '%abc'`: you can't start from a known prefix in the
  sorted tree;
- **a type mismatch** — comparing a string column with a number (or vice versa) needs
  coercion that breaks index use;
- (also) a small table — the planner itself decides a full scan is **cheaper**.
