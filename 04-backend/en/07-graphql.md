# GraphQL

> 🌐 Russian version: [07-graphql.md](../ru/07-graphql.md)

An alternative to REST where the client shapes the response. Reads as a story; the
comparison with [REST](./03-rest-api-design.md) runs throughout.

The through-line of the topic:

> **GraphQL is a query language for an API: in one request to one endpoint the
> client specifies which fields and relations it needs and gets exactly those. This
> solves REST's over/under-fetching but moves the complexity to the server (N+1,
> caching, query-complexity limiting).**

---

## The REST problem GraphQL solves

- **Over-fetching** — an endpoint returns more than needed (you pull the whole `user`
  when you only need the name).
- **Under-fetching / N+1 requests** — to build a screen you call many endpoints
  (`/user`, then `/user/1/orders`, then per order...).

GraphQL: **one request** describes the needed shape, the server returns exactly it:
```graphql
query {
  user(id: 1) {
    name
    orders { id total }     # related data — in the same request
  }
}
```

---

## Schema, types, and resolvers

- **Schema** — a strongly typed API contract: types, fields, queries, mutations.
  ```graphql
  type User { id: ID!, name: String!, orders: [Order!]! }
  type Query { user(id: ID!): User }
  ```
- **Resolver** — a function that **fetches** the data for a specific field (from the
  DB, a service). Each field can have its own resolver; the engine assembles the
  response by the query's shape.

---

## Three operation types

- **Query** — reading (like GET).
- **Mutation** — changing (create/update/delete).
- **Subscription** — subscribing to real-time events (usually over WebSocket) — the
  observer pattern.

Everything goes through **one endpoint** (`POST /graphql`), unlike REST's many URLs.

---

## The N+1 problem and DataLoader

The main trap. A query `users { orders }` for 100 users would naively do **1 query
for the list + 100 queries for each one's orders** — N+1.
The solution — **DataLoader**: it collects ids within one tick and does **one batch
query** (`WHERE user_id IN (...)`), plus caches within the request. A must-have in
production.

---

## REST vs GraphQL — trade-offs

| | REST | GraphQL |
|---|---|---|
| Response shape | server fixes it | client chooses |
| Endpoints | many (resources) | one |
| Over/under-fetching | happens | solved |
| HTTP cache | simple (by URL) | **harder** (one POST URL) |
| Query complexity | bounded | client can request heavy → need a limit |
| Learning curve | low | higher (schema, resolvers, N+1) |

GraphQL is good with **many clients with different needs** (mobile/web), rich
relations. REST is simpler, caches great, is more predictable under load.

---

## Pitfalls

- **Caching** — no URL-per-resource, HTTP cache/CDN work worse; cache on the client
  (Apollo) or per field.
- **Query complexity/depth** — a client can request recursively heavy things → you
  need **depth/cost limits**, or it's a DoS.
- **Authorization** — at the field level, not just the route; harder.
- **N+1** — without DataLoader it's easy to drown the DB.

---

## Interview phrasing

> "GraphQL is a typed query language: in one request to one endpoint the client
> specifies the needed fields and relations, solving REST's over- and under-fetching.
> The server describes a schema (types, query/mutation/subscription) and resolvers —
> field-level data-fetching functions. The main trap is N+1 with related data, fixed
> by DataLoader (batching + per-request cache). The trade-off vs REST: client
> flexibility at the cost of harder caching (no URL-per-resource), the risk of heavy
> queries (need depth/cost limits), and per-field authorization. I pick GraphQL with
> many clients and rich relations, REST where simplicity and HTTP caching matter."

---

See [07-graphql.tasks.md](./07-graphql.tasks.md) — tasks. Solutions in
[07-graphql.answers.md](./07-graphql.answers.md).
