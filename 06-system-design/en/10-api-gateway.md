# API gateway

> 🌐 Russian version: [10-api-gateway.md](../ru/10-api-gateway.md)

A single entry point in front of microservices. Reads as a story; adjacent to
[microservices](./08-microservices-vs-monolith.md).

The through-line of the topic:

> **An API gateway — one entrance in front of many services: it routes requests, takes
> on cross-cutting concerns (authentication, limits, TLS), and optionally aggregates
> responses. The client doesn't know the internal topology, and services don't
> duplicate the shared plumbing.**

---

## The problem without a gateway

If a client (mobile, web) calls **each microservice directly**:
- it **knows the whole topology** (addresses, ports, the split) — tight coupling, any
  change to the internal structure breaks the client;
- **cross-cutting concerns are duplicated** in every service (authentication, rate
  limiting, CORS, logging);
- one screen = **many round trips** to different services.

You need an intermediary (the Indirection pattern from
[GRASP](../../02-paradigms-and-design/en/04-grasp.md)) — a gateway.

---

## What a gateway does

A single door in front of the services takes on:
- **routing** — `/orders/*` → the orders service, `/users/*` → the users service; the
  client sees one address;
- **authentication/authorization** — verify the token once at the entry, then pass the
  verified identity to services (see [auth](../../04-backend/en/04-auth.md));
- **rate limiting / throttling** — protection from overload (see
  [reliability-patterns](./03-reliability-patterns.md));
- **TLS termination** — decrypt HTTPS at the boundary;
- **aggregation** — assemble one response from several services (fewer client round
  trips);
- request/response transformation, caching, logging/metrics.

The point — **remove the cross-cutting plumbing from services** (high cohesion: a service
minds its own business) and **hide the internal topology** from the client.

---

## BFF — Backend for Frontend

Different clients need a different format/set of data (mobile — more compact, web —
fuller). A **BFF** — a separate gateway **per client type**, aggregating and shaping the
response just for it. Instead of one universal gateway — a BFF per mobile/web/partners.

---

## Gateway vs service mesh

A frequent question — don't confuse them:
- **API gateway** — **north-south** traffic: from outside (client) inward (services), at
  the boundary.
- **Service mesh** (Istio, Linkerd) — **east-west** traffic: **between** services inside,
  via sidecars; takes on retries, mTLS, tracing, load balancing of internal calls.
They complement each other: the gateway at the entrance, the mesh inside.

---

## Pitfalls

- The gateway can become a **bottleneck** and a single point of failure → it too is scaled
  and made redundant.
- Risk of a **"god object"** — stuffing business logic into the gateway bloats it. Keep it
  thin: routing and cross-cutting concerns, **not business rules** (those live in
  services).

---

## Interview phrasing

> "An API gateway — a single entrance in front of microservices: it routes to services,
> centralizes cross-cutting concerns (authentication, rate limiting, TLS, logging), and
> aggregates responses, hiding the internal topology from the client and removing the
> plumbing from services. For different clients — the BFF pattern (a gateway per
> mobile/web). Not to be confused with a service mesh: the gateway is north-south
> (client→services) at the boundary, the mesh is east-west (between services) inside. I
> keep the gateway thin, without business logic, and scale it so it doesn't become a
> bottleneck/SPOF."

---

See [10-api-gateway.tasks.md](./10-api-gateway.tasks.md) — tasks. Solutions in
[10-api-gateway.answers.md](./10-api-gateway.answers.md).
