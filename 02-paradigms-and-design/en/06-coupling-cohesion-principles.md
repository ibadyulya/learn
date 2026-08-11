# Coupling, cohesion & core principles

> 🌐 Russian version: [06-coupling-cohesion-principles.md](../ru/06-coupling-cohesion-principles.md)

The two main metrics of structural quality — plus a set of practical rules
(DRY/KISS/YAGNI, the Law of Demeter) that serve those same two metrics. Reads as a
story.

The through-line of the topic:

> **Good structure reduces to two quantities: cohesion — how much the elements
> inside a module belong to one task — which we maximize; coupling — how much
> modules depend on each other — which we minimize. The goal of all design: high
> cohesion + low coupling. The other principles (SOLID, GRASP, DRY, Law of Demeter)
> are ways to achieve it.**

---

## Cohesion — "everything in the module is about one thing"

How much the elements of one module/class **belong to one task**. High cohesion —
the class is focused on one role; low — a "Swiss army knife" / a `utils` dump where
DB, formatting, and validation are mixed.

```ts
// Low cohesion: the class does everything
class Helper { formatDate() {} sendEmail() {} parseCsv() {} hashPassword() {} }
// High: each about one thing
class DateFormatter {} class Mailer {} class CsvParser {} class PasswordHasher {}
```
Why: a focused module is easier to understand, change, and reuse. A direct relative
of [SRP](./03-solid.md) and High Cohesion from [GRASP](./04-grasp.md).

---

## Coupling — "how much a module knows about others"

How much one module **depends** on another's internals. Tight coupling — the module
knows another's implementation details, reaches into its fields, breaks when it
changes. Loose — they communicate through a narrow stable interface.

```ts
// Tight: OrderService knows the concrete Postgres and its API
class OrderService { private db = new PostgresClient(); save(o) { this.db.rawInsert(o) } }
// Loose: depends on an abstraction, the concretion is injected
class OrderService { constructor(private repo: OrderRepository) {} save(o) { this.repo.save(o) } }
```
Why: the less a module knows about others, the safer to change, test (plug a mock),
and reuse it. Loose coupling is the goal DIP, interfaces, and Pure Fabrication exist
for.

Signs of tight coupling: `a.b.c.d` chains, dependence on global state/singletons,
"changed here — broke there", a module importing a dozen others.

---

## Why they're always named as a pair

They **pull the same way**: if each module is focused on its role (high cohesion),
it needs to know less about others (low coupling). And vice versa: a God class that
does everything inevitably clings to everything. So the goal is stated as a pair —
**high cohesion + low coupling**. It's essentially the criterion for "is the system
well split into modules".

---

## The Law of Demeter — "don't talk to strangers"

A practical rule for low coupling: a method should talk only to its **nearest
neighbors** — its own fields, arguments, things it created itself — not reach deep
into other objects.

```ts
// Bad: the chain knows the whole structure end to end
order.getCustomer().getAddress().getCity().toUpperCase();
// Better: ask the nearest object
order.getCustomerCity();
```
Why: a long `a.b.c.d` chain rigidly ties the code to the internal structure of
every link — change any one and the call breaks. Ask the one you work with
directly.

---

## DRY / KISS / YAGNI — practical reminders

- **DRY (Don't Repeat Yourself)** — don't duplicate **knowledge**: one rule lives
  in one place. Careful: DRY is about duplicating *meaning*, not similar-looking
  code. Forcibly merging two coincidentally similar pieces = **coupling the
  unrelated** ("the wrong abstraction" is worse than duplication). Duplication is
  cheaper than the wrong abstraction.
- **KISS (Keep It Simple)** — a simple solution beats a clever one. Clever code is
  more expensive to maintain.
- **YAGNI (You Aren't Gonna Need It)** — don't build ahead "for the future". An
  abstraction for a non-existent requirement = dead weight and extra coupling.
  Introduce it when the need has **actually appeared** (the rule of three).

All three are about **managing complexity**: don't add it needlessly, but don't
duplicate knowledge either.

---

## Interview phrasing

> "I measure structural quality by two quantities: cohesion — how much a module is
> about one task (I maximize it, that's SRP), and coupling — how much modules depend
> on each other (I minimize it, which is what DIP, interfaces, injection are for).
> They're paired: a focused module knows less about others. Loose coupling is kept
> in practice by the Law of Demeter (don't reach through `a.b.c.d`), depending on
> abstractions, and narrow interfaces. DRY — one piece of knowledge in one place,
> but not to be confused with similar code (the wrong abstraction is worse than
> duplication); KISS — simpler is better; YAGNI — don't abstract ahead of need. All
> of it is about high cohesion + low coupling."

---

See [06-coupling-cohesion-principles.tasks.md](./06-coupling-cohesion-principles.tasks.md)
— tasks. Solutions in [06-coupling-cohesion-principles.answers.md](./06-coupling-cohesion-principles.answers.md).
