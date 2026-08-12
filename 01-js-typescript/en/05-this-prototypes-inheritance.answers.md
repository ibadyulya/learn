# `this`, prototypes and inheritance — solutions

> 🌐 Russian version: [05-this-prototypes-inheritance.answers.md](../ru/05-this-prototypes-inheritance.answers.md)

---

## Question 1
On the **call site**, not the declaration. Four rules (increasing priority):
1. **plain call** `fn()` → `this` = `undefined` (strict) / global;
2. **as a method** `obj.method()` → `this` = the object left of the dot;
3. **explicit** `call`/`apply`/`bind` → the given object;
4. **`new Fn()`** → the newly created object.
Separately: arrows take `this` lexically (have none of their own).

## Question 2
It prints **`undefined`**. `const fn = user.hi` tore the method off the object;
`fn()` is a plain call (rule 1), `this` isn't `user`. Fixes:
```ts
const fn = user.hi.bind(user);   // hard-bind
// or
console.log(user.hi());          // call through the object
// or
const fn2 = () => user.hi();     // arrow wrapper
```

## Question 3
A regular function gets `this` **at call time** (by the four rules). An arrow has
**no `this` of its own** — it takes it from the enclosing scope **at declaration
time**, like an ordinary variable from a closure. So `call/apply/bind` don't affect
it: it has no own `this` slot, nothing to rebind.

## Question 4
- **`prototype`** — a property of the **constructor function**: the object that
  becomes the prototype for instances created via `new` (methods go there).
- **`__proto__`** (`[[Prototype]]`) — the reference **of the object/instance
  itself** to its prototype.
Link: `new User()` → `instance.__proto__ === User.prototype`.

## Question 5
The engine looks for `method` on `obj` itself; not finding it — goes via
`obj.__proto__` to the prototype and looks there; not there — to the prototype's
prototype, and so on to `null`. This is the **prototype chain**. The first one found
along the chain is used; if it reaches `null` — `undefined` (or an error on call).

## Question 6
A method on `prototype` is stored **in one place** and **shared** by all instances —
they merely reference the prototype. Assigning `this.method = ...` in the
constructor creates **a new copy of the function per instance** — wasted memory and
no benefit. The prototype = one source for all.

## Question 7
`class` is **syntactic sugar** over prototypal inheritance, not a new model. Under
the hood: methods are put on `Class.prototype`; `extends` links the chain
(`Child.prototype.__proto__ === Parent.prototype`); `super(...)` calls the parent's
constructor/method along that chain. `instanceof` checks for `Class.prototype` in
the object's chain.

## Question 8
Inside `setInterval(function () { this.seconds++ })` the regular function is called
"plainly" → `this` isn't the `Timer` instance (in strict — `undefined`, giving an
error/`NaN`). `seconds` doesn't grow. Fix — an **arrow** that closes over `this`
from `start`:
```ts
start() { setInterval(() => { this.seconds++; }, 1000); }
```
(Or `bind(this)`.) An arrow has no `this` of its own and takes it lexically — from
the `start` method, where `this` is the instance.
