# SQL vs NoSQL — solutions

> 🌐 Russian version: [03-sql-vs-nosql.answers.md](../ru/03-sql-vs-nosql.answers.md)

---

## Question 1
**Relational** — data in tables with a strict schema, linked by keys, joined with JOIN;
**non-relational** — documents/key-value pairs/columns/graphs with a flexible schema.
SQL's strength — **integrity and transactions** (constraints, ACID, complex queries);
its weakness — a **rigid schema** (migrations) and hard horizontal scaling (sharding).

## Question 2
No, it's a family. The families:
- **document** (Mongo) — a profile/order as a nested document;
- **key-value** (Redis) — cache, sessions;
- **column** (Cassandra) — huge volumes, analytics/streaming writes;
- **graph** (Neo4j) — social networks, recommendations (traversing relations).

## Question 3
- **Schema-on-write** (SQL): the structure is defined up front, the DB checks it **on
  write** → data is uniform, but changing the schema is costly (migrations).
- **Schema-on-read** (document NoSQL): you write anything, the app interprets the shape
  **on read** → flexible, evolves fast, but no guarantee of uniformity (responsibility
  on the app).

## Question 4
- **ACID** — Atomicity, Consistency, Isolation, Durability: strict transactional
  guarantees (typical of SQL).
- **BASE** — Basically Available, Soft state, Eventually consistent: availability and
  scale at the cost of instant consistency (many NoSQL).
The **CAP** link: under distribution you choose between strong consistency (ACID/CP) and
availability/scale (BASE/AP) — the same trade-off.

## Question 5
- **SQL** — orders/payments: related data, transactions and integrity needed (debit
  money and create an order atomically), complex JOIN reports.
- **Document NoSQL** — a product catalog/CMS: heterogeneous, evolving records, each
  self-contained as a document, horizontal read scaling needed.

## Question 6
**Polyglot persistence** — using different stores for different tasks in one product,
not "one DB for everything". Example: **PostgreSQL** for orders/users (transactions) +
**Redis** for cache and sessions + **Elasticsearch** for full-text search. Each for its
strength.

## Question 7
It's superficial because: (1) SQL **does scale** — via replicas (reads), sharding
(writes); it's just harder than in NoSQL; (2) Mongo doesn't "scale for free" — you pay
with weaker consistency and the loss of JOINs/transactions; (3) the choice is by the
**data model and consistency requirements**, not "scales/doesn't". For related
transactional data Mongo may be **worse**. The right answer is "it depends on the task".
