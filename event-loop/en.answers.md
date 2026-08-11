# Event Loop — task solutions

First try it yourself in [en.tasks.md](./en.tasks.md), then check against these.

> 🌐 Russian version: [ru.answers.md](./ru.answers.md)

---

## Task 1

```
A
D
C
B
```

- Synchronously top to bottom: `A`, `D`. `setTimeout` puts `B` in the macro-queue,
  `.then` puts `C` in the micro-queue.
- Stack empty → drain microtasks → `C`.
- One macrotask → `B`.

---

## Task 2

```
4
3
5
2
1
```

- Synchronously: `4`.
- Micro-phase, priority: **all of nextTick first** (`3`, then `5` — in insertion
  order), **then promises** (`2`).
- Macrotask: `1`.

---

## Task 3

```
timer 1
promise in 1
timer 2
nextTick in 2
```

Both callbacks sit in the **timers** phase queue, but macrotasks are taken **one
at a time**, and after each — a turnstile with microtasks.

- `timer 1` → prints, schedules a promise. Callback ends → turnstile → `promise in 1`.
- `timer 2` → prints, schedules a nextTick. Callback ends → turnstile → `nextTick in 2`.

Key: microtasks wedge in **between** the two timers, even though both are in the
same phase.

---

## Task 4

```
start
1
end
2
promise
3
```

`await` is the same as `.then`: the code after `await` becomes a microtask.

- Synchronously: `start`. `main()` prints `1`, reaches `await null` → suspends,
  the continuation (`2...`) goes into the micro-queue. Control returns.
- `Promise...then` puts `promise` in the micro-queue **after** main's continuation.
- Synchronously: `end`.
- Micro-queue `[main→2, promise]`:
  - main's continuation → `2`, `await` again → new continuation (`3`) to the end of the queue;
  - `promise`;
  - main's continuation → `3`.

---

## Task 5

```
Order is NON-DETERMINISTIC: either "timeout, immediate" or "immediate, timeout".
```

`setTimeout(0)` is clamped to 1ms. At the top level it's a race: whether 1ms has
elapsed by the time we enter the `timers` phase.
- fast start (<1ms) → timers is skipped → `check` → `immediate` first;
- slow start (≥1ms) → `timeout` first.

The right interview answer is **"it depends, it's a race"**, not a specific order.

---

## Task 6

```
nextTick
promise
immediate
timeout
```

**Deterministic**, because we're inside an I/O callback (the `poll` phase).

- The `readFile` callback ended → turnstile: `nextTick`, then `promise`.
- The next phase after `poll` is `check` → `immediate`.
- We only reach `timers` on the next lap → `timeout`.

Inside I/O, `setImmediate` is always before `setTimeout(0)`.

---

## Task 7

```
start
end
nextTick
promise 1
promise 2
nextTick in promise
timeout
```

- Synchronously: `start`, `end`.
- Micro-phase: nextTick first (`nextTick`), then promises.
- `promise 1` prints, schedules `promise 2` (microtask) and `nextTick in promise` (nextTick).
- Subtlety: the engine drains the **entire** promise micro-queue to the end → `promise 2`
  runs **before** `nextTick in promise`. A nextTick added from a promise waits until
  the current promise drain finishes.
- Then the macrotask: `timeout`.

Takeaway: nextTick outranks promises **among those already queued**. But a nextTick
scheduled FROM a promise doesn't interrupt the current promise drain.

---

## Task 8 (boss)

The guaranteed part:

```
=== 1 ===
=== 2 ===
nextTick 1
promise 1
promise in nextTick
nextTick in promise
```

Then a **timers vs check race** (top level!). Two possible tails:

If the timers phase fires first:
```
setTimeout 1
promise in setTimeout
setImmediate 1
```
If the check phase fires first:
```
setImmediate 1
setTimeout 1
promise in setTimeout
```

Walkthrough of the top (deterministic) part:
- Synchronously: `=== 1 ===`, `=== 2 ===`.
- The micro-phase starts with nextTick: `nextTick 1` prints, schedules `promise in nextTick`
  (a microtask — it queues after `promise 1`).
- Promise drain: `promise 1` (schedules `nextTick in promise`), then `promise in nextTick`.
- Promise micro-queue empty → the nextTick added from a promise is processed:
  `nextTick in promise`.
- Then libuv: `setTimeout 1` (and right after it a turnstile → `promise in setTimeout`)
  and `setImmediate 1`. The order of these two groups relative to each other is a race.

`promise in setTimeout` always comes right after `setTimeout 1` (turnstile after the
callback), while `setImmediate 1` has no microtask after it.
