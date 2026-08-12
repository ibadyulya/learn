# Memory and garbage collection — solutions

> 🌐 Russian version: [09-memory-garbage-collection.answers.md](../ru/09-memory-garbage-collection.answers.md)

---

## Question 1
The criterion is **reachability**: an object is alive as long as there's a chain of
references to it from the roots. **Roots (GC roots)** are the definitely-alive: the
global object, variables and parameters in the current call stack, closed-over
variables of active closures. An object unreachable from the roots is eligible for
collection.

## Question 2
1. **mark** — from the roots, traverse all references and mark every reachable
   object.
2. **sweep** — everything unmarked is considered unreachable, that memory is freed.

## Question 3
**Yes, it's collected.** A modern GC works by reachability from the roots, not by a
reference count. Mutual references form an "island", but since it can't be reached
from the roots, it's all unreachable and gets collected. **Reference counting** would
get it wrong: each object's count = 1 (a reference from its neighbor), it would never
reach zero → the pair would "hang" forever. That's exactly why pure reference counting
was abandoned.

## Question 4
A leak is an object that **stays reachable** (a reference to it is alive) even though
it's logically no longer needed, so the GC can't reclaim it and memory grows. Three
sources: (1) un-removed event listeners / timers; (2) growing global
caches/arrays/`Map`s; (3) detached DOM nodes held by a reference (or closures over
heavy objects).

## Question 5
`cache` is in a closure/module, reachable from a root, and **only grows**: nobody
deletes entries, so all `data` lives forever, even when unneeded. Fix — bound and
clear it: delete stale entries (`delete cache[id]`), add a limit/TTL (LRU), or use a
**`WeakMap`** if the keys are objects (entries go away on their own with the keys).

## Question 6
`Map` holds keys **strongly** — while an entry is in the Map, the key-object is alive,
even if needed nowhere else (a potential leak). `WeakMap` holds keys **weakly**: when
no other (strong) references remain to the key-object, the GC collects it and the
WeakMap entry disappears automatically. It saves you when attaching metadata/cache to
objects with an unknown lifetime (DOM nodes, entities) — no manual cleanup needed.

## Question 7
The leak: the `setInterval` in the constructor isn't stopped in `destroy`, so the
callback (and through it the whole `this` widget) is held forever, even after
"removal". Fix — clear the interval:
```ts
destroy() {
  clearInterval(this.timer);
  // + remove any event listeners
}
```
After `clearInterval` the callback is no longer held by the timer → the widget
becomes unreachable and is collected.
