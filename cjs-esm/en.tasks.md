# CJS / ESM — comprehension questions

Answer in your own words, then check against [en.answers.md](./en.answers.md).

> 🌐 Russian version: [ru.tasks.md](./ru.tasks.md)

---

## Question 1 — basics
Name the main **mechanical** difference between CJS and ESM loading (not about
syntax). Why does it follow that `require` can go inside an `if` but `import`
cannot?

## Question 2 — live bindings
```js
// counter.cjs / counter.mjs
export let count = 0;          // in CJS: module.exports.count = 0
export function inc() { count++; }
```
The importer called `inc()` and read `count` again. What prints in CJS and what
in ESM? Why?

## Question 3 — interop
Can a CJS file do `const x = require('./esm-module.mjs')`? And the other way —
import CJS from ESM? How do you work around the forbidden direction?

## Question 4 — format detection
There's a file `index.js` with no `.mjs` anywhere. What determines how Node reads
it — CJS or ESM? List the signals in priority order.

## Question 5 — event loop
Why does the classic "top-level race" `setTimeout(0)` vs `setImmediate` lean
toward the timer more often specifically in ESM? Does the guarantee change inside
an I/O callback?

## Question 6 — __dirname
ESM has no `__dirname`. How do you get the path to the current module's directory?
