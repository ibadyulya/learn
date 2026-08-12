# Microservices vs monolith

> 🌐 Russian version: [08-microservices-vs-monolith.md](../ru/08-microservices-vs-monolith.md)

One big service or many small ones — and why it's not about "trendy". Reads as a
story.

The through-line of the topic:

> **A monolith — the whole application in one deployment. Microservices — a set of
> small, independently deployable services around business capabilities.
> Microservices give independence (deploy, scale, teams) at the cost of distributed
> complexity. It's not the default: start with a monolith, split when the pain is
> real.**

---

## Monolith — everything in one

One codebase, one deployment, usually one DB. Inside — modules, but it's deployed and
scaled **as a whole**.
- ➕ **simple**: one deploy, local calls (not the network), an end-to-end transaction in
  one DB, easy to test and debug;
- ➖ with growth: **coupling** blurs, deploy everything for one line, can't scale/choose
  technology **per part**, a big team jostles in one repo.

A "modular monolith" (clear internal boundaries) removes some downsides without going
distributed.

---

## Microservices — many independent ones

The application is split into **services around business capabilities** (orders,
payments, notifications), each:
- **deployed** and scaled **independently**;
- **owns its data** (its own DB — **database per service**), doesn't touch another's
  directly;
- communicates with others **over the network** (HTTP/gRPC or events via a broker, see
  [event-driven-messaging](./02-event-driven-messaging.md)).

Benefits:
- **independent deploy** — ship one service without touching the rest;
- **independent scale** — scale only the bottleneck (see [scaling](./05-scaling.md));
- **team autonomy** — each owns a service and its technologies;
- **fault isolation** — one service goes down, the rest (with the right design) stay
  alive.

---

## The price — distributed complexity

For independence you pay by **everything becoming a distributed system**:
- **the network is unreliable** — calls fail/lag → retries, timeouts, circuit breaker
  (see [reliability-patterns](./03-reliability-patterns.md));
- **no shared transaction** — consistency between services via **saga/2PC** and
  idempotency (see [saga-2pc](./09-saga-2pc.md));
- **observability** is harder — a request goes through many services → you need
  distributed tracing (see [observability](./11-observability.md));
- harder **testing, deployment, contract versioning**, higher infra overhead.

This is exactly the CAP/trade-off thinking from [foundations](./01-foundations.md):
distribution isn't free.

---

## When to use what

- **Monolith** — by default, especially for a startup/small team/unclear domain: faster,
  simpler, cheaper.
- **Microservices** — when the pain is real: big teams get in each other's way, parts of
  the system need to scale/deploy independently, different technology requirements.

> "You must be **this tall**": don't start with microservices for the hype — their
> complexity pays off only at organizational scale. The common path is monolith →
> modular monolith → extract services as needed.

---

## Interview phrasing

> "A monolith — one deployment: simple, local calls, an end-to-end transaction, but
> deployed/scaled as a whole and it sprawls with growth. Microservices — services around
> business capabilities, each independently deployed/scaled and owning its DB,
> communicating over the network/events. The benefit — independence of deploy/scale/teams
> and fault isolation; the price — distributed complexity: an unreliable network
> (retries, circuit breaker), consistency via saga/idempotency, distributed tracing,
> harder deploy/tests. So by default a monolith (better modular), and I split into
> microservices when the independence is really needed — it's a trade-off for scale, not
> a fashion."

---

See [08-microservices-vs-monolith.tasks.md](./08-microservices-vs-monolith.tasks.md) —
tasks. Solutions in [08-microservices-vs-monolith.answers.md](./08-microservices-vs-monolith.answers.md).
