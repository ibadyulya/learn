# MongoDB and Redis — solutions

> 🌐 Russian version: [08-mongodb-redis.answers.md](../ru/08-mongodb-redis.answers.md)

---

## Question 1
MongoDB is a **document** database: data is stored as **documents** (BSON/JSON) in
**collections**. Unlike a flat SQL row with a strict schema, a document is **flexible**
and **nested** — it can contain objects and arrays, and documents in one collection can
differ in their set of fields (schema-on-read).

## Question 2
- **Embedding** — nest related data right in the document (items inside the order). ➕
  read in one query, updated atomically; for "read together, bounded in size".
- **Referencing** — store the related entity's `id` (like a foreign key). ➕ against
  duplication, for many-to-many, large/growing relations.
The choice — by **how you read**: together → embed, independently/reusable → reference.

## Question 3
The **aggregation pipeline** — a sequence of stages where one's output feeds the next's
input: `$match` (filter) → `$group` (grouping/aggregates) → `$sort`, etc. Comparable to
**SQL `GROUP BY` + `WHERE` + `ORDER BY`**, but expressed step by step (a pipeline), like
a stream of transformations.

## Question 4
Redis keeps data **in RAM** — no disk on the read/write path, hence microsecond latency.
The main downside: **everything must fit in RAM** (memory is expensive and limited), and
on a crash without persistence data is lost. Memory is the bottleneck and a cost item.

## Question 5
No, Redis can do **persistence**:
- **RDB** — periodic **snapshots** of the whole dataset to disk: fast restart, but between
  snapshots you can lose the last seconds;
- **AOF** — a **log** of all write operations: more reliable (minimal loss), but a bigger
  file and slower.
You can enable both. But the working set must still fit in memory.

## Question 6
- **PostgreSQL** — orders/payments: relations, transactions, integrity, complex reports
  (JOINs).
- **MongoDB** — catalog/CMS/profiles: self-contained documents, a flexible schema, read
  scale.
- **Redis** — cache, sessions, counters/rate limiting, queues: where speed and
  ephemerality matter.
Often together (polyglot persistence).

## Question 7
Good: commands run **one at a time**, so simple operations are **atomic without locks**
(no races, `INCR` is safe). Bad: **one heavy command blocks** the whole server (everyone
else waits), and you can't spread load across cores with one instance — you scale with
replicas/shards and avoid expensive commands.
