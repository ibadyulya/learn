# Advanced TypeScript

Type-level programming: generics, keyof/`T[K]`, mapped types, conditional types,
`never`, key remapping, `infer`, distributivity, a walkthrough of the utility
types. The goal — not to memorize tricks, but to be able to **derive** a type
out loud, like code. Reads as a story; the final anchor task (`ExtractByType`)
is assembled from building blocks introduced one at a time.

> 🌐 Russian version: [README.ru.md](./README.ru.md)

The through-line of the whole topic: **types in TypeScript are a second
programming language that runs at compile time and operates on types the way
ordinary JS operates on values. Everything "advanced" is just programming in
that language.**

---

## The key mental shift: types are a second language

In TypeScript two languages live at once.

The first is ordinary JavaScript. It runs at runtime and operates on **values**:
numbers, strings, objects.

The second is the type language. It runs **only at compile time**, before
execution, and operates on **types** the way the first operates on values. And
this is the crux: **types are the "values" of the second language.** You don't
just tag variables with them — you **compute** them: take one type, transform it,
get another.

When you write `Partial<User>` — you aren't "declaring a type". You're **calling
a function** of the second language: input the type `User`, output a new type
where all fields are optional. `Partial` is the function, `User` the argument,
the result a new type.

Once you see this, all "advanced" TypeScript stops being a bag of tricks and
becomes ordinary programming — just in a language where the values are types. It
has:
- variables and functions — **generics**;
- conditions — **conditional types** (`extends ? :`);
- loops over properties — **mapped types**;
- pattern matching and unpacking — **`infer`**;
- an "empty value" — **`never`**.

Everything below is just these constructs. Keep in mind: **we write a program
that takes types as input and produces types as output.**

---

## Generics — the functions of the second language

Look at a generic literally as a function, only with angle brackets instead of
parentheses.

```ts
type Box<T> = { value: T };
```

Read it as a function definition: `Box` takes a parameter `T`, returns the type
`{ value: T }`. "Calling" it:

```ts
type StringBox = Box<string>;   // { value: string }
type NumberBox = Box<number>;   // { value: number }
```

You substituted `string` for `T` — exactly like an argument to an ordinary
function. No magic: `Box<T>` is `function Box(T) { return {value: T} }`, only in
the type language.

Parameters can have **constraints** — via `extends`. The word `extends` is
overloaded in TS; in the context of a generic parameter it means **"I accept not
any type, but only one that fits this shape"**:

```ts
type Lengthy<T extends { length: number }> = T;
// T extends {length: number}  ⟶  "T must have a length: number field"

Lengthy<string>;     // ok — a string has length
Lengthy<number[]>;   // ok — an array has length
Lengthy<number>;     // error — a number has no length
```

Read `T extends U` as **"T qualifies as U / is a special case of U"**. This same
word shows up in conditional types and means the same thing there, just used as a
question. Remember the feel: **`extends` = "does it fit / is it a special case"**.

---

## keyof and `T[K]` — reading the structure of a type

Two tools for working with object types: get the keys, and pull the value type
by key.

`keyof` takes an object type and returns **the union of its keys** (as string
literals):

```ts
type User = { id: number; name: string; active: boolean };

type Keys = keyof User;   // "id" | "name" | "active"
```

This is not an array but a **union of literal types** — "either this, or that, or
that".

`T[K]` — **indexed access**, "give me the value type by key", like `obj[key]` in
JS but at the type level:

```ts
type IdType = User["id"];       // number
type Vals = User[keyof User];   // number | string | boolean
```

These two — `keyof T` (all keys) and `T[K]` (the value type by key) — are our
working hands. In almost every task you take the keys via `keyof`, then rummage
through the values via `T[K]`.

---

## Mapped types — a loop over properties

We have `keyof T` — the union of keys. Can we walk over them and build a new
object type? Yes. That's a mapped type. The syntax is scary, but it's literally
a loop over the keys:

```ts
type Copy<T> = {
  [K in keyof T]: T[K]
};
```

Word for word: "for each `K` in `keyof T` — create a key `K` with value type
`T[K]`". `[K in keyof T]` is the loop header, `K` the variable running over the
union of keys; `T[K]` on the right reads the value by key. `Copy<T>` copies the
type one to one, but now we have a **loop body** where we can change things.

```ts
type Stringify<T> = { [K in keyof T]: string };

type User = { id: number; active: boolean };
type S = Stringify<User>;   // { id: string; active: string }
```

The built-in utilities work under the hood the same way — you can write them
yourself:

```ts
type MyPartial<T>  = { [K in keyof T]?: T[K] };          // all fields optional
type MyReadonly<T> = { readonly [K in keyof T]: T[K] };  // all readonly
```

Keep in mind: **a mapped type = a loop over keys, building a new object.**

---

## Conditional types — the `if` of the second language

Looks like a ternary, works like an `if` at the type level:

```ts
type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;   // true
type B = IsString<42>;        // false
```

That same `extends`, and it means the same — **"T fits / is a special case"** —
only now as a **question**: `T extends string ?` = "is T a string? yes → first
branch, no → second".

