# GoF patterns — solutions

> 🌐 Russian version: [05-design-patterns-gof.answers.md](../ru/05-design-patterns-gof.answers.md)

---

## Question 1
- **Creational** — how to create objects, hiding `new` behind an abstraction
  (Factory Method, Abstract Factory, Singleton, Builder, Prototype).
- **Structural** — how to compose objects into larger structures (Adapter,
  Decorator, Facade, Proxy, Composite, Bridge, Flyweight).
- **Behavioral** — how objects interact and distribute responsibilities (Strategy,
  Observer, Command, State, Iterator, Template Method, Chain of Responsibility,
  etc.).

## Question 2
**Decorator.** Wrap the object in another with the same interface, adding behavior:
```ts
interface Notifier { send(msg: string): void }
class BaseNotifier implements Notifier { send(m: string) {} }
class Slack implements Notifier {
  constructor(private n: Notifier) {}
  send(m: string) { this.n.send(m); /* + send to Slack */ }
}
new Slack(new BaseNotifier());   // composes freely, no class per combination
```

## Question 3
All three wrap, but with a different **goal relative to the interface**:
- **Adapter** — *changes* the interface: converts an incompatible one to what the
  client expects.
- **Facade** — *simplifies*: hides many objects/steps behind one simple interface.
- **Decorator** — *keeps* the interface and *adds* behavior: same contract in and
  out, but with an add-on.

Mnemonic: adapter — a "plug converter", facade — a "single button", decorator — a
"wrapper with filling".

## Question 4
**Adapter.** Wrap the foreign logger in your interface:
```ts
interface Logger { log(msg: string): void }
class LibLoggerAdapter implements Logger {
  constructor(private lib: LibLogger) {}
  log(msg: string) { this.lib.write('info', msg); }
}
```
The client keeps calling `logger.log()`, knowing nothing about `write(level, msg)`.

## Question 5
**Observer:** a subject object keeps a list of subscribers and, on change, notifies
them all without knowing who they are or what they do. Decouples the event source
from the reactions.

Out of the box in JS:
- **`EventEmitter`** in Node (`emitter.on/emit`);
- **DOM events** (`addEventListener`);
- **RxJS `Observable`**, **Vue** reactivity, store subscriptions (Redux
  `subscribe`, Pinia). Any "subscribe — you'll be called on change" is Observer.

## Question 6
- **Strategy** — the algorithm is chosen **from outside** (the client/config plugs
  in the strategy) and usually doesn't change on the fly; it varies *how* to do the
  same thing (different sorts, different payment methods).
- **State** — the object **switches its own** behavior as its internal state
  changes (an order: `new → paid → shipped`, and the methods behave differently).
  Structurally similar (behavior behind an interface), but with State the object
  itself drives the transitions, and the point is to model a state machine.

## Question 7
By packing an action into an object with `execute()` (and, say, `undo()`), you get:
1. **undo/redo** — keep a history of executed commands and roll them back;
2. **queue/deferred execution/log** — a command can be queued, serialized,
   replayed, executed later.
A direct function call only gives "do it now"; a command turns the action into
**data** you can manage.

## Question 8
**Singleton = global state with a global access point.** Downsides: a hidden
dependency (a class secretly reaches for the singleton — invisible in the
signature), hard to test (shared state leaks between tests, can't swap a mock),
hidden coupling.

In Node a module is **already** cached — you export an instance and it's one per
process. In Nest the **DI container** makes providers singletons by default and
injects them via the constructor — you get "one instance", but the dependency is
**explicit** and swappable in tests. So a manual Singleton is rarely needed.

## Question 9
- **Strategy** → just a **function parameter** (`arr.sort(compareFn)`, a callback).
- **Singleton** → a **module** (imported once, cached).
- **Command** → a **closure** (`() => doThing(x)`) you can push into an array/queue.
- **Observer** → `EventEmitter`/callbacks are already built in.
- **Factory** → a plain function returning an object.
The JS idiom is often shorter than the "classic" OOP pattern — the textbook pattern
is worth it only when the problem has actually grown into it.
