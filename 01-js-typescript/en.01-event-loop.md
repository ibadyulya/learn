# Event Loop (Node.js) — conspect

Prep material for a senior fullstack JS interview. Topic: how Node decides what
to run next.

> 🌐 Russian version: [ru.01-event-loop.md](./ru.01-event-loop.md)

## 1. Four participants

The confusion almost always comes from mixing them up. There are four:

| Participant | What it is | Priority |
|---|---|---|
| **Call stack** | where synchronous code runs right now | while non-empty — nothing else runs |
| **Macrotask queue** | callbacks from `setTimeout`, `setInterval`, `setImmediate`, I/O | low |
| **Microtask queue** | `Promise.then/catch/finally`, continuation after `await`, `queueMicrotask` | high |
| **`process.nextTick` queue** | Node-only, a separate queue | even higher than promises |

Scheduling mechanics on the `setTimeout(cb, 0)` example: the timer counts down
**in the background** (in the browser — Web API, in Node — libuv/C++), and when
ready — **puts `cb` into the queue**. The callback itself just waits its turn.

## 2. The loop algorithm (memorize verbatim)

When the call stack is empty:

```
1. Drain the ENTIRE microtask queue (in Node: nextTick first, then promises) — to the bottom.
2. Take EXACTLY ONE macrotask, run it to completion.
3. Drain the ENTIRE microtask queue again.
4. Take the next ONE macrotask.
5. Repeat.
```

Two keys:
- Microtasks are drained **fully** (including those added during the drain).
- Macrotasks are taken **one at a time**, and between them the microtasks are cleared again.

Hence: infinite recursion in `nextTick`/promises **freezes** the event loop —
it never gets to the macrotasks.

## 3. The libuv phase carousel

In Node the macrotasks live not in one queue but in several — by phase. libuv
runs them in a fixed circle, each phase with its own queue:

```
timers        → setTimeout / setInterval (those whose time expired)
pending       → deferred system callbacks
idle/prepare  → internal
poll          → I/O callbacks (files, network); here the loop "parks" and waits
check         → setImmediate
close         → 'close' events (socket.on('close'))
```

For an interview it's enough to know three: **timers**, **poll**, **check**.

## 4. Where microtasks and nextTick live

**They are NOT on the phase carousel.** Phases are the libuv level (C). The
nextTick and promise queues are the Node/V8 level (JS), on top. Rule:

> After EVERY callback (and between phases) Node drains the micro-queues:
> first all of nextTick, then all promises.

So they aren't a "spot on the carousel" but a **turnstile between every step**.
Wherever a callback runs — right after it, nextTick + promises are cleared, and
only then does the loop move on.

The full priority order:
```
1) synchronous code on the stack   — while the stack isn't empty, nothing else
2) process.nextTick queue          — drain fully
3) Promise microtask queue         — drain fully
4) one macrotask from the current libuv phase → and straight back to step 2
```

## 5. The setTimeout(0) vs setImmediate race

Key fact: **`setTimeout(fn, 0)` in Node is clamped to `setTimeout(fn, 1)`** —
a minimum of 1 millisecond.

**At the top level of a module the order is NON-DETERMINISTIC:**
- if process startup was fast (< 1ms) → the timer isn't ripe → the timers phase
  is skipped → we reach check → `setImmediate` first;
- if startup was slow (≥ 1ms) → the timer is ready in the timers phase →
  `setTimeout` first.

Depends on startup speed → "on the hardware". This is the "gotcha moment".

**Inside an I/O callback the order is GUARANTEED — `setImmediate` first:**
the I/O callback runs in the `poll` phase, and `check` (setImmediate) comes
right after it; the loop only reaches `timers` on the next lap.

```js
const fs = require('fs');
fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
});
// ALWAYS: immediate, timeout
```

## 6. setImmediate vs process.nextTick — the names lie

- `process.nextTick` fires **earlier** (micro-queue, before the loop moves).
- `setImmediate` — **later** (the check phase).

Mnemonic: "`nextTick` — not on the next tick, but right now; `setImmediate` —
not immediately, but on the check phase".

## 7. Interview phrasing

> "At the top level the order of `setTimeout(0)` and `setImmediate` is
> non-deterministic — because the timer is clamped to 1ms it's a race that
> depends on startup speed. But inside an I/O callback `setImmediate` is
> guaranteed to come first, because the `check` phase follows `poll`
> immediately, while `timers` only comes on the next iteration."

## 8. CommonJS vs ESM (briefly, details — separate conspect)

The event loop itself is identical. The practical difference for this topic: in
ESM there's **top-level `await`**, which shifts when timers start relative to
microtasks; CJS doesn't have it. ESM's asynchronous loading also makes a "slow
start" more likely — the race leans toward the timer more often. Still no
guarantees.

---

See [en.01-event-loop.tasks.md](./en.01-event-loop.tasks.md) — trace tasks. Solutions in [en.01-event-loop.answers.md](./en.01-event-loop.answers.md).
