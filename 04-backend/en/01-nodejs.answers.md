# Node.js — solutions

> 🌐 Russian version: [01-nodejs.answers.md](../ru/01-nodejs.answers.md)

---

## Question 1
1. **V8** — the engine, runs JavaScript.
2. **libuv** — the event loop and async I/O (plus a background thread pool).
3. **C++ bindings and core modules** (`fs`, `http`, `net`, ...) — access to the file
   system, network, OS.
Together — server-side JS with a non-blocking model.

## Question 2
Waiting I/O operations go to libuv and don't occupy the thread → Node serves
thousands of connections concurrently. A CPU-bound computation doesn't wait but
**loads the single thread**: while it computes, the event loop doesn't spin, other
requests **queue up** — the server "freezes" until the computation ends.

## Question 3
Single-threaded is the execution of **your JavaScript** (the event loop). libuv's
thread pool is for some system operations (file I/O, `crypto`, DNS) the OS doesn't
give non-blockingly; they run in the background, and the result returns to the main
thread via a callback. Your code sees no thread races — it's one.

## Question 4
`EventEmitter` is the base class of the event model: `emit('name', ...)` emits an
event, `on('name', cb)` subscribes a handler (the observer pattern). Streams, the
http server, and processes are built on it. The **`'error'`** event is special: if it
has no listener, the error isn't swallowed but **thrown and crashes the process** —
emitters that can error always get a handler.

## Question 5
Reading whole loads **the entire file into memory** — on large data that's an OOM and
latency (waiting for the full load). A stream processes **in portions**, memory stays
almost constant, and work starts right away. **Backpressure** is the mechanism where
a slow sink signals a fast source to "slow down" so the buffer doesn't overflow
memory; `pipe` does this automatically.

## Question 6
- **sync** — `try/catch`;
- **promise / async-await** — `try/catch` around `await` or `.catch`;
- **error-first callback** — check the first `err` argument;
- **stream/EventEmitter** — listen for the `'error'` event.
`process.on('uncaughtException')`/`'unhandledRejection'` — only for **logging and a
graceful shutdown**: after an uncaught error the process state is undefined,
"swallow and continue" is dangerous; the right way — log, close resources, exit (a
manager/orchestrator restarts it).

## Question 7
Password hashing (bcrypt/scrypt/argon2) is CPU-bound. Options to avoid blocking the
event loop:
- compute in **`worker_threads`** (a worker pool) — the heavy work goes to separate
  threads;
- use the **async** versions from `crypto` (some go via libuv's thread pool), not the
  synchronous ones;
- move it to a **queue/separate service**, process in batches off the HTTP path.
The key — don't spin a synchronous heavy loop on the main thread.
