# CJS / ESM — solutions

> 🌐 Russian version: [02-modules-cjs-esm.answers.md](../ru/02-modules-cjs-esm.answers.md)

---

## Question 1
**CJS is synchronous and dynamic, ESM is asynchronous and static.**
`require('./x')` is an ordinary function call, executed at runtime at the point
where it stands: Node blockingly reads the file, executes it, returns
`module.exports`. Since it's code — you can put it in an `if`, in a function,
with a computed path.

`import` is not a call but a declaration: the engine parses all import/export
**before** running the code (phases construct → instantiate → evaluate), building
the dependency graph in advance. That's why `import` must be static and at the
top level — otherwise the graph can't be built before running. For conditional
loading there's a separate dynamic `import()`, it's asynchronous and returns a
promise.

## Question 2
- **CJS**: the export is copied by value at the moment of `require` → the importer
  got a snapshot of `0`. After `inc()` it still sees `0`.
- **ESM**: the import is a live read-only reference to the export cell. After
  `inc()` the importer sees `1`.

Key: ESM links by reference (binding), CJS by value (snapshot).

## Question 3
- `require('./esm.mjs')` from CJS — historically **not allowed**: ESM loads
  asynchronously, while `require` is synchronous. Workaround — dynamic
  `await import(...)` (inside an async function), it returns a promise with the
  namespace.
- The reverse — **allowed**: `import fs from 'fs'`, the CJS is pulled in as the
  `default` export.

Mnemonic: ESM→CJS is fine synchronously, CJS→ESM — only through `import()`.

## Question 4
In priority order:
1. Extension — `.mjs` always ESM, `.cjs` always CJS.
2. For `.js` — the `"type"` field in the nearest `package.json`:
   `"type":"module"` → ESM; `"commonjs"` or absent → CJS.

## Question 5
The event loop is common, but in ESM the moment of timer **registration** shifts:
asynchronous ESM loading + `top-level await` fragment execution (the code after
`await` is a microtask), so a "slow start" is more likely and by the time of the
`timers` phase 1ms has managed to pass → the timer ripens → `setTimeout` first.
But it's still **not a guarantee**, just a probability. Inside an I/O callback
the guarantee doesn't change: `setImmediate` is always earlier (`check` follows
`poll` immediately).

## Question 6
```js
import { fileURLToPath } from 'node:url';
import { dirname } from 'node:path';
const __dirname = dirname(fileURLToPath(import.meta.url));
```
`import.meta.url` — the URL of the current module; `fileURLToPath` turns it into
a path.
