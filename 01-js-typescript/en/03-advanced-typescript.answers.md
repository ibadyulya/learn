# Advanced TypeScript — solutions

> 🌐 Russian version: [03-advanced-typescript.answers.md](../ru/03-advanced-typescript.answers.md)

---

## Task 1 — your own `Pick`
```ts
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P]
};
```
Key point: the loop runs **not** over `keyof T`, but directly over `K` — i.e.
only over the keys we were given. `K extends keyof T` guarantees there are no
stray keys. `T[P]` pulls the original value type.

## Task 2 — `Readonly` / `Mutable`
```ts
type MyReadonly<T> = {
  readonly [K in keyof T]: T[K]
};

type Mutable<T> = {
  -readonly [K in keyof T]: T[K]
};
```
The `readonly` and `?` modifiers can be not only added but also **stripped** —
with a `-` prefix. `-readonly` removes readonly, `-?` makes a field required.
Similarly `Required<T>` is `{ [K in keyof T]-?: T[K] }`.

## Task 3 — your own `Record`
```ts
type MyRecord<K extends string, V> = {
  [P in K]: V
};
```
We walk the union of keys `K`, giving each the same type `V`. (In the real
`Record<K, V>` the constraint is broader: `K extends string | number | symbol`.)

## Task 4 — the anchor
```ts
type ExtractByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};
type ExcludeByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K]
};
```
The only difference is which branch hands out `K` and which `never`. `never` in
the key position (after `as`) throws the property out.

## Task 5 — functions only
```ts
type FunctionProps<T> = {
  [K in keyof T as T[K] extends (...args: any[]) => any ? K : never]: T[K]
};
```
This is `ExtractByType` where `U` = "any function", i.e.
`(...args: any[]) => any`. Same skeleton — we changed only the condition. That's
the whole point: one pattern, a different sieve.

## Task 6 — renaming keys (getters)
```ts
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K]
};
```
Three things together here:
- **a template literal type** `` `get${...}` `` — assembles the new string key;
- `Capitalize<...>` — a built-in utility, capitalizes the first letter;
- `string & K` — because `K` is formally `string | number | symbol`, but the
  template and `Capitalize` need exactly a `string`; the intersection `string & K`
  narrows to the string part.

The value `() => T[K]` — a function with no arguments returning the field's
original type.

## Task 7 — `Flatten`
```ts
type Flatten<T> = T extends (infer E)[] ? E : T;
```
`(infer E)[]` — the pattern "an array of something"; `E` catches the element
type. Not an array — the pattern didn't match, return `T` itself.

## Task 8 — `UnwrapPromise`
```ts
type UnwrapPromise<T> = T extends Promise<infer V> ? V : T;
```
The pattern `Promise<infer V>` — `V` catches what the promise resolves with. (The
real `Awaited` does this recursively, to unwrap `Promise<Promise<...>>`.)

## Task 9 — distributivity
```ts
type R1 = "yes" | "no";
// A<string|number> distributive: A<string>→"yes", A<number>→"no" → "yes"|"no"

type R2 = never;
// never — empty union, condition applies 0 times → never

type R3 = "no";
// [T] extends [string]: union wrapped in a tuple → NOT distributive, compared
// as a whole. (string|number) is not string → "no"
```

## Task 10 — `MyNonNullable`
```ts
type MyNonNullable<T> = T extends null | undefined ? never : T;
```
`T` is naked → the conditional type is distributive, the check runs over each
member of the union: `string`→keep, `null`→never (drops out),
`undefined`→never (drops out). Result `string`. Without distributivity
`(string|null|undefined) extends null|undefined` as a whole would be false → the
entire union would come back, nothing would be filtered.
