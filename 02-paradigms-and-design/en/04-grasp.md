# GRASP patterns

> 🌐 Russian version: [04-grasp.md](../ru/04-grasp.md)

Nine principles of **responsibility assignment** (Craig Larman). Reads as a story;
all nine answer one question.

The through-line of the topic:

> **GRASP answers one design question: who should be assigned this responsibility?
> These aren't ready-made constructs like [GoF](./05-design-patterns-gof.md), but
> principles of *reasoning*. The order: GRASP first decides *who* is responsible
> for what, then GoF gives a ready form for how to shape it. GRASP is the
> foundation under the patterns.**

The key metrics everything reduces to are **low coupling** and **high cohesion**
(in detail in [coupling-cohesion-principles](./06-coupling-cohesion-principles.md));
the other seven principles work toward them.

---

## 1. Information Expert
Assign a responsibility to the one **who has the data** for it. Computing an
order's total should be `Order`'s job — it has the line items — not an external
`OrderCalculator` that would have to pull everything out. Why: logic lives next to
data → fewer getters exposed, lower coupling.

## 2. Creator
Assign creating object B to class A if A **contains/aggregates/closely uses** B.
`Order` creates `OrderLine` (it contains them). Why: the owner creates; connections
don't smear around.

## 3. Controller
The first object behind the UI that receives a **system operation** and coordinates
it — not the UI itself, not a domain object. On the web it's a controller/handler
(including Nest's `@Controller`): received a request, delegated work to services,
returned a response. Why: the UI doesn't know business logic, the domain doesn't
know about HTTP.

## 4. Low Coupling
Minimize dependencies between modules. The less a class knows about others, the
easier to change, reuse, test. This is the **goal** the other principles serve.

## 5. High Cohesion
Keep only **closely related** things in a class, one focused role (echoes
[SRP](./03-solid.md)). A "Swiss-army-knife" class has low cohesion; it's hard to
understand and change.

## 6. Polymorphism
When behavior varies **by type**, express it with polymorphism, not `if/switch` by
type. (The same OCP stands on.) Different payment methods → a `PaymentMethod`
interface + implementations, not `switch (type)`.

## 7. Pure Fabrication
Sometimes there's no domain class you can logically hand a responsibility to — so
you introduce an **artificial** class (not from the problem domain) for the sake of
low coupling / high cohesion. Classics: `Repository` (persistence), `Service`,
`Mapper`. `Order` shouldn't know SQL itself (that would blur its role) → we fabricate
`OrderRepository`.

## 8. Indirection
To decouple two objects, insert an **intermediary** between them. That's how
`Adapter`, `Mediator`, a controller, an event bus work: A doesn't depend on B
directly but communicates through an in-between layer. Why: decouple, but at the
cost of an extra link — don't overuse.

## 9. Protected Variations
Wrap points of **likely change** behind a stable interface so that wobble in one
place doesn't ripple across the system. You hide a concrete DB behind `Repository`,
an external API behind an adapter. Essentially a generalization of DIP/OCP: "find
what's most likely to change and put a seam there".

---

## How this relates to SOLID and GoF

GRASP is the **reasoning** level ("who to give the responsibility to"), SOLID is a
set of structural **rules**, GoF is **ready-made constructs**. They view the same
thing from different heights: Information Expert + High Cohesion ≈ SRP; Polymorphism
≈ OCP; Protected Variations ≈ DIP/OCP; Indirection materializes as GoF's
Adapter/Mediator. The shared goal of all — **low coupling + high cohesion**.

---

## Interview phrasing

> "GRASP is nine principles about who to assign a responsibility to. The key ones:
> Information Expert (responsibility to who has the data), Creator (the owner
> creates), Controller (coordinator of a system operation behind the UI, like a
> Nest controller). It all serves two metrics — Low Coupling and High Cohesion.
> Polymorphism removes switch-by-type, Pure Fabrication introduces an artificial
> class like Repository for role purity, Indirection puts an intermediary for
> decoupling, Protected Variations wraps likely changes behind a stable interface.
> GRASP is how to reason about responsibility distribution; SOLID and GoF are the
> rules and ready forms on top."

---

See [04-grasp.tasks.md](./04-grasp.tasks.md) — tasks. Solutions in
[04-grasp.answers.md](./04-grasp.answers.md).
