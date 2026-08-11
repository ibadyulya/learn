# OOP fundamentals — solutions

> 🌐 Russian version: [01-oop-fundamentals.answers.md](../ru/01-oop-fundamentals.answers.md)

---

## Question 1
- **Encapsulation** — hide state behind methods so the object guards its
  invariants.
- **Abstraction** — show the "what", hide the "how".
- **Inheritance** — reuse by is-a.
- **Polymorphism** — one interface, many implementations.

The main one is **encapsulation**: hiding state is what gives modularity (you can
change internals without touching clients). The other three serve the same goal.

## Question 2
- **Abstraction** — about **interface design**: what to expose as essential
  (`repo.save(user)`).
- **Encapsulation** — about the **hiding mechanism**: how to hide and protect the
  rest (private fields, invariants in methods).
Abstraction decides *what to show*, encapsulation *how to hide and protect it*.

## Question 3
Inheritance problems: (1) **rigid coupling** — the subclass knows the parent's
internals, change the base class → subclasses break ("fragile base class"); (2)
**LSP violation** — inheriting by "sounds similar", the subtype doesn't substitute;
(3) a single parent is **inflexible** (you can't combine several behaviors).

Composition: instead of `class Car extends Engine` — `class Car { constructor(private
engine: Engine) }`. The car *contains* an engine and delegates; the engine can be
swapped, several parts combined, the coupling is weaker.

## Question 4
`Stack extends Array` inherits the **whole** array interface: `push`, but also
`splice`, `[i]=`, `length=0` — anyone can bypass the stack's rules (LIFO) and reach
into the middle. Encapsulation is broken (the internal storage sticks out), and so
is LSP (Stack doesn't behave as a full Array and vice versa). The right way is
**composition**:
```ts
class Stack<T> {
  #items: T[] = [];
  push(x: T) { this.#items.push(x) }
  pop() { return this.#items.pop() }
  get size() { return this.#items.length }
}
```
The array is hidden inside; only the stack contract is exposed.

## Question 5
Polymorphism — different types respond to one call in their own way, and the caller
doesn't know the concrete type. The OCP link: adding a new type = a new interface
implementation, the **existing code doesn't change**.
```ts
// instead of switch(shape.type) — a method:
interface Shape { area(): number }
function total(shapes: Shape[]) { return shapes.reduce((s, x) => s + x.area(), 0) }
// a new shape → a new class implements Shape; total() untouched
```

## Question 6
`class` is **syntactic sugar over prototypal inheritance**. Under the hood, methods
are put on a prototype object (`Stack.prototype`), and the instance merely
**references** it. On `obj.method()` the engine looks for `method` on the object
itself, doesn't find it, and walks up the **prototype chain**. `extends` links the
prototypes child → parent. So a method is stored **in one place** (the prototype)
rather than copied into every instance — saving memory and giving a single source.

## Question 7
- **`#field`** — a real private slot at the language level: inaccessible from
  outside **at runtime**, access is a syntax error. Actually hides.
- **`_field`** — just a **convention** ("don't touch"), the field is fully
  accessible; hides nothing technically.
- **`private` in TS** — a **compile-time** check only; in the compiled JS the field
  is ordinary and accessible at runtime.
Only **`#field`** actually hides data at runtime.
