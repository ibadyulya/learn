# Async: promises, async/await

> 🌐 Russian version: [06-async-patterns.md](../ru/06-async-patterns.md)

How JS handles operations that **wait** (network, disk, timers) without blocking
its single thread. Reads as a story — from callbacks to async/await. For the
scheduling mechanics (micro/macrotasks) see [event-loop](./01-event-loop.md); here
it's about the tools on top of it.

The through-line of the topic:

> **JS is single-threaded, so you can't "wait" synchronously — you'd freeze
> everything. Instead of waiting, a task goes to the background and the result
> arrives later via a callback. A promise is an object promising a future value;
> async/await is syntax over promises that makes async code look synchronous. All
> of it is a layer over the event loop.**

---

## Why async at all

There's one thread. If you synchronously wait a second for a network response, the
whole runtime stalls (doesn't render, doesn't respond). The solution: **don't wait
in place** — say "start the operation, and when it's done, call this", and move on.
"This" is a callback.

---

## Callbacks and callback hell

The first approach — pass a function to be called when ready:
```ts
readFile('a.txt', (err, data) => {
  if (err) return handle(err);
  parse(data, (err, parsed) => { /* ... */ });   // nesting grows
});
```
Problems: the **"pyramid of doom"** (nesting per step), manual error handling in
each callback, hard to combine. Hence — promises.

---

## Promises — a promise object

A **Promise** is an object representing a value that **isn't there yet** but will
be (or will be an error). Three states:
- **pending** — waiting;
- **fulfilled** — resolved successfully with a value;
- **rejected** — rejected with an error.

It transitions from pending **exactly once** and irreversibly. Work with the result
via `.then` (success), `.catch` (error), `.finally` (either way):
```ts
fetch('/api')
  .then(res => res.json())      // then returns a new promise → a chain
  .then(data => use(data))
  .catch(err => handle(err))    // catches an error from ANY then above
  .finally(() => stopLoader());
```
Key: **`.then` returns a new promise**, so the chain "unrolls" the callback nesting
into a flat sequence, and one `.catch` at the end catches an error from any link
(the error "falls through" down the chain).

---

## async/await — syntax over promises

`async/await` doesn't replace promises — it's **sugar** over them, making the code
visually synchronous:
```ts
async function load() {
  try {
    const res = await fetch('/api');   // await "waits" for the promise without blocking the thread
    const data = await res.json();
    return use(data);
  } catch (err) {
    handle(err);                        // ordinary try/catch instead of .catch
  }
}
```
- an `async` function **always returns a promise**.
- `await p` suspends the function until `p` resolves; **under the hood it's the same
  `.then`** — the code after `await` goes into a microtask (see event-loop).
- errors are caught with ordinary `try/catch`.

Readability is higher, but remember: `await` is `.then`, not "stop the world".

---

## Concurrency: parallel vs sequential

A common mistake — `await` in a loop where operations are independent:
```ts
// slow: wait for each in turn (sum of the times)
for (const url of urls) results.push(await fetch(url));

// fast: start them all at once, wait for all (the longest one's time)
const results = await Promise.all(urls.map(u => fetch(u)));
```
Combinators:
- **`Promise.all`** — waits for all; if **any** fails, the whole fails (fail-fast).
- **`Promise.allSettled`** — waits for all, returns both successes and errors (never
  fails) — when you need a result for each.
- **`Promise.race`** — the first to **settle** (success or error) — timeouts.
- **`Promise.any`** — the first **success** (ignores errors while there's hope).

---

## Common pitfalls

- **Forgot `await`** — you work with a promise instead of the value
  (`[object Promise]`, "always truthy" conditions).
- **Sequential `await` instead of `Promise.all`** — independent operations run in a
  chain, times add up.
- **Unhandled rejection** — a promise failed, no `.catch`/`try` → `unhandledRejection`.
- **`forEach` with `async`** — `arr.forEach(async ...)` doesn't await the callbacks;
  for sequence use `for...of` with `await`, for parallelism `map` + `Promise.all`.

---

## Interview phrasing

> "JS is single-threaded, so waiting operations are done asynchronously — a task
> goes to the background, the result arrives via a callback. A promise is a
> future-value object with states pending → fulfilled/rejected, transitioning once;
> `.then` returns a new promise, so chains are flat, and one `.catch` catches an
> error from any link. async/await is sugar over promises: an async function always
> returns a promise, `await` is `.then` under the hood (the code after goes into a
> microtask), errors via ordinary try/catch. For independent tasks use `Promise.all`
> (or allSettled), not `await` in a loop, or the times add up."

---

See [06-async-patterns.tasks.md](./06-async-patterns.tasks.md) — tasks. Solutions
in [06-async-patterns.answers.md](./06-async-patterns.answers.md).
