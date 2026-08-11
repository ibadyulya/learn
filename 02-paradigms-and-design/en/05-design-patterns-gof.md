# Design patterns (Gang of Four)

> 🌐 Russian version: [05-design-patterns-gof.md](../ru/05-design-patterns-gof.md)

The 23 classic patterns from the "Gang of Four" book. Reads as a story: a pattern
is **not code to copy, but a named solution to a recurring design problem**. The
interview value isn't memorizing 23 of them, but recognizing the problem and
naming the solution (and not forcing a pattern where it isn't needed).

The through-line of the whole topic:

> **GoF splits patterns into three families by the kind of problem: how to
> *create* objects (creational), how to *compose objects into structures*
> (structural), how objects *interact and distribute responsibilities*
> (behavioral). Knowing the family, you're already halfway to naming the pattern.**

Below — not all 23, but the ~10 that actually get asked and that you see daily in
JS/Node/React/Nest.

---

## Creational — how to create objects

The family's problem: a direct `new ConcreteThing()` nails the code to a concrete
class. Creational patterns hide creation behind an abstraction.

**Factory Method.** Instead of `new` — a method that decides which object to
create. The client asks "give me a suitable object" without knowing the concrete
class.
```ts
interface Button { render(): string }
class HtmlButton implements Button { render() { return '<button>' } }
class IosButton  implements Button { render() { return 'UIButton' } }

function createButton(os: 'web' | 'ios'): Button {
  return os === 'ios' ? new IosButton() : new HtmlButton();
}
```
Why: adding a new platform = a new class + a branch in the factory, while all the
client code (`btn.render()`) stays untouched — direct OCP.

**Abstract Factory** — a factory of families of related objects (button + checkbox
+ window of one style). Same principle, but creates a consistent *set*.

**Singleton.** Guarantees **one** instance for the whole app and a global access
point (config, connection pool, logger).
```ts
class Config {
  private static instance: Config;
  private constructor() {}
  static get(): Config { return this.instance ??= new Config(); }
}
```
Careful: often an **anti-pattern** — it's global state, hides dependencies, hurts
tests. In JS a module is itself a singleton (it's cached), and in Nest the DI
container gives you singletons (the default scope) — you rarely need it by hand.

**Builder.** Step-by-step assembly of a complex object, when a constructor with
ten arguments turns into mush.
```ts
new QueryBuilder().select('id').from('users').where('age > 18').build();
```
Why: readable, parts are optional, order is flexible. Seen in query builders,
`supertest`, configuration.

---

## Structural — how to compose objects into structures

The family's problem: how to connect objects/classes into larger structures
without turning everything into a rigid monolith.

**Adapter.** Wraps an incompatible interface in the one the client expects. A
"plug adapter".
```ts
// the client expects Logger.log(); a third-party lib gives winston.info()
class WinstonAdapter implements Logger {
  constructor(private w: Winston) {}
  log(msg: string) { this.w.info(msg); }
}
```
Why: make foreign code fit your contract without rewriting either side.

**Decorator.** Adds behavior to an object by **wrapping** it, keeping the same
interface. An alternative to a subclass explosion.
```ts
interface Coffee { cost(): number }
class Espresso implements Coffee { cost() { return 2 } }
class WithMilk implements Coffee {
  constructor(private c: Coffee) {}
  cost() { return this.c.cost() + 0.5 }   // wraps and augments
}
new WithMilk(new Espresso()).cost(); // 2.5
```
Why: combine behaviors dynamically (`WithMilk(WithSugar(Espresso))`) without a
class per combination. Seen in Express middleware, Nest interceptors, TS
decorators.

**Facade.** One simple interface over a complex subsystem. Hides a pile of details
behind one method.
```ts
class OrderFacade {
  place(cart: Cart) { this.stock.reserve(cart); this.pay.charge(cart); this.ship.schedule(cart); }
}
```
Why: the client calls `facade.place()` without knowing about stock/payment/
shipping. Lowers coupling.

**Proxy.** A stand-in object with the same interface, controlling access to the
real one: lazy init, cache, permissions, logging.
```ts
class CachingApiProxy implements Api {
  private cache = new Map();
  constructor(private real: Api) {}
  get(id: string) { return this.cache.get(id) ?? this.cache.set(id, this.real.get(id)).get(id); }
}
```
Why: add access control/cache without changing the client or the real object. JS
even has a built-in `Proxy`.

> Adapter vs Facade vs Decorator (a frequent question): **Adapter changes the
> interface** (incompatible → needed), **Facade simplifies** (many → one),
> **Decorator keeps the interface and adds behavior** (same → same + feature).

---

## Behavioral — how objects interact

The family's problem: how to distribute responsibilities and organize
communication between objects.

**Strategy.** A family of interchangeable algorithms behind a common interface;
the client picks one at runtime. (Already came up in [SOLID](./03-solid.md) as a
way to achieve OCP.)
```ts
interface SortStrategy { sort(a: number[]): number[] }
class QuickSort implements SortStrategy { sort(a) { /* ... */ return a } }
class Sorter { constructor(private s: SortStrategy) {} run(a: number[]) { return this.s.sort(a) } }
```
Why: instead of `if (type===...)` — a substitutable algorithm object. A new
algorithm = a new class.

**Observer.** A "subject" notifies subscribers of changes without knowing who they
are. The basis of reactivity and events.
```ts
class Subject {
  private subs: ((v: number) => void)[] = [];
  subscribe(fn: (v: number) => void) { this.subs.push(fn) }
  emit(v: number) { this.subs.forEach(fn => fn(v)) }
}
```
Why: decouple the event source from the reactions. Seen everywhere: `EventEmitter`
in Node, RxJS, Vue reactivity, `addEventListener`, store subscriptions.

**Command.** Packs an action into an object: it can be passed around, queued,
logged, undone.
```ts
interface Command { execute(): void; undo(): void }
class AddText implements Command {
  constructor(private doc: Doc, private text: string) {}
  execute() { this.doc.append(this.text) }
  undo()    { this.doc.removeLast(this.text.length) }
}
```
Why: undo/redo, task queues, transactionality. The action becomes data.

More behavioral ones (name them if asked): **State** (an object changes behavior
as its internal state changes — like Strategy, but it switches itself),
**Iterator** (traverse a collection without exposing its internals — in JS it's
`Symbol.iterator`), **Template Method** (the skeleton of an algorithm in the base
class, steps in subclasses), **Chain of Responsibility** (a request travels a
chain of handlers — like middleware).

---

## The main caution

Patterns are a **vocabulary**, not a goal. A pattern forced in (AbstractFactory
where a function would do) is over-engineering. The right move: **problem first,
then — is there a named solution for it**. And half the "patterns" in JS are
expressed more simply: a strategy is just a function parameter, a singleton is a
module, a command is a closure.

---

## Interview phrasing

> "GoF patterns are named solutions to typical design problems, split into three
> families. Creational (Factory Method, Singleton, Builder) hide object creation
> behind an abstraction. Structural (Adapter, Decorator, Facade, Proxy) compose
> objects into structures: adapter changes the interface, facade simplifies,
> decorator wraps and adds behavior, proxy controls access. Behavioral (Strategy,
> Observer, Command) organize interaction: strategy — swappable algorithms,
> observer — subscriptions/events (EventEmitter, RxJS), command — an action as an
> object with undo. In JS many are expressed more simply — a function, a module, a
> closure — and a pattern is only needed when the problem actually repeats."

---

See [05-design-patterns-gof.tasks.md](./05-design-patterns-gof.tasks.md) — tasks.
Solutions in [05-design-patterns-gof.answers.md](./05-design-patterns-gof.answers.md).
