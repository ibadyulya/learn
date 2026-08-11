# Event Loop — задачи-трассировки

> 🌐 English version: [en.01-event-loop.tasks.md](./en.01-event-loop.tasks.md)

Для каждой задачи предскажи **порядок вывода**, не запуская код. Где порядок
недетерминирован — так и напиши и объясни почему. Разборы — в [ru.01-event-loop.answers.md](./ru.01-event-loop.answers.md).

Все примеры — CommonJS (Node), запуск `node file.js`.

---

## Задача 1 — база (микро vs макро)

```js
console.log('A');
setTimeout(() => console.log('B'), 0);
Promise.resolve().then(() => console.log('C'));
console.log('D');
```

---

## Задача 2 — добавляем nextTick

```js
setTimeout(() => console.log('1'), 0);
Promise.resolve().then(() => console.log('2'));
process.nextTick(() => console.log('3'));
console.log('4');
process.nextTick(() => console.log('5'));
```

---

## Задача 3 — микротаски вклиниваются внутри фазы

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

## Задача 4 — async/await (await === .then)

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

## Задача 5 — гонка на верхнем уровне

```js
setTimeout(() => console.log('timeout'), 0);
setImmediate(() => console.log('immediate'));
```

---

## Задача 6 — тот же код, но внутри I/O

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

## Задача 7 — микротаска порождает микротаску

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

## Задача 8 — всё вместе (босс)

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
