# `this`, прототипы и наследование

> 🌐 English version: [05-this-prototypes-inheritance.md](../en/05-this-prototypes-inheritance.md)

Две темы, которые в JS работают не как в классических ООП-языках и потому пугают.
Читается как рассказ; обе сводятся к простым правилам.

Сквозная идея темы:

> **`this` определяется тем, КАК функцию вызвали (место вызова), а не где её
> объявили — это противоположно замыканиям. А наследование в JS — не про классы, а
> про прототипы: объект одалживает свойства у другого объекта по цепочке, а `class`
> — синтаксический сахар над этим.**

---

## Часть 1. `this` — решает место вызова

Главное правило: `this` не привязан к функции навсегда — он определяется **в момент
вызова**. Четыре правила по приоритету (от низшего к высшему):

**1. Обычный вызов (default).** `fn()` сам по себе → `this` — `undefined` (в strict
mode) или global-объект (без strict).
```ts
function f() { return this; }
f();   // undefined в strict
```

**2. Вызов как метод (implicit).** `obj.method()` → `this` — объект **слева от
точки**.
```ts
const user = { name: 'Ann', hi() { return this.name; } };
user.hi();   // 'Ann' — this === user
```

**3. Явная привязка (explicit).** `call` / `apply` / `bind` задают `this` руками.
```ts
function hi() { return this.name; }
hi.call({ name: 'Bob' });    // 'Bob'
const bound = hi.bind({ name: 'Cid' });
bound();                     // 'Cid' — bind возвращает функцию с зафиксированным this
```

**4. Вызов с `new` (конструктор).** `new Fn()` создаёт новый объект и делает его
`this`.
```ts
function User(name: string) { this.name = name; }
new User('Dan').name;   // 'Dan'
```

**Стрелочные функции — исключение и спасение.** У них **нет своего `this`** — они
берут его **лексически**, из места объявления (как обычную переменную). Их нельзя
перепривязать `call`/`bind`.
```ts
const obj = {
  name: 'Eve',
  greet() { return [1].map(() => this.name); }   // стрелка берёт this из greet → 'Eve'
};
```

### Классический баг: потеря `this`
```ts
const user = { name: 'Ann', hi() { return this.name; } };
const fn = user.hi;
fn();                        // undefined — вызов «обычный», this потерян
setTimeout(user.hi, 0);      // тоже теряет this
```
Метод, оторванный от объекта, вызывается «обычно» → правило 1. Лечение: `bind`
(`user.hi.bind(user)`), обёртка-стрелка (`() => user.hi()`) или стрелочный метод в
классе (`hi = () => ...`, замыкает `this` экземпляра).

---

## Часть 2. Прототипы — как устроено наследование

В JS **нет классов в классическом смысле.** Есть **прототипное наследование**: у
каждого объекта есть скрытая ссылка `[[Prototype]]` (доступна как `__proto__`) на
**другой объект** — его прототип. Свойства ищутся так:

> Обращение `obj.x` → ищем `x` на самом `obj`; нет — идём по `__proto__` на прототип,
> потом на прототип прототипа, и так до `null`. Это **цепочка прототипов**.

```ts
const animal = { eats: true };
const dog = Object.create(animal);   // dog.__proto__ === animal
dog.barks = true;
dog.eats;   // true — своего нет, взяли у прототипа
dog.barks;  // true — своё
```
Не путать два слова:
- **`prototype`** — свойство **функции-конструктора**: объект, который станет
  прототипом для создаваемых через `new` экземпляров.
- **`__proto__`** (`[[Prototype]]`) — ссылка **самого объекта** на свой прототип.

```ts
function User(name: string) { this.name = name; }
User.prototype.hi = function () { return this.name; };   // метод в прототипе
const u = new User('Ann');
u.hi();                       // 'Ann' — нашли hi в User.prototype по цепочке
u.__proto__ === User.prototype;   // true
```
Почему метод кладут в `prototype`, а не в конструктор: тогда он хранится **в одном
месте** и общий для всех экземпляров, а не копируется в каждый — экономия памяти.

---

## `class` — сахар над прототипами

`class` в JS не вводит новую модель — это **синтаксический сахар** над тем же
прототипным механизмом:
```ts
class User {
  constructor(public name: string) {}
  hi() { return this.name; }       // фактически кладётся в User.prototype
}
class Admin extends User {          // extends связывает цепочку прототипов
  constructor(name: string) { super(name); }
}
```
`hi` живёт в `User.prototype`; `extends` выставляет `Admin.prototype.__proto__ ===
User.prototype`; `super` зовёт метод родителя по цепочке. Понимая это, не удивляешься,
почему `this` в методе может «потеряться» (метод — обычная функция в прототипе) и
почему `instanceof` — это проверка «есть ли `X.prototype` в цепочке».

---

## Формулировка для собеса

> «`this` в JS определяется местом вызова, а не объявления: обычный вызов →
> undefined/global, `obj.method()` → this = obj, `call/apply/bind` задают явно,
> `new` → новый объект. Стрелочные функции своего this не имеют — берут лексически,
> и их нельзя перепривязать, поэтому ими удобно чинить «потерю this» в колбэках.
> Наследование в JS прототипное: у объекта есть ссылка на объект-прототип, свойства
> ищутся вверх по цепочке прототипов до null. `prototype` — свойство конструктора
> (будущий прототип экземпляров), `__proto__` — ссылка самого объекта. Методы кладут
> в prototype, чтобы они были общими. `class` — сахар над всем этим.»

---

См. [05-this-prototypes-inheritance.tasks.md](./05-this-prototypes-inheritance.tasks.md)
— задачи. Разборы — в [05-this-prototypes-inheritance.answers.md](./05-this-prototypes-inheritance.answers.md).
