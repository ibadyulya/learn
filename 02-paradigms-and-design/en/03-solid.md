# SOLID principles

> 🌐 Russian version: [03-solid.md](../ru/03-solid.md)

Five principles of object-oriented design (Robert Martin). The goal isn't
"pretty for its own sake" but code you can **change without breaking the rest**.
Reads as a story: each principle is introduced through the concrete pain it cures.

The through-line of the whole topic:

> **SOLID is five ways of saying the same thing: code should be arranged so that
> a change in requirements touches few places and doesn't "leak" across the whole
> system. Each letter fights its own kind of fragility, but all five are about
> localizing change and managing dependencies.**

Keep the main enemy in mind: **change is inevitable.** Requirements always change.
Bad design is when one change ripples across dozens of files and breaks something
along the way. Good design is when the change stays **local**. All five principles
are about that.

---

## S — Single Responsibility Principle

**A class (module, function) should have one reason to change.**

The main confusion: "one responsibility" ≠ "does one thing". The precise wording
is **one reason to change** — one "customer", one axis of change. A class that
calculates pay, renders a report, and saves to the DB has **three** reasons to
change (accounting rules, report format, DB schema) and three different customers.

```ts
// Bad: three reasons to change in one class
class Employee {
  calculatePay() { /* accounting rules */ }
  renderReport() { /* HR format */ }
  save()         { /* DB schema */ }
}
```
```ts
// Good: each axis of change is its own class
class PayCalculator      { calculate(e: Employee): Money { /* ... */ } }
class EmployeeReport     { render(e: Employee): string   { /* ... */ } }
class EmployeeRepository { save(e: Employee): void       { /* ... */ } }
```

Why: when the accountant changes the pay rules, you touch only `PayCalculator` —
the report and DB are irrelevant, zero risk of breaking them. **The change is
local.** The flip side — don't shatter it to absurdity: a class per tiny action
breeds a pile of trivia. The guide isn't "one action" but "one source of
requirements".

---

## O — Open/Closed Principle

**Entities are open for extension but closed for modification** — new behavior is
added, existing code isn't rewritten.

The pain it cures: every new case is an edit to the same ever-growing
`switch`/`if-else` that's scary to touch, because proven logic lives right there.

```ts
// Bad: a new shape type = editing this function
function area(shape: any): number {
  if (shape.kind === 'circle') return Math.PI * shape.r ** 2;
  if (shape.kind === 'square') return shape.s ** 2;
  // add a triangle → back in HERE, risking the circle and square
}
```
```ts
// Good: extend by adding a class, don't touch area()
interface Shape { area(): number }
class Circle implements Shape { constructor(private r: number) {} area() { return Math.PI * this.r ** 2 } }
class Square implements Shape { constructor(private s: number) {} area() { return this.s ** 2 } }
// a new triangle = a new class Triangle implements Shape; old code unchanged
```

The mechanism is **polymorphism**: a common interface + new implementations. Why:
new functionality physically can't break the proven old code, because the old
code isn't touched.

Honest caveat: OCP isn't absolute. You can't guess every axis of extension in
advance, and abstraction "just in case" is over-engineering. Close it where
changes already **repeat** (the second, third such `if` is the signal to extract
polymorphism).

---

## L — Liskov Substitution Principle

**An object of a subtype must be substitutable for the base type without breaking
the program's correctness.** A subclass has no right to "lie" about the parent's
contract.

The pain: inheriting by outward form, not by behavior. The classic is `Square
extends Rectangle`. A rectangle's width and height are independent; a square's
aren't. Code correct for `Rectangle` breaks when handed a `Square`:

```ts
class Rectangle {
  constructor(protected w: number, protected h: number) {}
  setW(w: number) { this.w = w }
  setH(h: number) { this.h = h }
  area() { return this.w * this.h }
}
class Square extends Rectangle {
  setW(w: number) { this.w = w; this.h = w }   // breaks independence of sides
  setH(h: number) { this.w = h; this.h = h }
}

function stretch(r: Rectangle) {
  r.setW(5); r.setH(4);
  console.assert(r.area() === 20);   // Rectangle → 20; Square → 16, fails
}
```

