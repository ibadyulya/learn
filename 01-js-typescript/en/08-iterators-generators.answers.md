# Iterators and generators — solutions

> 🌐 Russian version: [08-iterators-generators.answers.md](../ru/08-iterators-generators.answers.md)

---

## Question 1
- **Iterable** — an object with a `[Symbol.iterator]()` method returning an iterator
  ("I can be traversed").
- **Iterator** — an object with a `next()` method that on each call returns
  `{ value, done }`: `value` is the current element, `done: true` when it's over.
An iterable hands out an iterator; the iterator yields elements one by one.

## Question 2
`for...of`, spread `[...x]`, array destructuring `const [a, b] = x`, `Array.from(x)`,
`new Set(x)` / `new Map(x)`, `Promise.all(x)`, `yield*`. They all internally pull
`Symbol.iterator` and drive `next()`.

## Question 3
An array has a built-in `Symbol.iterator`, a plain object does **not**, so `for...of`
on `{}` throws `is not iterable`. To iterate an object: `for...in` (over keys,
careful with inherited ones), or via iterable "views" — `Object.keys(o)`,
`Object.values(o)`, `Object.entries(o)` + `for...of`.

## Question 4
`function*` declares a generator, `yield` is the point where execution **freezes**,
handing out a value. Unlike an ordinary function (runs to `return` in one go), a
generator runs **in portions**: each `next()` continues from the last `yield`,
preserving local state. The function is effectively paused.

## Question 5
```ts
function* range(from: number, to: number, step = 1) {
  for (let i = from; i <= to; i += step) yield i;
}
[...range(0, 10, 2)];   // [0, 2, 4, 6, 8, 10]
```

## Question 6
```ts
function* naturals() { let n = 1; while (true) yield n++; }
```
`while (true)` doesn't hang the runtime because the generator is **lazy**: the body
runs only on `next()` and **stops at `yield`** until the next request. As long as
you take a finite number (`g.next()` N times or `for...of` with `break`), exactly N
elements are computed, not infinity.

## Question 7
```ts
const range = {
  from: 1, to: 3,
  *[Symbol.iterator]() {
    for (let i = this.from; i <= this.to; i++) yield i;
  }
};
[...range];   // [1, 2, 3]
```
A generator in `[Symbol.iterator]` is shorter than a manual `next()` and implements
the protocol itself.

## Question 8
**Async iterators** are needed when elements arrive **asynchronously** (reading a
stream/file in chunks, API pages, messages from a queue). Declared with
`Symbol.asyncIterator` / `async function*` with `yield`, each step is a promise.
Iterated with the **`for await (const x of source)`** loop. Node streams are
async-iterable.
