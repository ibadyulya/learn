# Saga and 2PC — solutions

> 🌐 Russian version: [09-saga-2pc.answers.md](../ru/09-saga-2pc.answers.md)

---

## Question 1
Three services = **three different DBs without a shared transaction**. ACID atomicity
works only **within one** DB, while "payment + reserve + order" touches three systems —
you can't wrap them in a single commit/rollback (the dual-write problem). A crash midway
leaves some steps done and some not: inconsistency.

## Question 2
1. **Prepare (voting):** the coordinator asks all participants "ready to commit?"; each
   reserves the changes and answers yes/no, **without committing**.
2. **Commit/Abort:** all yes → a commit command to everyone; even one no → abort to
   everyone.
Downsides: (1) **blocking** — resources locked between phases, if the coordinator crashes
→ participants hang; (2) **the coordinator is a single point of failure**; (3) **slow**,
scales poorly.

## Question 3
A saga is a **sequence of local transactions**: each service commits its own DB and
triggers the next step. On a step **failure**, **compensating transactions** run for the
already-done steps — in reverse order they "undo" the effect semantically (refund the
money, cancel the reserve). The system reaches consistency not instantly.

## Question 4
A **rollback** undoes an **uncommitted** transaction within one DB — as if it never
happened. In a saga the previous steps are **already committed** in their DBs (money is
really debited), you can't "unhappen" them. A **compensation** is a **new reverse
action**: refund the money, cancel the reserve. The effect is neutralized, but both
operations remain in history (the debit and the refund).

## Question 5
- **Orchestration** — a central **orchestrator** explicitly calls steps and compensations;
  the process logic is in one place (visible as a whole, easier to debug), but it's a node
  of complexity and a potential SPOF.
- **Choreography** — no center: services react to **events** and publish their own; looser
  coupling, but the process is "smeared" across services — harder to understand and debug.
The trade-off: explicitness/centralization vs autonomy/decentralization.

## Question 6
A saga gives **eventual consistency**: the system will reach a consistent state, but **not
instantly**. So the user may see **intermediate states** — an order "in progress",
temporarily reserved goods, a debit before confirmation. The UX accounts for this
(statuses, "please wait"), rather than pretending everything is atomic.

## Question 7
Because steps and events in distribution **repeat** (retries on network failures, message
redelivery — at-least-once). If a step isn't idempotent, a repeat runs it twice (a double
debit). Idempotency (by key/state) guarantees that repeating the same step **won't change
the result** — a mandatory condition for a reliable saga.
