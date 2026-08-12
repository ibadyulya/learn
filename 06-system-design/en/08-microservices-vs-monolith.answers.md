# Microservices vs monolith — solutions

> 🌐 Russian version: [08-microservices-vs-monolith.answers.md](../ru/08-microservices-vs-monolith.answers.md)

---

## Question 1
- **Monolith** — one deployment, one codebase, usually one DB; deployed and scaled **as a
  whole**.
- **Microservices** — a set of independently deployable services, each **owns its DB** and
  communicates with others over the network/events.
Key: a monolith is one, microservices are a distributed system of autonomous parts.

## Question 2
Benefits: (1) **independent deploy** — ship a service without touching others; (2)
**independent scale** — scale only the bottleneck; (3) **team autonomy** and technology
freedom (+ fault isolation). Not free because everything becomes a **distributed system**
with an unreliable network, complex consistency, observability, and deployment.

## Question 3
**Database per service** — each service has **its own** DB, and only it writes/reads it.
Reaching into another's DB directly is forbidden because it creates **hidden tight
coupling**: the schema becomes a shared contract, can't be changed independently, deploy
autonomy is lost — essentially a monolith again, only worse (distributed). Communication
— only via API/events.

## Question 4
As soon as calls go over the network: **the network is unreliable** (failures, timeouts,
latency) → you need retries, timeouts, a circuit breaker; **partial failures** (one
service down); **no shared transaction** between services; harder **debugging/tracing** (a
request through many services); contract versioning, latency. None of this exists with
local calls in a monolith.

## Question 5
Two services are two systems **without a shared transaction**, you can't atomically wrap
"debit in payments + create in orders" (the same dual-write problem). Instead of a simple
transaction: a **saga** (a sequence of local transactions with compensations on failure),
**2PC** (two-phase commit, expensive and blocking), and **idempotency** — eventual
consistency instead of instant.

## Question 6
A **modular monolith** — one deployment, but with **clear internal boundaries** between
modules (like future services), with minimal coupling. It gives some of the structural
benefits (isolation, clear contracts) **without** the distributed complexity, and eases
future extraction of services if needed. A good default: start with a modular monolith,
split as the real need arises.

## Question 7
Mistaken because: (1) microservices are **not "just scalable"** — they bring **distributed
complexity** (network, consistency, tracing, deployment) that only slows down a small
team/early product; (2) a monolith **also scales** (instances behind a balancer, DB
replication); (3) without a clear domain, service boundaries are hard to guess — bad
splitting is worse than a monolith. The right way — start with a monolith and split when
independence is really needed (organizational scale, not hype).
