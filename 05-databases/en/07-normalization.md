# Normalization

> 🌐 Russian version: [07-normalization.md](../ru/07-normalization.md)

How to lay data out across tables so as not to duplicate it or hit anomalies. Reads
as a story.

The through-line of the topic:

> **Normalization is organizing data so that each fact is stored in exactly one
> place. Duplication leads to update anomalies (one fact in many places goes out of
> sync). The normal forms (1NF→3NF) are steps toward that. Denormalization is
> deliberately breaking normalization for read speed.**

---

## The problem: duplication and anomalies

Dump everything into one table — data is **duplicated**, and that breeds anomalies:
```
orders: id | customer_name | customer_email | product | price
        1  | Ann           | ann@x.com      | Book    | 10
        2  | Ann           | ann@x.com      | Pen     | 2
```
- **Update anomaly** — Ann changed her email: you must edit it in **all** rows; miss one
  → a contradiction (two different emails for one person).
- **Insert anomaly** — you can't add a customer without an order (no row).
- **Delete anomaly** — you deleted Ann's last order → you also lost her contact.

The root — **one fact (Ann's email) is stored many times**. Normalization removes this.

---

## Normal forms (increasing)

**1NF — atomicity.** Values are **indivisible**, no repeating groups or arrays in a cell.
Not `phones = "111, 222"`, but separate rows/a phones table. Each cell — one value.

**2NF — no partial dependency.** Relevant with a **composite key**: every non-key
attribute depends on the **whole** key, not part of it. If in `(order_id, product_id)`
the field `product_name` depends only on `product_id` — move it to a products table.

**3NF — no transitive dependency.** A non-key attribute depends **only on the key**, not
on another non-key. If `zip → city` (the city is determined by the zip, not the order id)
— move it to a separate table.

Mnemonic: **"every non-key attribute depends on the key, the whole key, and nothing but
the key".** (1NF — the key; 2NF — the whole key; 3NF — nothing but the key.)

The result — data laid out across tables, linked by foreign keys, each fact in one place.

---

## Denormalization — a deliberate trade-off

Normalization is optimal for **writes and integrity**, but reading requires **JOINs**,
which are expensive on large volumes/analytics. **Denormalization** is deliberately
duplicating data (storing `customer_name` right in the order, keeping precomputed totals)
to read **without JOINs**, faster.

The price is that very anomaly risk: the duplicate must be **synchronized** on change
(triggers, the app, recomputation). The trade-off: **normalization — integrity and
writes; denormalization — read speed.**

When you denormalize: read-heavy load, reports/analytics, cache tables, when a JOIN became
a bottleneck (confirmed by `EXPLAIN`, see [query-optimization](./06-query-optimization.md)).

---

## Interview phrasing

> "Normalization is laying data out so each fact is in one place, otherwise duplication
> gives update/insert/delete anomalies. 1NF — atomic values with no repeating groups;
> 2NF — non-key attributes depend on the whole composite key, not part; 3NF — depend only
> on the key, no transitive dependencies. Short: 'the key, the whole key, and nothing but
> the key'. Denormalization — deliberately duplicate for read speed (remove expensive
> JOINs), at the cost of needing to sync duplicates. I normalize by default and
> denormalize selectively for a proven read bottleneck."

---

See [07-normalization.tasks.md](./07-normalization.tasks.md) — tasks. Solutions in
[07-normalization.answers.md](./07-normalization.answers.md).
