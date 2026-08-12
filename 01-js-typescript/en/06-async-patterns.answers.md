# Async — solutions

> 🌐 Russian version: [06-async-patterns.answers.md](../ru/06-async-patterns.answers.md)

---

## Question 1
There's one thread. Synchronous waiting would block it entirely: the runtime
wouldn't process other events, the UI would freeze, the server would stop answering
requests. Async lets you **start** a long operation, free the thread, and get the
result later via a callback/promise — the thread stays free for other work.

## Question 2
**pending** (waiting), **fulfilled** (success with a value), **rejected** (error).
The transition from pending happens **exactly once** and irreversibly: either to
fulfilled or to rejected; the state doesn't change after that.

## Question 3
The chain is **flat** (no nesting "pyramid"), because `.then` returns a new promise
and the next `.then` attaches to it. The `.catch` at the end catches an error from
**any** link above: a reject "falls through" down the chain to the nearest error
handler — no need to catch at every step.

## Question 4
`async/await` is a **layer (sugar)** over promises, not a replacement. An async
function **always returns a promise** (a value is wrapped in a resolved promise).
`await p` is under the hood equivalent to `p.then(...)`: it suspends the function,
and the continuation after `await` runs as a **microtask** once `p` resolves.

## Question 5
```ts
const results = await Promise.all(urls.map(url => fetch(url)));
```
All `fetch`es start at once (in parallel), we wait for all. Time ≈ the longest one,
not the sum. (If you need a result even with some errors — `Promise.allSettled`.)

## Question 6
- **`Promise.all`** — all succeed → array of results; any fails → the whole fails
  (fail-fast). Scenario: load all required resources, none is optional.
- **`Promise.allSettled`** — waits for all, returns each one's status, never fails.
  Scenario: send 100 emails and learn which went through.
- **`Promise.race`** — the first to settle (success/error). Scenario: a request
  against a timeout.
- **`Promise.any`** — the first success, ignores errors. Scenario: several mirrors,
  take whichever responds successfully first.

## Question 7
`forEach` **doesn't await** async callbacks: it synchronously starts all the `async`
functions and moves on, so `'all saved'` prints **before** `save` finishes. Fix:
```ts
// sequentially:
for (const item of items) { await save(item); }
// or in parallel:
await Promise.all(items.map(item => save(item)));
console.log('all saved');
```

## Question 8
```
1
4
3
2
```
Synchronously: `1`, `4`. Then microtasks (the promise) — `3` — run before
macrotasks. Then the timer macrotask — `2`. (Micro outranks macro — see event-loop.)
