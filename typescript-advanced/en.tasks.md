# Advanced TypeScript — tasks

Implement the type yourself, without hints, then check against
[en.answers.md](./en.answers.md). Test yourself by mentally substituting an
example, like in the `ExtractByType` walkthrough.

> 🌐 Russian version: [ru.tasks.md](./ru.tasks.md)

Covered so far: generics, keyof/`T[K]`, mapped types, conditional, `never`, key
remapping. Tasks on `infer` and distributivity are included too.

---

## Task 1 — your own `Pick`
Implement `MyPick<T, K>` — keep in `T` only the keys from `K`.
```ts
type MyPick<T, K extends keyof T> = /* ? */;
// MyPick<{a: number; b: string; c: boolean}, "a" | "c">  ⟶  {a: number; c: boolean}
```

## Task 2 — your own `Readonly` and stripping readonly
a) `MyReadonly<T>` — all fields `readonly`.
b) `Mutable<T>` — the reverse, strip `readonly` from all fields.
```ts
// Mutable<{ readonly a: number }>  ⟶  { a: number }
```

## Task 3 — your own `Record`
```ts
type MyRecord<K extends string, V> = /* ? */;
// MyRecord<"x" | "y", number>  ⟶  { x: number; y: number }
```

## Task 4 — the anchor, from memory
Without peeking, write `ExtractByType<T, U>` (keep fields whose value fits `U`)
and `ExcludeByType<T, U>` (throw out such fields).

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

## Task 5 — functions only
`FunctionProps<T>` — keep only those object fields that are functions.
```ts
// FunctionProps<{ id: number; save(): void; load(): string }>
//   ⟶  { save(): void; load(): string }
```

## Task 6 — renaming keys
`Getters<T>` — turn each key `foo` into `getFoo`, the value a function returning
the original type.
```ts
// Getters<{ name: string; age: number }>
//   ⟶  { getName: () => string; getAge: () => number }
// hint: you'll need template literal types and Capitalize
```

## Task 7 — infer, unpacking an array
`Flatten<T>` — if `T` is an array, return its element type; otherwise `T` itself.
```ts
// Flatten<string[]>  ⟶  string
// Flatten<number>    ⟶  number
```

## Task 8 — infer, your own Awaited (one level)
`UnwrapPromise<T>` — pull the type out of `Promise<...>`; not a promise — return
as is.
```ts
// UnwrapPromise<Promise<User>>  ⟶  User
// UnwrapPromise<number>         ⟶  number
```

## Task 9 — distributivity
What does each return and why?
```ts
type A<T> = T extends string ? "yes" : "no";
type R1 = A<string | number>;      // ?
type R2 = A<never>;                // ?

type B<T> = [T] extends [string] ? "yes" : "no";
type R3 = B<string | number>;      // ?
```

## Task 10 — your own NonNullable
`MyNonNullable<T>` — remove `null` and `undefined` from a union. Explain why
distributivity matters here.
```ts
// MyNonNullable<string | null | undefined>  ⟶  string
```
