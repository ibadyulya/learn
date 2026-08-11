# Advanced TypeScript — разборы

> 🌐 English version: [03-advanced-typescript.answers.md](../en/03-advanced-typescript.answers.md)

---

## Задача 1 — свой `Pick`
```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
};
```
Ключевой момент: цикл идёт **не** по `keyof T`, а прямо по `K` — то есть только
по тем ключам, что нам передали. `K extends keyof T` гарантирует, что там нет
лишних ключей. `T[P]` достаёт исходный тип значения.

## Задача 2 — `Readonly` / `Mutable`
```ts
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K]
};

type Mutable<T> = {
  -readonly [K in keyof T]: T[K]
};
```
Модификаторы `readonly` и `?` можно не только добавлять, но и **снимать** —
префиксом `-`. `-readonly` убирает readonly, `-?` делает поле обязательным.
Аналогично `Required<T>` это `{ [K in keyof T]-?: T[K] }`.

## Задача 3 — свой `Record`
```ts
type MyRecord<K extends string, V> = {
  [P in K]: V
};
```
Идём по юниону ключей `K`, каждому даём один и тот же тип `V`. (В настоящем
`Record<K, V>` ограничение шире: `K extends string | number | symbol`.)

## Задача 4 — якорная
```ts
type ExtractByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};
type ExcludeByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K]
};
```
Разница только в том, какая ветка отдаёт `K`, а какая `never`. `never` в позиции
ключа (после `as`) выкидывает свойство.

## Задача 5 — только функции
```ts
type FunctionProps<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? K : never]: T[K]
};
```
Это `ExtractByType`, где `U` = «любая функция», то есть
`(...args: any[]) => any`. Каркас тот же — поменяли только условие. Это и есть
смысл: один паттерн, разное сито.

## Задача 6 — переименование ключей (getters)
```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
```
Здесь три вещи вместе:
- **шаблонный литеральный тип** `` `get${...}` `` — собирает новую строку-ключ;
- `Capitalize<...>` — встроенная утилита, делает первую букву заглавной;
- `string & K` — потому что `K` формально `string | number | symbol`, а в
  шаблон и в `Capitalize` нужен именно `string`; пересечение `string & K`
  сужает до строковой части.

Значение `() => T[K]` — функция без аргументов, возвращающая исходный тип поля.

## Задача 7 — `Flatten`
```ts
type Flatten<T> = T extends (infer E)[] ? E : T;
```
`(infer E)[]` — образец «массив из чего-то»; `E` ловит тип элемента. Не массив —
образец не совпал, возвращаем сам `T`.

## Задача 8 — `UnwrapPromise`
```ts
type UnwrapPromise<T> = T extends Promise<infer V> ? V : T;
```
Образец `Promise<infer V>` — `V` ловит то, чем промис резолвится. (Настоящий
`Awaited` делает это рекурсивно, чтобы разворачивать `Promise<Promise<...>>`.)

## Задача 9 — дистрибутивность
```ts
type R1 = "yes" | "no";
// A<string|number> дистрибутивно: A<string>→"yes", A<number>→"no" → "yes"|"no"

type R2 = never;
// never — пустой юнион, условие применяется 0 раз → never

type R3 = "no";
// [T] extends [string]: юнион обёрнут в кортеж → НЕ дистрибутивно, сравнивается
// целиком. (string|number) не является string → "no"
```

## Задача 10 — `MyNonNullable`
```ts
type MyNonNullable<T> = T extends null | undefined ? never : T;
```
`T` голый → условный тип дистрибутивен, проверка идёт по каждому члену юниона:
`string`→оставить, `null`→never (выпадает), `undefined`→never (выпадает). Итог
`string`. Без дистрибутивности `(string|null|undefined) extends null|undefined`
целиком было бы ложью → вернулся бы весь юнион, ничего бы не отфильтровалось.
