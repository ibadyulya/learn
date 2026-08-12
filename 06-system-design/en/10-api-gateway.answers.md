# API gateway — solutions

> 🌐 Russian version: [10-api-gateway.answers.md](../ru/10-api-gateway.answers.md)

---

## Question 1
1. The client **knows the whole topology** (service addresses/split) → tight coupling, a
   change to the internal structure breaks the client.
2. **Duplication of cross-cutting plumbing** (authentication, rate limiting, CORS, logs)
   in every service.
3. One screen = **many round trips** to different services → latency and complexity on the
   client.

## Question 2
**Routing** (by paths to services), **authentication/authorization** (token check at the
entry), **rate limiting/throttling**, **TLS termination**, response **aggregation**,
request/response transformation, caching, logging/metrics. The point — remove this from
all services and do it once at the boundary.

## Question 3
**Aggregation** — the gateway assembles **one** response from several services (calls them
itself and stitches the result). It solves the **many round trips** problem: instead of 5
requests to different services, the client makes one to the gateway; less latency, a
simpler client, and it doesn't know the internal topology.

## Question 4
A **BFF (Backend for Frontend)** — a separate gateway **per specific client type** (mobile,
web, partner API) that aggregates and shapes the response for its needs (mobile — more
compact, web — fuller). It's needed so you don't make one universal gateway bloated for
everyone, but give each client an optimal contract.

## Question 5
- **API gateway** — **north-south** traffic: from outside (client) inward (services), at
  the system boundary; centralizes the entry.
- **Service mesh** — **east-west** traffic: **between** services inside, via sidecars;
  takes on retries, mTLS, balancing, tracing of internal calls.
They complement each other: the gateway at the entrance, the mesh in internal
communication.

## Question 6
A fat gateway with business logic turns into a **"god object"**: it bloats, becomes a
bottleneck and a coupling point (every feature touches the gateway). It should hold only
**routing and cross-cutting concerns** (auth, limits, TLS, aggregation), while
**business rules live in services**. A thin gateway = high cohesion and independent
services.

## Question 7
The gateway is a **single entry point** → potentially a **bottleneck** and a **SPOF**: if
the gateway is down, everything is unavailable. Mitigate with: **horizontal scaling** of
the gateway (several instances behind a balancer), **redundancy**/health-checks, keeping
it lightweight (no heavy logic), timeouts/circuit breaker to services, caching.
