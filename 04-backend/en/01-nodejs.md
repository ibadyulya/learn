# Node.js

> 🌐 Russian version: [01-nodejs.md](../ru/01-nodejs.md)

What Node is on the inside and what primitives it gives for server-side JS. Reads
as a story; builds on the [event loop](../../01-js-typescript/en/01-event-loop.md)
and [modules](../../01-js-typescript/en/02-modules-cjs-esm.md).

The through-line of the topic:

> **Node is the V8 engine (runs JS) + the libuv library (gives async I/O and the
> event loop) + C++ bindings (access to files, network, OS). Your JS runs on one
> thread, but waiting operations go to the background, so Node is strong on I/O load
> and weak on heavy computation.**

---

## What Node is made of

- **V8** — Google's JavaScript engine: compiles and runs JS.
- **libuv** — a C library: provides the **event loop** and **async I/O** (files,
  network, timers), plus a background **thread pool** for operations the OS can't do
  non-blockingly.
- **Bindings/core** — the C++ bridge and standard modules (`fs`, `http`, `net`,
  `crypto`) giving JS access to the system that the browser doesn't have.

The result: **JavaScript on the server** with a non-blocking model.

---

## Single thread and I/O vs CPU

Your JS runs on **one thread** (see
[event loop](../../01-js-typescript/en/01-event-loop.md)). Waiting operations (a DB
query, a file read) don't block the thread — they go to libuv, and the callback is
invoked when ready. So Node handles **thousands of concurrent I/O connections** well
(API gateways, chats, proxies).

The flip side — **CPU-bound** work (heavy computation, image processing) **blocks**
the single thread: while you compute, other requests wait. Solutions — move it to
`worker_threads`, a separate service, or a queue (see
[scaling](../../06-system-design/en/05-scaling.md)).

> Caveat: libuv does some I/O via a thread pool (4 by default), but **your JS** is
> one thread; that's the bottleneck for computation.

---

## EventEmitter — Node's event model

Much of Node is built on the observer pattern: an object emits **events** that others
subscribe to.
```ts
import { EventEmitter } from 'node:events';
const bus = new EventEmitter();
bus.on('order', (id) => console.log('order', id));   // subscribe
bus.emit('order', 42);                                // event
```
Streams, the HTTP server (`server.on('request', ...)`), and processes stand on
`EventEmitter`. Importantly, the **`'error'`** event is special — if it has no
listener, the uncaught error **crashes the process**.

---

## Streams and buffers — data in chunks

A **Buffer** is a chunk of raw binary data (which browser JS lacks). A **Stream** is
a flow of data processed **in portions**, not loaded whole into memory.
```ts
// bad for large files: the whole file in memory
const data = await readFile('huge.log');
// good: a stream, processed in chunks, memory stays constant
createReadStream('huge.log').pipe(gzip).pipe(createWriteStream('huge.log.gz'));
```
Why: process gigabytes with little memory, start work before the full load. The key
concept — **backpressure**: if the sink is slower than the source, the stream slows
the source down so it doesn't overflow memory. Streams are async-iterable
(`for await...of`).

Four types: Readable, Writable, Duplex, Transform.

---

## Error handling in Node

Different mechanisms for different models:
- **synchronous** — `try/catch`;
- **promises/async-await** — `try/catch` around `await` or `.catch`;
- **error-first callbacks** — the old core style: `(err, data) => { if (err) ... }`;
- **EventEmitter/streams** — listen for the `'error'` event;
- last resort — `process.on('uncaughtException')` / `'unhandledRejection'`: good only
  for **logging and a graceful shutdown**, not "swallow and keep running" (the
  process state is already unknown).

---

## Interview phrasing

> "Node is V8 plus libuv plus C++ bindings: JS on the server with non-blocking I/O.
> My code is one event-loop thread, but waiting operations go to libuv and come back
> via a callback, so Node handles thousands of I/O connections; CPU-bound tasks
> block the thread — I move them to worker_threads/a queue. The event model is
> EventEmitter (streams and http stand on it). Large data I process with streams in
> chunks with backpressure, not loading it all into memory. I handle errors by model:
> try/catch for sync/async, error-first for callbacks, the 'error' event for streams;
> uncaughtException only for logging and graceful shutdown."

---

See [01-nodejs.tasks.md](./01-nodejs.tasks.md) — tasks. Solutions in
[01-nodejs.answers.md](./01-nodejs.answers.md).
