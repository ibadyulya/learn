# Closures and scope

> 🌐 Russian version: [04-closures-scope.md](../ru/04-closures-scope.md)

One of the most frequent interview topics and the source of half of JS's "magic".
Reads as a story: it all grows from one rule about where a function gets its
variables.

The through-line of the topic:

> **A function takes variables from where it was DECLARED, not where it's called
> (lexical scope). A closure is the direct consequence: the function "remembers"
> that environment and keeps access to it even after the outer function has
> finished.**

---

## Lexical scope — "where declared is where we take from"

Scope is where a variable is accessible. In JS it's **lexical**: determined by the
**place in the code where the function is written**, not where it's called from.

```ts
const x = 'outer';
function outer() {
  const x = 'inner';
  function inner() { console.log(x); }   // takes x from the place of declaration
  return inner;
}
outer()();   // 'inner' — even though called from outside
```
Not finding a variable in the current function, the engine walks **up the scope
chain** — to the function this one is declared inside, and so on to the global one.
Key: the chain is built by **nesting in the code**, not by the call stack.

---

## What a closure is

**A closure is a function together with its "remembered" environment** (variables
from the scope where it was created). The function carries a reference to its outer
variables — and they live as long as the function does, even if the function that
produced them has already finished.

```ts
function makeCounter() {
  let count = 0;                 // a local variable
  return () => ++count;          // closes over count
}
const next = makeCounter();
next(); // 1
next(); // 2   — count didn't vanish after makeCounter returned; it lives in the closure
```
`makeCounter` finished, its stack frame is "popped" — but `count` isn't garbage
collected, because the returned function **references** it. That's a closure:
private, persistent state.

Two independent counters don't share state — each has its own closure:
```ts
const a = makeCounter(), b = makeCounter();
a(); a(); // 2
b();      // 1 — its own count
```

---

## The classic trap: `var` in a loop

A favorite question. Why does this print `3, 3, 3`, not `0, 1, 2`?
```ts
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
// 3, 3, 3
```
`var` is **one** variable for the whole function (function-scoped). All three
callbacks closed over the **same** `i`; by the time the timers fired, the loop had
long finished and `i === 3`. The cure — `let`, which has **its own binding per
iteration** (block-scoped):
```ts
for (let i = 0; i < 3; i++) setTimeout(() => console.log(i), 0);
// 0, 1, 2 — each callback closed over its own copy of i
```
This same `var`/`let` difference is the crux: `let`/`const` live in the `{}` block,
`var` lives in the whole function and hoists to its top.

---

## Why closures are useful in practice

- **Privacy / the module pattern** — hide state, expose only methods:
  ```ts
  const counter = (() => { let c = 0; return { inc: () => ++c, get: () => c }; })();
  ```
  From outside you can't reach `c` — only through methods. (Before `#private`
  fields this was the main way to encapsulate in JS.)
- **Function factories / currying** — `const add = a => b => a + b` (closes over `a`).
- **Callbacks and handlers** — a callback remembers variables from where it was set.
- **Memoization** — a cache lives in the closure between calls.

---

## The flip side: memory leaks

Since a closure **holds** outer variables from garbage collection, it's easy to
accidentally keep something big alive: an event handler that closed over a heavy
object and wasn't removed will prevent that object from being collected. The rule —
remove listeners/timers, don't close over more than needed. (More in
[memory-garbage-collection](./09-memory-garbage-collection.md).)

---

## Interview phrasing

> "JS has lexical scope: a function takes variables from where it's declared, not
> where it's called; not found — it walks up the scope chain. A closure is a
> function together with that remembered environment: it keeps access to outer
> variables even after the producing function finished, because it holds a
> reference to them. Hence private state (the module pattern, counters, currying).
> The classic trap is `var` in a loop: one function-scoped variable for all
> iterations gives 3,3,3; `let` creates a per-iteration binding and gives 0,1,2.
> Closures can also keep memory alive — a source of leaks."

---

See [04-closures-scope.tasks.md](./04-closures-scope.tasks.md) — tasks. Solutions
in [04-closures-scope.answers.md](./04-closures-scope.answers.md).
