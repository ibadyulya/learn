# Types, coercion and equality

> 🌐 Russian version: [07-types-coercion-equality.md](../ru/07-types-coercion-equality.md)

Why `[] == ![]` is true, why `===`, and where the coercion "chaos" comes from.
Reads as a story; behind the apparent mess are derivable rules.

The through-line of the topic:

> **JS has two kinds of values — primitives and objects. Primitives are copied by
> value, objects by reference. When types in an operation don't match, JS
> **implicitly coerces** them by fixed rules; `==` coerces, `===` doesn't. Know the
> coercion rules and the falsy list, and the "magic" disappears. In practice: always
> `===`.**

---

## Primitives and objects

**7 primitives:** `string`, `number`, `boolean`, `null`, `undefined`, `symbol`,
`bigint`. Everything else (objects, arrays, functions) is an **object**. Type check
— `typeof` (with the historical bug `typeof null === 'object'`).

**Copying:** a primitive is copied **by value**, an object **by reference**.
```ts
let a = 1, b = a; b = 2;           // a === 1 — its own copy
let o = { x: 1 }, p = o; p.x = 2;  // o.x === 2 — the same reference
```
Hence: comparing objects with `===` compares **references**, not content
(`{a:1} === {a:1}` → `false`).

---

## `==` versus `===`

- **`===` (strict)** — equal only if **same type AND same value**. No coercion.
- **`==` (loose)** — with different types it **coerces first**, then compares.

`==` coercion rules (simplified): number vs string → string to number; boolean → to
number (`true`→1, `false`→0); object vs primitive → object to primitive. Hence the
surprises:
```ts
0 == '';        // true  (both → 0)
0 == '0';       // true
'' == '0';      // false ('' and '0' are both strings, no coercion)
null == undefined;  // true  (special rule)
null == 0;          // false (null equals only undefined)
[] == ![];      // true  (![] → false → 0; [] → '' → 0; 0 == 0)
NaN == NaN;     // false (NaN equals nothing, including itself)
```
**Takeaway:** `==` is non-transitive (`0 == ''`, `0 == '0'`, but `'' != '0'`) and
full of traps. **In practice — always `===`.** The one justified exception is
`x == null` as a short check for "`null` or `undefined`".

---

## Truthy / falsy

In a boolean context a value is coerced to boolean. **Exactly eight falsy values:**
```
false, 0, -0, 0n, "" (empty string), null, undefined, NaN
```
Everything else is **truthy**, including `'0'`, `'false'`, `[]`, `{}` (an empty
array and object are truthy!).
```ts
if ([]) console.log('yes');   // prints — [] is truthy
Boolean('0');                  // true
```

---

## Coercion in operations

**`+` is overloaded:** if at least one operand is a string — **concatenation**,
otherwise — numeric addition. Hence the classics:
```ts
1 + '2';     // '12'  (number → string)
'5' - 2;     // 3     (- has no string version → string to number)
1 + 2 + '3'; // '33'  (left to right: 3, then '3')
[] + [];     // ''    (both arrays → empty strings)
[] + {};     // '[object Object]'
```
Rule: `-`, `*`, `/`, `%` are always numeric (coerce to number); `+` is stringy if a
string is nearby.

---

## `null` vs `undefined`, `NaN`

- **`undefined`** — "no value assigned" (an unset property, argument, `return`
  without a value).
- **`null`** — "intentionally empty", set by the developer.
- **`NaN`** — "not a number" (the result of `0/0`, `Number('x')`); equals nothing,
  including itself → check with `Number.isNaN(x)`.

`Object.is(a, b)` — like `===`, but with two differences: `Object.is(NaN, NaN) ===
true` and `Object.is(0, -0) === false`.

---

## Interview phrasing

> "Values split into primitives (copied by value) and objects (by reference), so
> `{a:1} === {a:1}` is false — references are compared. `===` compares without
> coercion, `==` coerces types first by rules (number↔string, boolean→number,
> object→primitive), which produces non-transitive oddities like `0 == ''` and
> `[] == ![]`; so I always use `===` (exception — `x == null` to check
> null/undefined). Eight falsy: false, 0, -0, 0n, '', null, undefined, NaN — the
> rest are truthy, including empty `[]` and `{}`. `+` concatenates with a string,
> other arithmetic coerces to number. NaN isn't equal to itself — I check
> `Number.isNaN`."

---

See [07-types-coercion-equality.tasks.md](./07-types-coercion-equality.tasks.md) —
tasks. Solutions in [07-types-coercion-equality.answers.md](./07-types-coercion-equality.answers.md).
