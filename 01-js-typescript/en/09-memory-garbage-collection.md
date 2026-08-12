# Memory and garbage collection

> 🌐 Russian version: [09-memory-garbage-collection.md](../ru/09-memory-garbage-collection.md)

How JS manages memory itself, by what rule it frees objects, and where leaks come
from. Reads as a story.

The through-line of the topic:

> **Memory in JS is managed automatically: allocated when a value is created and
> freed when the object can no longer be **reached** from the "roots". The garbage
> collector works by reachability (mark-and-sweep). A leak isn't "the GC broke" but
> an accidentally held reference to something no longer needed.**

---

## The memory lifecycle

Three steps, the first and third automatic:
1. **allocation** — on creation (`{}`, `[]`, a string, a closure);
2. **use** — reading/writing;
3. **release** — when the value is no longer needed; **the GC handles this**, memory
   isn't freed by hand.

The question is **how the GC knows "no longer needed"**.

---

## Reachability and roots

The criterion isn't "the object is unused" (that's undecidable) but
**reachability**: an object is alive if it can be **reached by references from the
roots (GC roots)**.

Roots are the definitely-alive:
- the global object (`globalThis`/`window`);
- variables and parameters **in the current call stack**;
- closed-over variables of live closures.

```
root (stack, globals)
  └─► userService ──► cache ──► user   ← all reachable, alive
```
As soon as an object has **no chain of references left from a root**, it becomes
unreachable and eligible for collection. Even if objects reference **each other**,
but their "island" is detached from the roots, the whole island is collected (so a
modern GC isn't afraid of cyclic references).

---

## Mark-and-sweep — how the GC works

The core algorithm in V8:
1. **mark** — go from the roots through all references and mark everything
   reachable.
2. **sweep** — everything unmarked = unreachable → the memory is freed.

In practice V8 does this **generationally**: most objects are short-lived, so the
"young generation" is cleaned often and fast, the "old" one rarely. Plus
incrementally/concurrently, so as not to freeze the thread for long. The details
don't matter — the principle does: **the reachable lives, the unreachable is
collected.**

---

## What a memory leak is

A leak is an object that **stays reachable** (a reference to it is alive) even though
it's logically no longer needed. The GC can't reclaim it, memory grows. So leaks in
JS are **forgotten references**.

Typical sources:
- **Forgotten timers/intervals** — a `setInterval` nobody `clearInterval`s holds the
  data closed over in the callback forever.
- **Un-removed event listeners** — `addEventListener` without `removeEventListener`;
  the callback holds everything it closed over (see [closures-scope](./04-closures-scope.md)).
- **Growing global structures** — a cache/array/`Map` in a global where everything is
  put and never cleared: reachable from a root → not collected.
- **Detached DOM nodes** — you removed an element from the page but hold a reference
  to it in a variable → the node and its subtree aren't collected.
- **Closures over heavy things** — a callback that accidentally closed over a large
  object and outlived its usefulness.

Symptom: memory grows monotonically over time/load and doesn't drop back. Investigated
via heap snapshots in DevTools (comparing snapshots — what accumulates).

---

## Weak references: WeakMap / WeakSet / WeakRef

An ordinary reference **holds** an object. A **weak** one doesn't: if only weak
references remain to an object, the GC collects it.
- **`WeakMap`** (keys are objects) / **`WeakSet`** — entries vanish automatically
  when the key-object is no longer strongly held anywhere. Ideal for a cache/metadata
  tied to an object, with no leak risk.
- **`WeakRef`** — an explicit weak reference to an object (rarely needed, for advanced
  caches/finalization).

```ts
const meta = new WeakMap();
meta.set(user, { lastSeen: Date.now() });   // when user dies, the entry goes on its own
```

---

## Interview phrasing

> "Memory in JS is managed automatically: the GC frees objects by **reachability** —
> alive is whoever has a chain of references from the roots (stack, globals,
> closures). The algorithm is mark-and-sweep (mark the reachable from the roots,
> sweep the rest), generational in V8. Cyclic references aren't a problem: an island
> detached from the roots is collected whole. A leak is a **forgotten reference**: an
> un-removed listener/timer, a growing global cache, a detached DOM node, a closure
> over something heavy. Fixed by cleaning up references (`removeEventListener`,
> `clearInterval`) and weak collections `WeakMap`/`WeakSet`, which don't prevent
> collection."

---

See [09-memory-garbage-collection.tasks.md](./09-memory-garbage-collection.tasks.md)
— tasks. Solutions in [09-memory-garbage-collection.answers.md](./09-memory-garbage-collection.answers.md).