Signs of an LSP violation: the subclass **throws** where the base works;
**strengthens preconditions** (demands more than the parent); **weakens
postconditions** (guarantees less); `NotImplemented` stub methods.

Why it's critical: polymorphism — and hence OCP — **rests on LSP**. If a subtype
can't be honestly substituted, all polymorphic code turns into a minefield of
`if (x instanceof ...)`. The cure — **composition over inheritance** and modeling
by behavior, not by "sounds similar" (a square is not a kind of mutable
rectangle).

---

## I — Interface Segregation Principle

**A client must not be forced to depend on methods it doesn't use.** Better
several narrow interfaces than one "fat" one.

The pain: a fat interface forces implementing classes to drag in the unneeded —
usually via `throw` stubs — and any change to the interface hits even those who
never wanted those methods.

```ts
// Bad: one fat interface
interface Worker { work(): void; eat(): void }
class Robot implements Worker {
  work() { /* ... */ }
  eat()  { throw new Error('a robot does not eat') }   // smell: implementing the unneeded
}
```
```ts
// Good: split by role
interface Workable { work(): void }
interface Eatable  { eat(): void }
class Robot implements Workable          { work() {} }
class Human implements Workable, Eatable { work() {} eat() {} }
```

Why: narrow interfaces = fewer false dependencies, fewer forced stubs, changes
don't spread. In essence **ISP is SRP at the interface level**: an interface, too,
should have one role.

---

## D — Dependency Inversion Principle

**High-level modules should not depend on low-level modules — both depend on
abstractions.** And abstractions don't depend on details; details depend on
abstractions.

The pain: business logic directly creates and calls a concrete DB/service — you
can't reuse it, test it, or swap the storage; it's all nailed down.

```ts
// Bad: the high level is nailed to a concretion
class OrderService {
  private db = new PostgresClient();     // hard coupling
  place(o: Order) { this.db.insert(o) }
}
```
```ts
// Good: depend on an abstraction, inject the concretion from outside
interface OrderRepository { save(o: Order): void }

class OrderService {
  constructor(private repo: OrderRepository) {}   // dependency injection
  place(o: Order) { this.repo.save(o) }
}
class PostgresOrderRepo implements OrderRepository { save(o: Order) { /* ... */ } }
class InMemoryOrderRepo implements OrderRepository { save(o: Order) { /* for tests */ } }
```

What "inverts": before, `OrderService` (high level) depended on `PostgresClient`
(low level). Now **both depend on the interface** `OrderRepository`. The
dependency arrow, once pointing at the detail, now points at the abstraction.
Hence swapping in `InMemoryOrderRepo` for tests, changing the DB in production
without touching the logic.

Don't conflate three things:
- **DIP** — the *principle*: depend on abstractions.
- **DI (dependency injection)** — the *technique*: pass dependencies from outside
  (via the constructor) instead of creating them inside. A way to achieve DIP.
- **IoC (inversion of control)** — the umbrella for "control handed outward";
  DI containers (including NestJS's) are its implementation.

---

## How the five hold together

They aren't independent — it's one idea from five angles:
- **SRP** and **ISP** — about a class/interface having one role (ISP = SRP for
  interfaces).
- **OCP** — the goal (extend without touching the old) — is **achievable only
  through polymorphism**, and polymorphism is correct **only under LSP**. So OCP
  stands on LSP.
- **DIP** makes OCP possible at the module level: swapping an implementation
  behind an interface = extending the system without changing its core.

The common denominator — **manage dependencies and localize change.**

---

## Interview phrasing

> "SOLID is five principles serving one goal: making a requirement change local.
> SRP — one class, one reason to change (not to be confused with 'does one
> thing'). OCP — add new behavior through polymorphism, without rewriting proven
> code. LSP — a subtype must honestly substitute for the base, otherwise
> polymorphism (and OCP) breaks; the classic counter-example is Square/Rectangle.
> ISP — narrow interfaces instead of fat ones, so a client doesn't depend on the
> unneeded; it's SRP for interfaces. DIP — high and low levels depend on an
> abstraction, not on each other; in practice achieved through dependency
> injection and it underpins DI containers like Nest's."

---

See [03-solid.tasks.md](./03-solid.tasks.md) — recognition & fix tasks. Solutions
in [03-solid.answers.md](./03-solid.answers.md).
