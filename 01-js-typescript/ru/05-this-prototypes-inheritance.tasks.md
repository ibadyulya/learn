# `this`, прототипы и наследование — задачи

Отвечай своими словами, потом сверяйся с [05-this-prototypes-inheritance.answers.md](./05-this-prototypes-inheritance.answers.md).

> 🌐 English version: [05-this-prototypes-inheritance.tasks.md](../en/05-this-prototypes-inheritance.tasks.md)

---

## Вопрос 1
От чего зависит `this` — от места объявления функции или места вызова? Назови четыре
правила определения `this`.

## Вопрос 2
Что напечатает и почему? Как починить, чтобы вывелось `'Ann'`?
```ts
const user = { name: 'Ann', hi() { return this.name; } };
const fn = user.hi;
console.log(fn());
```

## Вопрос 3
Чем стрелочная функция отличается от обычной в плане `this`? Почему стрелку нельзя
перепривязать через `bind`?

## Вопрос 4
В чём разница между `prototype` и `__proto__`? У кого какое?

## Вопрос 5
Опиши, что происходит при обращении `obj.method()`, если у `obj` своего `method`
нет. Что такое цепочка прототипов?

## Вопрос 6
Почему методы кладут в `Constructor.prototype`, а не присваивают в конструкторе
(`this.method = ...`)? В чём выгода?

## Вопрос 7
`class` в JS — это принципиально новая сущность или сахар? Что под капотом делает
`extends` и `super`?

## Вопрос 8
Что выведет и почему?
```ts
class Timer {
  seconds = 0;
  start() { setInterval(function () { this.seconds++; }, 1000); }
}
new Timer().start();
```
Как исправить?
