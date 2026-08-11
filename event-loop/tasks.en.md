# Event Loop — trace tasks

For each task, predict the **output order** without running the code. Where the
order is non-deterministic — say so and explain why. Solutions — in
[answers.en.md](./answers.en.md).

> 🌐 Russian version: [tasks.ru.md](./tasks.ru.md)

All examples are CommonJS (Node), run with `node file.js`.

---

## Task 1 — basics (micro vs macro)

```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```

---

## Task 2 — adding nextTick

```js
setTimeout(() => console.log('1'), 0);
Promise.resolve().then(() => console.log('2'));
process.nextTick(() => console.log('3'));
console.log('4');
process.nextTick(() => console.log('5'));
```

---

## Task 3 — microtasks wedge in within a phase

```js
setTimeout(() => {
  console.log('timer 1');
  Promise.resolve().then(() => console.log('promise in 1'));
}, 0);

setTimeout(() => {
  console.log('timer 2');
  process.nextTick(() => console.log('nextTick in 2'));
}, 0);
```

---

## Task 4 — async/await (await === .then)

```js
async function main() {
  console.log('1');
  await null;
  console.log('2');
  await null;
  console.log('3');
}

console.log('start');
main();
Promise.resolve().then(() => console.log('promise'));
console.log('end');
```

---

## Task 5 — top-level race

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

---

## Task 6 — same code, but inside I/O

```js
const fs = require('fs');

fs.readFile(__filename, () => {
  setTimeout(() => console.log('timeout'), 0);
  setImmediate(() => console.log('immediate'));
  Promise.resolve().then(() => console.log('promise'));
  process.nextTick(() => console.log('nextTick'));
});
```

---

## Task 7 — a microtask spawns a microtask

```js
console.log('start');

setTimeout(() => console.log('timeout'), 0);

Promise.resolve().then(() => {
  console.log('promise 1');
  Promise.resolve().then(() => console.log('promise 2'));
  process.nextTick(() => console.log('nextTick in promise'));
});

process.nextTick(() => console.log('nextTick'));

console.log('end');
```

---

## Task 8 — all together (boss)

```js
console.log('=== 1 ===');

setTimeout(() => {
  console.log('setTimeout 1');
  Promise.resolve().then(() => console.log('promise in setTimeout'));
}, 0);

setImmediate(() => console.log('setImmediate 1'));

Promise.resolve().then(() => {
  console.log('promise 1');
  process.nextTick(() => console.log('nextTick in promise'));
});

process.nextTick(() => {
  console.log('nextTick 1');
  Promise.resolve().then(() => console.log('promise in nextTick'));
});

console.log('=== 2 ===');
```
