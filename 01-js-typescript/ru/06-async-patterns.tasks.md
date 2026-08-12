# Асинхронность — задачи

Отвечай своими словами, потом сверяйся с [06-async-patterns.answers.md](./06-async-patterns.answers.md).

> 🌐 English version: [06-async-patterns.tasks.md](../en/06-async-patterns.tasks.md)

---

## Вопрос 1
Зачем в однопоточном JS вообще нужна асинхронность? Что случится, если ждать ответ
сети синхронно?

## Вопрос 2
Назови три состояния промиса. Сколько раз и как он может сменить состояние?

## Вопрос 3
Почему цепочка `.then().then().catch()` лучше вложенных колбэков? Что ловит `.catch`
в конце цепочки?

## Вопрос 4
`async/await` — это замена промисов или надстройка? Что возвращает async-функция?
Как `await` связан с `.then` и микротасками?

## Вопрос 5
Ускорь код, не меняя того, что `fetch` независимы:
```ts
const results = [];
for (const url of urls) {
  results.push(await fetch(url));
}
```

## Вопрос 6
В чём разница между `Promise.all`, `Promise.allSettled`, `Promise.race` и
`Promise.any`? По одному сценарию на каждый.

## Вопрос 7
Почему это не работает как ожидается и как починить?
```ts
items.forEach(async (item) => { await save(item); });
console.log('всё сохранено');
```

## Вопрос 8
Что напечатает? (связь с event loop)
```ts
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```
