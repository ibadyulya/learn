# Functional programming — solutions

> 🌐 Russian version: [02-functional-programming.answers.md](../ru/02-functional-programming.answers.md)

---

## Question 1
A pure function: (1) **deterministic** — the same input always gives the same
output; (2) **no side effects** — changes nothing external and depends on nothing
external.

Consequences: **testability** — input → check output, no mocks or state setup
needed; **cacheability** — since output depends only on input, the result can be
remembered (memoization); **parallelism** — no shared mutable state means no races,
so functions can run in parallel safely.

## Question 2
- `a` — **pure**: only input → output.
- `b` — **impure**: `Array.sort()` mutates the original array (a side effect on the
  argument).
- `c` — **impure**: `Date.now()` is non-deterministic (different output for the
  same "input").
- `d` — **pure**: `[...arr]` copies, the copy is sorted, the original is intact.

## Question 3
React/Redux detect "something changed" by **reference comparison** (`prev ===
next`), which is cheap. If you mutate `state.items.push(x)`, the object reference
**stays the same** → React/Redux conclude "nothing changed" → no re-render/update.
You must create a **new** object/array (`[...state.items, x]`), then the reference
changes and the change is noticed. Mutation breaks the whole change-detection model.

## Question 4
```ts
const result = users
  .filter(u => u.active)
  .map(u => u.name.toUpperCase());
```
No manual state or mutation — the "what", not the "how".

## Question 5
- **Currying** — turn `f(a, b, c)` into the chain `f(a)(b)(c)`.
- **Partial application** — fix some arguments, getting a new function.
```ts
const log = (level: string) => (msg: string) => console.log(`[${level}] ${msg}`);
const error = log('ERROR');   // a specialization
error('disk full');           // [ERROR] disk full
```
Handy for reusable specializations and plugging into `map`/handlers without
wrappers.

## Question 6
Effects are inevitable — without I/O a program does nothing. FP doesn't **forbid**
them, it **localizes** them: keeps the logic core pure and pushes effects (network,
DB, console) into a thin layer at the **edges** of the app. That keeps the main
logic testable and predictable, with the "dirt" gathered in predictable places.

## Question 7
```ts
const compose = (...fns: Function[]) => (x: any) =>
  fns.reduceRight((acc, fn) => fn(acc), x);

// check:
const f = (n: number) => n + 1;
const g = (n: number) => n * 2;
const h = (n: number) => n - 3;
compose(f, g, h)(10);   // f(g(h(10))) = f(g(7)) = f(14) = 15
```
`reduceRight` goes right to left — exactly the nesting order of `f(g(h(x)))`.
