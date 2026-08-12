# GraphQL — solutions

> 🌐 Russian version: [07-graphql.answers.md](../ru/07-graphql.answers.md)

---

## Question 1
- **Over-fetching** — an endpoint returns **more** data than the client needs (you get
  the whole object for one field) → wasted traffic.
- **Under-fetching** — one endpoint isn't **enough**, you make many requests to build a
  screen (N+1 network calls).
GraphQL solves both: in one request the client asks for exactly the needed fields and
relations.

## Question 2
- **Schema** — a typed API contract: types, fields, available query/mutation.
- **Resolver** — a function that **fetches data for a specific field** (from the DB, a
  service). Parsing the query, the engine calls the resolvers of the needed fields and
  assembles the response in the requested shape.

## Question 3
**Query** (reading), **Mutation** (changing), **Subscription** (real-time events,
usually over WebSocket). Everything goes through **one endpoint** (`POST /graphql`),
unlike REST's many URLs.

## Question 4
**N+1:** a query for related data (`users { orders }`) naively does 1 query for the
list + one query for **each** user's orders. **DataLoader** collects all requested ids
within one event-loop tick and does **one batch query** (`WHERE id IN (...)`), plus
caches results within the request — one query instead of N.

## Question 5
1. **Caching** is harder — no URL-per-resource, HTTP cache/CDN barely apply.
2. **Query complexity/depth** — the client can request heavy/recursive things → you
   need depth and cost limits (or it's a DoS).
3. **Authorization** at the field level, not just the route — harder.
4. A higher **entry barrier** (schema, resolvers, fighting N+1).

## Question 6
In REST a response is tied to a **resource URL** → HTTP cache and CDN cache by URL out
of the box. In GraphQL everything goes as **one POST to one URL** with a query body,
responses differ — you can't cache by URL. You have to cache on the client (Apollo's
normalized cache) or per field/persisted queries — harder.

## Question 7
- **GraphQL** — when there are **many different clients** (mobile, web, partners) with
  different needs and **rich relations**: each takes its own shape in one request.
- **REST** — when **simplicity, predictable load, and HTTP cache/CDN** matter (a public
  API, static resources), and there are few clients with similar requests.
