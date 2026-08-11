# Functional programming

> 🌐 Russian version: [02-functional-programming.md](../ru/02-functional-programming.md)

A paradigm where a program is assembled from **pure functions** and **immutable
data**. Reads as a story: from one property — predictability — everything else
grows.

The through-line of the topic:

> **FP is built on one requirement: a function must be pure — the same input always
> gives the same output, with no hidden effects. From this predictability grows
> everything: composition, easy testing, safe concurrency, declarativeness. Side
> effects aren't forbidden — they're pushed to the edges of the program.**

---

## 1. Pure functions — the heart of it all

A **pure function** satisfies two conditions:
1. **Determinism** — same input → same output, always.
2. **No side effects** — changes nothing outside (globals, arguments, DOM, DB,
   console) and depends on nothing external.

```ts
const add = (a: number, b: number) => a + b;        // pure: only input → output
let total = 0;
const addImpure = (x: number) => { total += x; };    // impure: mutates the outside
```
Why it's gold: a pure function can be **evaluated in your head** (no need to track
the state of the world), **tested without mocks** (input → check output),
**cached** (memoization), **parallelized safely** (nothing shared to break).
Predictability = controllability.

---

## 2. Immutability — don't mutate, create new

Data isn't mutated: instead of changing an object/array, a **new version** is
created.
```ts
// bad (mutation)
const addItem = (cart: Item[], x: Item) => { cart.push(x); return cart; };
// good (new array)
const addItem2 = (cart: Item[], x: Item) => [...cart, x];
```
Why: if data doesn't change under your feet, a whole class of "someone silently
changed my array" bugs disappears. React/Redux reactivity rests on immutability
(reference comparison: a new object = a change happened). Tools: spread,
`map/filter`, `Object.freeze`, immer.

---

## 3. Side effects — not removed, but localized

A program with no effects is useless (you do need to read the DB, send responses,
write to the console). FP doesn't forbid effects — it **pushes them to the edges**:
the logic core is pure, and effects (I/O, network) are a thin layer outside. That
separates the testable core from the "dirty" edges.

---

## 4. Functions are first-class citizens

In JS functions are ordinary values: put in variables, passed as arguments,
returned. Hence **higher-order functions** (HOFs) — they take or return functions:
```ts
[1,2,3].map(x => x * 2);              // map is a HOF, takes a function
const multiplier = (n: number) => (x: number) => x * n;   // returns a function
```
`map/filter/reduce` are FP's workhorses in JS: instead of a manual loop with
mutation — a declarative "what to do with each element".

---

## 5. Composition and currying

**Composition** — build a big function from small ones: one's output → the next's
input.
```ts
const compose = (f, g) => (x) => f(g(x));
const shout = compose((s: string) => s + '!', (s: string) => s.toUpperCase());
shout('hi');   // 'HI!'
```
**Currying** — turn `f(a, b)` into `f(a)(b)`, to fix some arguments (partial
application) and combine conveniently:
```ts
const add = (a: number) => (b: number) => a + b;
const add10 = add(10);   // a reusable specialization
```
Why: small pure pieces + composition = complex behavior without shared state.

---

## 6. Declarative, not imperative

Imperative describes **how** (steps, loops, mutations); declarative — **what** (the
result).
```ts
// imperative
let evens = []; for (const n of nums) if (n % 2 === 0) evens.push(n);
// declarative
const evens2 = nums.filter(n => n % 2 === 0);
```
FP leans declarative — less manual state management, less room for error.

---

## FP and OOP — not enemies

A false dichotomy. Real TS/Node code is usually **mixed**: modular structure and
boundaries from OOP/DDD, and inside — pure functions, `map/filter/reduce`,
immutability. React is a vivid example: components as pure functions of props,
immutable state. Take the strong part of each paradigm: boundary encapsulation from
OOP, core predictability from FP.

---

## Interview phrasing

> "FP is programming with pure functions over immutable data. A pure function is
> deterministic and side-effect-free, so it's easy to test, cache, and
> parallelize. Immutability removes 'data changed under my feet' bugs and underpins
> React/Redux reactivity. Effects aren't forbidden — they're pushed to the edges,
> the core stays pure. Functions are first-class, hence HOFs and
> `map/filter/reduce`, composition and currying. In practice FP and OOP are
> combined: boundaries from OOP, a predictable core from FP."

---

See [02-functional-programming.tasks.md](./02-functional-programming.tasks.md) —
tasks. Solutions in [02-functional-programming.answers.md](./02-functional-programming.answers.md).
