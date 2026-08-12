# MongoDB and Redis

> 🌐 Russian version: [08-mongodb-redis.md](../ru/08-mongodb-redis.md)

The two most common NoSQL stores and when to use each. Reads as a story; a
continuation of [SQL vs NoSQL](./03-sql-vs-nosql.md).

The through-line of the topic:

> **MongoDB — a document database: flexible JSON-like documents, rich queries and
> aggregation; used when data fits into a document and the schema evolves. Redis — an
> in-memory key-value store: super fast, for cache/sessions/counters/queues. These are
> tools for different tasks, not a "replacement for SQL".**

---

## MongoDB — a document database

Data is **documents** (BSON, essentially JSON) in **collections**. The schema is
flexible: documents in one collection can differ in fields.
```js
db.orders.insertOne({
  _id: 1,
  customer: { name: "Ann", email: "ann@x.com" },   // a nested object
  items: [{ product: "Book", qty: 2 }],            // an array
  total: 20
});
db.orders.find({ "customer.name": "Ann", total: { $gt: 10 } });
```
There are **indexes** (including on nested fields and arrays), an **aggregation
pipeline** (`$match → $group → $sort` — like SQL GROUP BY, but step by step), and since
recent versions — **multi-document transactions**.

**Embedding vs referencing** — the key design decision:
- **embedding** (nesting items right in the order) — read in one query, updated
  atomically; good for "read together, bounded in size";
- **referencing** (storing `customer_id`, as in SQL) — against duplication and for
  many-to-many/large/growing collections.
The Mongo rule: model **for the queries** ("what we read together"), not for entities.

Good for: a flexible/evolving schema, self-contained documents (a profile, an order),
horizontal scale (sharding is built in). Weak where there are many complex relations and
JOINs — that's SQL territory.

---

## Redis — an in-memory key-value store

Data lives **in RAM** → microsecond latency. Not just "string → string", but rich
structures:
- **types:** string, hash, list, set, sorted set (zset), + a TTL per key;
- **uses:** cache, sessions, counters/rate limiting, queues, pub/sub, leaderboards
  (zset), distributed locks (in detail — in
  [caching-redis](../../04-backend/en/06-caching-redis.md)).

**Persistence** (Redis isn't only "a cache you don't mind losing"):
- **RDB** — periodic snapshots to disk (fast restart, but you can lose the last
  seconds);
- **AOF** — a log of all operations (more reliable, slower/bigger).
You can combine them. But the working set must still **fit in memory**.

Redis is **single-threaded per command** → simple operations are atomic without locks,
but a heavy command can't be spread across cores.

---

## When to use what

- **PostgreSQL/SQL** — related transactional data, integrity, complex queries.
- **MongoDB** — document-oriented data, a flexible schema, self-contained records, read
  scale.
- **Redis** — speed: cache, sessions, ephemeral state, counters, queues.

Often **together** (polyglot persistence): Postgres as the source of truth + Redis as a
cache + Mongo/Elastic for their model.

---

## Interview phrasing

> "MongoDB — a document database: flexible JSON documents in collections, indexes on
> nested fields, an aggregation pipeline, multi-document transactions. The key decision
> is embedding (read in one query, atomic) vs referencing (against duplication, for
> relations); I model for the queries. Redis — in-memory key-value with rich structures
> (hash/list/set/zset) and TTL, for cache, sessions, rate limiting, queues, locks;
> persistence via RDB snapshots or an AOF log, but data must fit in memory, and it's
> single-threaded per command. I use them not 'instead of SQL' but for their task —
> usually together (polyglot persistence)."

---

See [08-mongodb-redis.tasks.md](./08-mongodb-redis.tasks.md) — tasks. Solutions in
[08-mongodb-redis.answers.md](./08-mongodb-redis.answers.md).
