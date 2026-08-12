# SQL vs NoSQL

> 🌐 Russian version: [03-sql-vs-nosql.md](../ru/03-sql-vs-nosql.md)

How to choose a database type for the task. Reads as a story; the takeaway is not
"trendy/not trendy" but a trade-off.

The through-line of the topic:

> **SQL — relational databases with a strict schema, relations, and ACID guarantees.
> NoSQL — a family of non-relational ones (document, key-value, column, graph)
> sacrificing some strictness for flexibility and horizontal scale. The choice — by
> the data model and the requirements for consistency/scale, not by hype.**

---

## SQL — relational databases

Data lives in **tables** with a **strict schema** (columns, types). Rows are linked via
keys, queries join them via **JOIN**. The language is SQL. They give **ACID** (see
[isolation-levels](./01-isolation-levels.md)) and strong consistency.
- ➕ integrity (constraints, foreign keys), powerful JOIN queries, transactions,
  predictability;
- ➖ a rigid schema (migrations), harder horizontal scaling (sharding is a pain, see
  [scaling-data](../../06-system-design/en/06-scaling-data.md)).

Examples: PostgreSQL, MySQL.

---

## NoSQL — a family of non-relational ones

Not one type but **several different ones**, united by "non-relational":
- **Document** (MongoDB) — JSON-like documents, a flexible/nested structure. Good when
  data naturally fits into a document (a profile, a whole order).
- **Key-value** (Redis, DynamoDB) — just "key → value", super fast but primitive
  queries. Cache, sessions.
- **Column** (Cassandra, ClickHouse) — storage by columns, huge volumes,
  analytics/streaming writes.
- **Graph** (Neo4j) — nodes and relations, when the **relations** matter (social
  networks, recommendations).

Common: a **flexible schema** (schema-on-read) and a focus on **horizontal scale**.

---

## Schema-on-write vs schema-on-read

- **SQL — schema-on-write:** you define the structure up front, the DB checks it on
  write. Data is always consistent in shape, but changing the schema is costly
  (migrations).
- **NoSQL — schema-on-read:** you write anything, the application interprets the shape
  on read. Flexible and evolves fast, but no guarantee of uniformity — the
  responsibility is on the app.

---

## ACID vs BASE

- **ACID** (SQL) — atomicity, consistency, isolation, durability: strict guarantees.
- **BASE** (many NoSQL) — Basically Available, Soft state, Eventually consistent: they
  sacrifice instant consistency for availability and scale (see
  [CAP in foundations](../../06-system-design/en/01-foundations.md)).

It's the same CAP trade-off: strong consistency vs availability/scale under
distribution.

> Caveat: the line has blurred. Postgres can do JSONB (documents), MongoDB added
> multi-document transactions. Look at the **specific guarantees**, not the label.

---

## How to choose

- **SQL** — related data with integrity and transactions: finance, orders, accounting;
  when complex JOIN queries and strong consistency matter.
- **NoSQL document** — a flexible/evolving schema, data fits into a document, horizontal
  scale needed (catalogs, profiles, content).
- **Key-value** — cache, sessions, counters (speed over queries).
- **Graph** — when the core of the task is traversing relations.

**Polyglot persistence** is the norm: in one product Postgres for orders + Redis for
cache + Elastic for search. You pick per task, not "one DB for everything".

---

## Interview phrasing

> "SQL — relational databases with a strict schema, JOINs, and ACID: integrity and
> complex queries, but a rigid schema and hard horizontal scaling. NoSQL — a family
> (document, key-value, column, graph) with a flexible schema (schema-on-read) and a
> focus on scale, often BASE instead of ACID — the same CAP trade-off of consistency vs
> availability. I choose by the data model and requirements: related transactional data
> → SQL; a flexible document and scale → Mongo; cache/sessions → key-value; relations →
> a graph. The line has blurred (JSONB, transactions in Mongo), so I look at the
> specific guarantees, and often use several DBs (polyglot persistence)."

---

See [03-sql-vs-nosql.tasks.md](./03-sql-vs-nosql.tasks.md) — tasks. Solutions in
[03-sql-vs-nosql.answers.md](./03-sql-vs-nosql.answers.md).