```ts
type ElementType<T> = T extends any[] ? "array" : "not array";
ElementType<number[]>;   // "array"
ElementType<string>;     // "not array"
```

Now we have a loop (mapped) and a condition (conditional). One detail remains —
`never`.

---

## `never` — "this key isn't here"

`never` is the most misunderstood type. Formally it's "a type with no possible
value" (a function that never returns but always throws has return type `never`).
But for our tasks one **practical property** matters, and it's magical:

**If in a mapped type you assign `never` to a key in the key position — that key
vanishes from the result.** It doesn't become a key with value `never` — it
**disappears entirely.** As if you said "and this field won't be here".

This property — "`never` in the key position = delete the key" — is the mechanism
for **filtering out** unneeded fields. Remember it separately, it's the core of
the task. But to put `never` exactly in the *key position*, we need one last
detail.

---

## Key remapping — renaming a key via `as`

By default a mapped type preserves the key. But you can **reassign** it — via
`as` right in the loop header:

```ts
type T = {
  [K in keyof Obj as NewKey]: Obj[K]
};
```

`as` says: "the key in the result will be not `K` but this one". Usually that's
how you rename fields. But the trick: **if you compute `as ... never` — the key
disappears** (the tie-in with the previous point). And if you put a condition in
the `as` position:

```ts
[K in keyof T as (condition) ? K : never]
```

"for each key `K` — if the condition holds, keep the key as `K`; if not, rename
it to `never`, i.e. **throw it out**". This is a filter over keys: the condition
is the sieve, `never` the trash bin.

---

## Assembling `ExtractByType<T, U>` (the anchor interview task)

Given an object type `T`, keep only the fields whose **value type fits `U`**.

```ts
type SomeType = { f1: number; f2: boolean; f3: string; f4: boolean };
// ExtractByType<SomeType, boolean>  ⟶  { f2: boolean; f4: boolean }
```

Assemble it step by step:

**Step 1 — loop over keys** (mapped type), for now just a copy:
```ts
type ExtractByType<T, U> = { [K in keyof T]: T[K] };
```

**Step 2 — we want to filter keys**, so a condition in the key position via `as`:
```ts
type ExtractByType<T, U> = { [K in keyof T as ...condition...]: T[K] };
```

**Step 3 — the condition**: "does the value type `T[K]` fit `U`?" that's
`T[K] extends U`. Yes → keep key `K`, no → `never` (throw it out):
```ts
type ExtractByType<T, U> = {
  [K in keyof T as T[K] extends U ? K : never]: T[K]
};
```

Check on `ExtractByType<SomeType, boolean>`:
- `f1`: `number extends boolean`? no → `never` → **thrown out**;
- `f2`: `boolean extends boolean`? yes → **kept**;
- `f3`: `string extends boolean`? no → **thrown out**;
- `f4`: `boolean extends boolean`? yes → **kept**.

Result: `{ f2: boolean; f4: boolean }`. Exactly what was asked.

The beauty is that the whole task is **three general tools**: a loop
(`[K in keyof T]`), a condition (`T[K] extends U ?`) and `never`-as-"throw out"
via `as`. None of them is about "this specific task". Give a variant — you change
only the condition, the skeleton stays. For example, the reverse task (throw out
the matching ones) — just swap the branches:

```ts
type ExcludeByType<T, U> = {
  [K in keyof T as T[K] extends U ? never : K]: T[K]
};
```

---

## `infer` — "catch me this piece of the type"

So far we could read structure that already exists (`keyof`, `T[K]`). But often
you need to **extract a type hidden inside another type**: "given a function
type — pull out the type of what it returns". You don't know in advance what's
there — the type has to be *caught*. That's what `infer` is for.

Works **only inside a conditional type** and means literally: "I don't know
what's here — name it a variable and give it to me".

```ts
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

type A = MyReturnType<() => string>;         // string
type B = MyReturnType<(x: number) => User>;  // User
type C = MyReturnType<number>;               // never — not a function, pattern didn't match
```

Read it as pattern matching: "does `T` fit the shape *a function returning
something*? Name that *something* `R`. Matched → return `R`, no → `never`".

Intuition: **`infer R` is a hole in the pattern, into which TypeScript itself
substitutes what's actually there and returns it under the name `R`.** You
describe a shape with a hole, the engine matches and fills it. The hole can go
anywhere in the pattern:

```ts
type ElementOf<T> = T extends (infer E)[] ? E : never;
ElementOf<string[]>;   // string  — "an array of something? name it E"

type FirstArg<T> = T extends (first: infer F, ...rest: any[]) => any ? F : never;
FirstArg<(x: number, y: string) => void>;   // number
```

Caveat: `infer` lives **only in the position after `extends`** in a conditional
type. You can't write it in an ordinary mapped type or just anywhere.

---

## Distributivity — a conditional type "spreads" over a union

A subtle spot interviewers love. Logic suggests one answer, TypeScript gives
another:

```ts
type ToArray<T> = T extends any ? T[] : never;
type X = ToArray<string | number>;   // string[] | number[]  — NOT (string|number)[]
```

