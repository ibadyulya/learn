# Advanced TypeScript — задачи

> 🌐 English version: [en.03-advanced-typescript.tasks.md](./en.03-advanced-typescript.tasks.md)

Реализуй тип сам, без подсказок, потом сверься с [ru.03-advanced-typescript.answers.md](./ru.03-advanced-typescript.answers.md).
Проверяй себя мысленной подстановкой примера, как в разборе `ExtractByType`.

Пока покрыто: generics, keyof/`T[K]`, mapped types, conditional, `never`, key
remapping. Задачи на `infer` и дистрибутивность добавим после.

---

## Задача 1 — свой `Pick`
Реализуй `MyPick<T, K>` — оставить в `T` только ключи из `K`.
```ts
type MyPick<T, K extends keyof T> = /* ? */;
// MyPick<{a: number; b: string; c: boolean}, "a" | "c">  ⟶  {a: number; c: boolean}
```

## Задача 2 — свой `Readonly` и снятие readonly
а) `MyReadonly<T>` — все поля `readonly`.
б) `Mutable<T>` — наоборот, снять `readonly` со всех полей.
```ts
// Mutable<{ readonly a: number }>  ⟶  { a: number }
```

## Задача 3 — свой `Record`
```ts
type MyRecord<K extends string, V> = /* ? */;
// MyRecord<"x" | "y", number>  ⟶  { x: number; y: number }
```

## Задача 4 — якорная, по памяти
Не подглядывая, напиши `ExtractByType<T, U>` (оставить поля, чьё значение
подходит под `U`) и `ExcludeByType<T, U>` (выкинуть такие поля).

```ts
type SomeType = {
  f1: number;
  f2: boolean;
  f3: string;
  f4: boolean;
};
// ExtractByType<SomeType, boolean>  ⟶  { f2: boolean; f4: boolean }

type ExtractByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
}
```



## Задача 5 — только функции
`FunctionProps<T>` — оставить только те поля объекта, которые являются функциями.
```ts
// FunctionProps<{ id: number; save(): void; load(): string }>
//   ⟶  { save(): void; load(): string }
```

## Задача 6 — переименование ключей
`Getters<T>` — превратить каждый ключ `foo` в `getFoo`, значение — функция,
возвращающая исходный тип.
```ts
// Getters<{ name: string; age: number }>
//   ⟶  { getName: () => string; getAge: () => number }
// подсказка: понадобятся шаблонные литеральные типы и Capitalize
```

## Задача 7 — infer, распаковка массива
`Flatten<T>` — если `T` массив, вернуть тип его элемента; иначе сам `T`.
```ts
// Flatten<string[]>  ⟶  string
// Flatten<number>    ⟶  number
```

## Задача 8 — infer, свой Awaited (один уровень)
`UnwrapPromise<T>` — достать тип из `Promise<...>`; не промис — вернуть как есть.
```ts
// UnwrapPromise<Promise<User>>  ⟶  User
// UnwrapPromise<number>         ⟶  number
```

## Задача 9 — дистрибутивность
Что вернёт каждый и почему?
```ts
type A<T> = T extends string ? "yes" : "no";
type R1 = A<string | number>;      // ?
type R2 = A<never>;                // ?

type B<T> = [T] extends [string] ? "yes" : "no";
type R3 = B<string | number>;      // ?
```

## Задача 10 — свой NonNullable
`MyNonNullable<T>` — убрать из юниона `null` и `undefined`. Объясни, почему тут
важна дистрибутивность.
```ts
// MyNonNullable<string | null | undefined>  ⟶  string
```
