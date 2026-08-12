# Normalization — solutions

> 🌐 Russian version: [07-normalization.answers.md](../ru/07-normalization.answers.md)

---

## Question 1
Normalization removes **data duplication** — so each fact is stored in **one place**.
Anomalies from duplicates:
- **update** — a fact (a customer's email) in many rows; you edit not everywhere → a
  contradiction;
- **insert** — you can't add an entity without a related record (a customer without an
  order);
- **delete** — deleting one record loses data that existed only in it.

## Question 2
**1NF** requires **atomicity**: one indivisible value per cell, no repeating
groups/lists. Violation: `phones = "111, 222"` in one cell. Fix — move phones to
**separate rows/a table** (`user_id, phone`), one value per row.

## Question 3
**3NF**: every non-key attribute depends **only on the primary key**, not on another
non-key. A **transitive dependency** — when key → attribute A → attribute B (B depends on
A, not directly on the key). Example: storing `zip` and `city` in an order, where `zip →
city`; the city depends on the zip, not the order id → move it to an address lookup.

## Question 4
- **"the key"** — 1NF: there's a primary key, values are atomic.
- **"the whole key"** — 2NF: non-key attributes depend on the **whole** composite key,
  not part (no partial dependency).
- **"nothing but the key"** — 3NF: they depend **only** on the key, not on each other (no
  transitive dependencies).

## Question 5
**Denormalization** — deliberately **duplicate** data (store the customer's name right in
the order, keep precomputed aggregates) to read **without expensive JOINs** — faster. The
trade-off: **read speed at the cost of integrity** — duplicates must be synchronized,
anomaly risks return.

## Question 6
Denormalization is justified under **read-heavy** load, reports/analytics, when a JOIN
became a **proven** bottleneck (by `EXPLAIN`). After it, your responsibility is to **keep
duplicates in sync** on every change of the source (via the app, triggers, periodic
recomputation), or you'll get drift and update anomalies.

## Question 7
The customer's name and email are duplicated in every order row → anomalies: **update**
(an email change — edit in all orders), **delete** (deleting the last order — lost the
contact), **insert** (can't add a customer without an order). Normalization: move the
customer to a `customers (id, name, email)` table, and keep only **`customer_id`** (a
foreign key) in `orders`. Now the customer's email is in one place.