What happened: TS didn't take the union whole, but **broke it into members,
applied the conditional type to each separately, and reassembled into a union**:

```
ToArray<string | number>
=  ToArray<string> | ToArray<number>     ← broke it apart
=  string[]        | number[]            ← applied to each
```

This is distributivity — like multiplication over addition: `2*(3+4) = 2*3 + 2*4`.

**Trigger condition:** distributivity kicks in only if the checked type is a
"naked" parameter (`T` itself on the left of `extends`, not wrapped). The switch
to turn it off — wrap both sides in a tuple:

```ts
type A<T> = T extends any ? T[] : never;       // naked T → DISTRIBUTIVE
type B<T> = [T] extends [any] ? T[] : never;   // in a tuple → NOT distributive

B<string | number>;   // (string | number)[]  — the whole union at once
```

`[T] extends [U]` is the standard way to say "don't take the union apart, compare
as is" (for example, to check that `T` is *exactly* some union).

**Why `Exclude` rests on this:**
```ts
type Exclude<T, U> = T extends U ? never : T;   // naked T → distributive
Exclude<"a" | "b" | "c", "a">
// "a" extends "a" ? never : "a"  → never
// "b" extends "a" ? never : "b"  → "b"
// "c" extends "a" ? never : "c"  → "c"
// never | "b" | "c"  →  "b" | "c"   (never dissolves in a union)
```
Without distributivity `"a"|"b"|"c" extends "a"` as a whole would be false → the
entire union would come back, the filter wouldn't work. All the work of
`Exclude`/`Extract`/`NonNullable` rests on breaking apart by members.

**A trap — `never` is the empty union.** A distributive type breaks a union into
members; `never` has zero of them → the condition applies 0 times → the result is
`never` ("the loop spun 0 times"):
```ts
type R = Exclude<never, string>;   // never
```
Practical takeaway: if `never` accidentally reaches a distributive type, it
silently returns `never`, doing nothing — a source of quiet bugs.

---

## Standard library utility types

Almost all of them are **not compiler magic, but ordinary types on top of the
mechanisms we've covered**. Grouped by mechanism so you can see the kinship.

**Change field modifiers (mapped types):**
```ts
type Partial<T>  = { [K in keyof T]?: T[K] };            // all fields optional
type Required<T> = { [K in keyof T]-?: T[K] };           // all required (strip ?)
type Readonly<T> = { readonly [K in keyof T]: T[K] };    // all readonly
```

**Select a subset of keys:**
```ts
type Pick<T, K extends keyof T> = { [P in K]: T[P] };    // keep only K
type Omit<T, K> = Pick<T, Exclude<keyof T, K>>;          // drop K = Pick + Exclude
```
`Omit` stands on the shoulders of `Pick` and `Exclude`: "Pick over all keys
except these". On the backend every day: `Omit<User, "password">`.

**Filter unions (conditional + never + distributivity):**
```ts
type Exclude<T, U>    = T extends U ? never : T;              // remove members of U
type Extract<T, U>    = T extends U ? T : never;             // keep the matching ones
type NonNullable<T>   = T extends null | undefined ? never : T;
```

**Build an object:**
```ts
type Record<K extends keyof any, V> = { [P in K]: V };   // keyof any = string|number|symbol
```

**Extract types from functions (infer):**
```ts
type ReturnType<T>  = T extends (...a: any[]) => infer R ? R : never;
type Parameters<T>  = T extends (...a: infer P) => any ? P : never;   // tuple of arguments
// plus Awaited<T> (unwrap a promise, recursively), InstanceType, ConstructorParameters
```

**String ones (these really are built into the compiler, you can't write them yourself):**
```ts
Uppercase<"abc">  // "ABC"      Capitalize<"hello">   // "Hello"
Lowercase<"ABC">  // "abc"      Uncapitalize<"Hello"> // "hello"
```

Conclusion: **the entire standard type library is those same five mechanisms.**
Modifiers → mapped; key selection → mapped + remapping; union filtering →
conditional + never + distributivity; function unpacking → infer. Not a list to
cram, but familiar techniques.

---

## Interview phrasing

> "TypeScript's type system is a separate language where the values are types.
> Generics are its functions, `extends` in a conditional type is its `if`, a
> mapped type (`[K in keyof T]`) is its loop over keys, and `never` in the key
> position deletes the key. To filter an object's properties by value type, I
> walk the keys with a mapped type, and in key remapping via `as` I put the
> condition `T[K] extends U ? K : never`: matching keys stay, the rest become
> `never` and drop out. `infer` in a conditional type unpacks a nested type (for
> example a function's return: `... => infer R ? R : never`). And a conditional
> type over a naked parameter is also distributive — it spreads over the members
> of a union, which is what `Exclude`/`Extract` are built on. That's why almost
> all utility types aren't compiler magic but ordinary types on top of these
> mechanisms."

---

See [tasks.en.md](./tasks.en.md) — implementation tasks. Solutions — in [answers.en.md](./answers.en.md).
