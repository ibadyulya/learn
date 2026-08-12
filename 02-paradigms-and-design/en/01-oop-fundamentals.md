# OOP fundamentals

> 🌐 Russian version: [01-oop-fundamentals.md](../ru/01-oop-fundamentals.md)

Object-oriented programming as a way to **manage complexity**. Reads as a story;
the four "pillars" are introduced not as a list but through the problem each one
solves.

The through-line of the topic:

> **OOP is packing data and behavior into objects that hide their internals and
> communicate through a clear interface. The main pillar is encapsulation
> (hiding); the other three (abstraction, inheritance, polymorphism) serve the
> same goal: letting parts of the system change independently.**

---

## Why objects at all

Without OOP, data and functions live apart: the `user` structure here, a dozen
functions that manipulate it there, and anyone can reach into a field directly. As
the code grows, nobody knows who changes the state and when. OOP says: **keep data
and the operations on it together, and expose only what's needed.** An object is a
"box with state and buttons"; the internals are hidden.

---

## 1. Encapsulation — hide state behind an interface

The most important pillar. An object **hides internal data** and gives access only
through methods. From outside you don't touch fields directly — you ask the object
to do something.

```ts
class BankAccount {
  #balance = 0;                       // private (# is a real private in JS)
  deposit(sum: number) {
    if (sum <= 0) throw new Error('sum must be positive');
    this.#balance += sum;             // the invariant stays under control
  }
  get balance() { return this.#balance; }
}
```
Why: the object **guards its own invariants** (the balance won't go negative from a
crooked external assignment). Change the internal representation and external code
won't notice, as long as the interface is the same. This is the foundation of
modularity.

---

## 2. Abstraction — show the "what", hide the "how"

Abstraction is extracting the essential and hiding implementation details behind a
simple interface. You call `array.sort()` without knowing the algorithm;
`repo.save(user)` without knowing the SQL.

Encapsulation and abstraction are two sides of one coin: **abstraction is about
interface design** (what to show), **encapsulation is about the hiding mechanism**
(how to hide the rest).

---

## 3. Inheritance — reuse via "is-a"

A subclass gets the parent's fields and methods and can extend/override them.
`class Admin extends User`.

But here's the main caution: **inheritance is overrated.** It creates a rigid
coupling (the subclass knows the parent's internals and breaks when they change —
the "fragile base class problem") and is often applied by "sounds similar",
violating [LSP](./03-solid.md). The rule:

> **Prefer composition over inheritance.** "Has-a" (an object *contains* another
> and delegates to it) is more flexible than "is-a".

```ts
// instead of class Car extends Engine — composition:
class Car { constructor(private engine: Engine) {} start() { this.engine.run() } }
```
Reach for inheritance when there's a genuine "is-a" AND substitution works (LSP);
otherwise — composition.

---

## 4. Polymorphism — one interface, many implementations

"Many forms": different classes respond to the same call in their own way, and the
calling code doesn't know and doesn't want to know who exactly it's talking to.

```ts
interface Shape { area(): number }
const shapes: Shape[] = [new Circle(2), new Square(3)];
shapes.forEach(s => console.log(s.area()));   // each computes its own way
```
Why: you can add a new shape without touching the loop — this is the mechanism
[OCP](./03-solid.md) stands on. Polymorphism turns `if (type===...)` into a method
call.

Kinds: **subtype** polymorphism (via interfaces/inheritance, as above),
**parametric** (generics — `Array<T>`), **ad-hoc** (overloading). In interviews
they almost always mean the first.

---

## OOP in JavaScript: classes are sugar over prototypes

An important quirk (details in [this-prototypes-inheritance](../../01-js-typescript/en/05-this-prototypes-inheritance.md)):
JS has no "real" classes like Java. `class` is syntactic sugar over **prototypal**
inheritance: an object references a prototype object from which it takes methods.
`extends` builds a prototype chain. You need to understand this so you're not
surprised by `this`, `Object.create`, and why methods live in one place rather than
being copied into every instance.

---

## Interview phrasing

> "OOP packs state and behavior into objects to manage complexity. Encapsulation —
> hide data behind methods so the object guards its own invariants and internals
> can change without touching clients. Abstraction — show the 'what', hide the
> 'how'. Polymorphism — one interface, many implementations; OCP stands on it.
> Inheritance — reuse by is-a, but it creates rigid coupling and fragility, so by
> default I prefer composition (has-a) and use inheritance only for a genuine is-a
> that honors LSP. In JS, classes are sugar over prototypal inheritance."

---

See [01-oop-fundamentals.tasks.md](./01-oop-fundamentals.tasks.md) — tasks.
Solutions in [01-oop-fundamentals.answers.md](./01-oop-fundamentals.answers.md).
