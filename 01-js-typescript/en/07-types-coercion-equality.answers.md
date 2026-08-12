# Types, coercion and equality — solutions

> 🌐 Russian version: [07-types-coercion-equality.answers.md](../ru/07-types-coercion-equality.answers.md)

---

## Question 1
Primitives: `string`, `number`, `boolean`, `null`, `undefined`, `symbol`, `bigint`.
A primitive is copied **by value** (variables are independent); an object **by
reference** (two variables point to one object, a change is visible to both).

## Question 2
Two object literals are **two different objects in memory**; `===` for objects
compares **references**, and they differ → `false`. To compare by content: manually
by fields, via `JSON.stringify(a) === JSON.stringify(b)` (careful with key order and
types), or a deep equality from a library (lodash `isEqual`).

## Question 3
`===` — equal only with **matching type and value**, no coercion. `==` coerces types
before comparing. Surprise examples:
```ts
0 == '';      // true — both coerce to 0
[] == ![];    // true — ![]→false→0, []→''→0, 0==0
'' == '0';    // false — both strings, no coercion
```

## Question 4
Falsy (8): `false`, `0`, `-0`, `0n`, `''`, `null`, `undefined`, `NaN`.
From the list, truthy: **`'0'`, `[]`, `{}`, `'false'`** (non-empty strings and any
objects/arrays are truthy). Falsy: `''`, `0`.

## Question 5
```ts
1 + '2'      // '12'  — a string is present → concatenation, 1→'1'
'5' - 2      //  3    — "-" has no string version → '5'→5
1 + 2 + '3'  // '33'  — left to right: 1+2=3, then 3+'3'='33'
'3' + 2 + 1  // '321' — '3'+2='32', then '32'+1='321'
```
Key: `+` is stringy with a string and evaluates left to right.

## Question 6
By the IEEE-754 standard `NaN` "equals nothing", including itself, so `NaN === NaN`
is `false`. Check with **`Number.isNaN(x)`** (not the global `isNaN`, which coerces
its argument first and lies: `isNaN('foo') === true`). Another option —
`Object.is(x, NaN)`.

## Question 7
- **`undefined`** — no value set: an unset property, a missing argument, `return`
  without a value, an uninitialized variable.
- **`null`** — an intentional "emptiness", set by the developer.
`null == undefined` → **true** (a `==` special rule); `null === undefined` →
**false** (different types).

## Question 8
`Object.is` is almost like `===`, but:
1. `Object.is(NaN, NaN)` → **true** (`===` → false);
2. `Object.is(0, -0)` → **false** (`===` → true).
Otherwise it matches `===`.
