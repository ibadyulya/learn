# Iterators and generators

> 🌐 Russian version: [08-iterators-generators.md](../ru/08-iterators-generators.md)

How JS's unified iteration mechanism works and how generators give lazy and even
infinite sequences. Reads as a story.

The through-line of the topic:

> **Iterability is a contract "how to traverse me": an object with a
> `Symbol.iterator` method can hand out elements one by one, and on this contract
> `for...of`, spread and destructuring work. A generator is a function that can
> **pause and resume** (`yield`), automatically implementing that contract; hence
> laziness.**

---

## The iteration protocol

Two levels of the contract:
- **Iterable** — an object with a `[Symbol.iterator]()` method that returns an
  **iterator**.
- **Iterator** — an object with a `next()` method returning `{ value, done }`:
  `value` is the next element, `done` is whether the sequence has ended.

```ts
const it = ['a', 'b'][Symbol.iterator]();
it.next();   // { value: 'a', done: false }
it.next();   // { value: 'b', done: false }
it.next();   // { value: undefined, done: true }
```
It's exactly this protocol that's "pulled" under the hood when you write `for...of`,
`[...arr]`, `const [a, b] = ...`, `Array.from`, `new Set(iterable)`. Built-in
iterables: arrays, strings, `Map`, `Set`, `arguments`, NodeList. A plain object `{}`
is **not** (`for...of` on it throws; iterate via `Object.entries`).

---

## Your own iterable

It's enough to implement `Symbol.iterator`:
```ts
const range = {
  from: 1, to: 3,
  [Symbol.iterator]() {
    let cur = this.from; const last = this.to;
    return { next: () => cur <= last ? { value: cur++, done: false } : { value: undefined, done: true } };
  }
};
[...range];   // [1, 2, 3]  — now for...of, spread, destructuring work
```
A manual iterator is verbose — generators simplify it.

---

## Generators — pausable functions

`function*` creates a **generator**: a function that on `yield` **freezes**, hands
out a value, and waits for the next `next()`, continuing from the same spot.
```ts
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
const g = gen();
g.next();   // { value: 1, done: false }
g.next();   // { value: 2, done: false }
[...gen()]; // [1, 2, 3] — a generator is itself iterable
```
Key: the code between `yield`s runs **in portions, on demand**. The function doesn't
run to the end — it's "paused", preserving all local state (like a live closure with
a resume point).

The same `range`, shorter:
```ts
function* range(from: number, to: number) {
  for (let i = from; i <= to; i++) yield i;
}
[...range(1, 3)];   // [1, 2, 3]
```

---

## Laziness and infinite sequences

Since values are computed on demand, you can describe an **infinite** sequence — it
won't loop the runtime forever, as long as you take a finite number:
```ts
function* naturals() { let n = 1; while (true) yield n++; }
const g = naturals();
g.next().value; // 1
g.next().value; // 2   — computes exactly as much as asked
```
That's the benefit of laziness: don't materialize the whole list in memory, generate
an element at the moment of need (stream processing, pagination, pipelines).

---

## Async iterators (briefly)

For streams where elements arrive asynchronously (reading a file in chunks, API
pages), there are **async iterators**: `Symbol.asyncIterator`, `async function*`
with `yield`, and iteration via `for await (const x of stream)`. Each step returns a
promise. Node streams are async-iterable.

---

## Interview phrasing

> "Iterability in JS is a protocol: an object with `Symbol.iterator` returning an
> iterator with `next()` → `{value, done}`. On it work `for...of`, spread,
> destructuring, `Array.from`; arrays/strings/Map/Set are iterable, a plain object
> isn't. A generator (`function*`) is a function that pauses and resumes on `yield`
> preserving state, automatically implementing the iterator protocol. Hence
> laziness: values are computed on demand, you can describe an infinite sequence and
> not hold the whole list in memory. For async streams — `async function*` and
> `for await...of`."

---

See [08-iterators-generators.tasks.md](./08-iterators-generators.tasks.md) — tasks.
Solutions in [08-iterators-generators.answers.md](./08-iterators-generators.answers.md).
