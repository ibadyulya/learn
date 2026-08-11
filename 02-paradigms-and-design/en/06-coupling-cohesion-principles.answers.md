# Coupling, cohesion & principles — solutions

> 🌐 Russian version: [06-coupling-cohesion-principles.answers.md](../ru/06-coupling-cohesion-principles.answers.md)

---

## Question 1
- **Cohesion** — how much the elements inside a module belong to **one** task/role.
  **Maximize it.**
- **Coupling** — how much a module **depends** on other modules. **Minimize it.**
Design goal: high cohesion + low coupling.

## Question 2
Because they **pull the same way**. If a module is focused on one role (high
cohesion), it needs less data and fewer services from others → it knows less about
them (low coupling). And vice versa: a God class doing everything inevitably depends
on everything. One almost automatically drags the other, so the goal is stated as a
pair.

## Question 3
This is **low cohesion**: unrelated responsibilities (formatting, email, CSV,
hashing) dumped into one class — a "utils dump" / God class. Bad: hard to
understand, any of those concerns drags edits to the class, you can't reuse parts.
Fix — **split by role**: `DateFormatter`, `Mailer`, `CsvParser`, `PasswordHasher`,
each with high cohesion (echoes SRP).

## Question 4
The **Law of Demeter** ("don't talk to strangers") — a method should talk only to
its nearest neighbors (its own fields, arguments, things it created), not reach deep
into other objects. The chain `order.getCustomer().getAddress().getCity()` is bad
because it's rigidly tied to the internal structure of Customer and Address — change
any link and the call breaks (tight coupling). Rewrite it by asking the nearest
object: `order.getCustomerCity()` — the detail is hidden behind a method.

## Question 5
"Remove any duplication" is dangerous because DRY is about duplicating **knowledge**
(one meaning), not **similar-looking** code. Two pieces may coincidentally match now
but change for different reasons. Forcibly merging them yields a shared abstraction
pulled in different directions by two different requirements — you end up
parameterizing it with flags, it grows complex and **couples the unrelated**. "The
wrong abstraction is worse than duplication": it's easy to extract commonality from
duplication later, but expensive to untangle a wrong abstraction. First wait until
the code duplicates the **meaning**.

## Question 6
YAGNI says don't build an abstraction for a requirement that doesn't exist yet.
Over-engineering is exactly its violation: introducing layers ahead of time "for the
future". Example: for a single payment method you immediately build a
`PaymentStrategyFactory` + interface + registry "in case others get added". While
there's one method, that's just extra classes and indirection, and `PaymentService`
now depends on the factory and registry — **coupling has grown** with no benefit.
The right way — a simple implementation now, the abstraction when a second/third case
appears (the rule of three).

## Question 7
Signs of tight coupling "at a glance":
1. **`a.b.c.d` chains** — reaching deep into other objects (a Demeter violation).
2. **Dependence on globals/singletons** — a hidden link, invisible in the signature.
3. **"Changed here — broke there"** — an edit in one module requires edits in
   seemingly unrelated places.
4. **A long import list** — the module knows about a dozen others / `new
   ConcreteClass()` instead of depending on an interface.
