# GRASP — solutions

> 🌐 Russian version: [04-grasp.answers.md](../ru/04-grasp.answers.md)

---

## Question 1
All nine answer one question: **who should be assigned this responsibility?** In
terms of level, GRASP is principles of **reasoning** about responsibility
distribution, while GoF is **ready-made constructs** (recipes). The order: GRASP
first decides *who* is responsible, then GoF gives the form for *how* to shape it.
GRASP is the foundation under the patterns.

## Question 2
By **Information Expert** — `Order` itself: it has the data (the line items), so it
has the responsibility to sum them. Extracting it into an `OrderCalculator` that
pulls everything out is bad because it forces `Order` to expose its internals via
getters (breaks encapsulation), **raises coupling** (the calculator depends on the
order's structure), and smears logic away from the data. Logic should live next to
the data.

## Question 3
**Pure Fabrication** — an artificial class that **doesn't exist in the problem
domain**, introduced when no domain class can logically take a responsibility. A
typical backend example is a **`Repository`** (save/load): `Order` shouldn't know
SQL, or its role blurs. It's introduced for two metrics — **low coupling** (the
domain doesn't depend on the DB) and **high cohesion** (`Order` keeps only domain
logic). Same for `Service`, `Mapper`.

## Question 4
It's suggested by **Polymorphism**: behavior that varies by type is expressed with
polymorphism, not `switch (type)`. The solution — a `PaymentMethod` interface with
a `pay()` method and one implementation per method (`CardPayment`, `PaypalPayment`,
...); the client calls `payment.pay()`, the three `switch`es disappear. A new method
= a new class, existing code untouched (that's also OCP).

## Question 5
**Protected Variations** — wrap points of **likely change** behind a stable
interface so the wobble doesn't ripple across the system. The link: it's a
generalization of **DIP** (depend on an abstraction, not a detail) and **OCP**
(extend without changing). A "seam" example: a concrete DB hidden behind an
`OrderRepository` interface; swap Postgres for Mongo — only the repository
implementation changes, all the business logic over the interface stays untouched.
You put the seam where change is **most likely**.

## Question 6
**Controller** — the coordinator of a system operation between UI and domain. The
UI shouldn't receive a business operation, because then business logic leaks into
the interface (can't reuse, hard to test); a domain object shouldn't, because it
doesn't know about the transport (HTTP, a queue) and the coordination of several
services. In **NestJS** it's a class with `@Controller`: receives a request,
delegates work to services, returns a response — it holds no business logic itself.

## Question 7
**Information Expert** says "responsibility where the data is", **High Cohesion**
says "keep only closely related things in a class, one role". Together they give a
class that does **one** thing and answers to **one** customer — which is exactly
**SRP** ("one reason to change"). SRP is the same principle from the "how many
reasons to change" angle; GRASP from the "who to assign what" angle.
