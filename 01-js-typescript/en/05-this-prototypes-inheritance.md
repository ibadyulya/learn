# `this`, prototypes and inheritance

> 🌐 Russian version: [05-this-prototypes-inheritance.md](../ru/05-this-prototypes-inheritance.md)

Two topics that in JS don't work like in classic OOP languages and therefore
intimidate. Reads as a story; both reduce to simple rules.

The through-line of the topic:

> **`this` is determined by HOW a function is called (the call site), not where
> it's declared — the opposite of closures. And inheritance in JS isn't about
> classes but about prototypes: an object borrows properties from another object
> along a chain, and `class` is syntactic sugar over that.**

---

## Part 1. `this` — the call site decides

The main rule: `this` isn't bound to a function forever — it's determined **at call
time**. Four rules by priority (lowest to highest):

**1. Plain call (default).** `fn()` on its own → `this` is `undefined` (strict mode)
or the global object (non-strict).
```ts
function f() { return this; }
f();   // undefined in strict
```

**2. Method call (implicit).** `obj.method()` → `this` is the object **left of the
dot**.
```ts
const user = { name: 'Ann', hi() { return this.name; } };
user.hi();   // 'Ann' — this === user
```

**3. Explicit binding.** `call` / `apply` / `bind` set `this` by hand.
```ts
function hi() { return this.name; }
hi.call({ name: 'Bob' });    // 'Bob'
const bound = hi.bind({ name: 'Cid' });
bound();                     // 'Cid' — bind returns a function with a fixed this
```

**4. Call with `new` (constructor).** `new Fn()` creates a new object and makes it
`this`.
```ts
function User(name: string) { this.name = name; }
new User('Dan').name;   // 'Dan'
```

**Arrow functions — the exception and the rescue.** They have **no `this` of their
own** — they take it **lexically**, from the declaration site (like an ordinary
variable). They can't be rebound with `call`/`bind`.
```ts
const obj = {
  name: 'Eve',
  greet() { return [1].map(() => this.name); }   // the arrow takes this from greet → 'Eve'
};
```

### The classic bug: losing `this`
```ts
const user = { name: 'Ann', hi() { return this.name; } };
const fn = user.hi;
fn();                        // undefined — a "plain" call, this is lost
setTimeout(user.hi, 0);      // also loses this
```
A method torn from its object is called "plainly" → rule 1. The cure: `bind`
(`user.hi.bind(user)`), an arrow wrapper (`() => user.hi()`), or an arrow method in
a class (`hi = () => ...`, closes over the instance's `this`).

---

## Part 2. Prototypes — how inheritance works

JS has **no classes in the classic sense.** It has **prototypal inheritance**:
every object has a hidden reference `[[Prototype]]` (accessible as `__proto__`) to
**another object** — its prototype. Properties are looked up like this:

> Accessing `obj.x` → look for `x` on `obj` itself; not there — go via `__proto__`
> to the prototype, then the prototype's prototype, and so on to `null`. This is the
> **prototype chain**.

```ts
const animal = { eats: true };
const dog = Object.create(animal);   // dog.__proto__ === animal
dog.barks = true;
dog.eats;   // true — not its own, taken from the prototype
dog.barks;  // true — its own
```
Don't confuse two words:
- **`prototype`** — a property of the **constructor function**: the object that
  becomes the prototype for instances created via `new`.
- **`__proto__`** (`[[Prototype]]`) — the reference **of an object itself** to its
  prototype.

```ts
function User(name: string) { this.name = name; }
User.prototype.hi = function () { return this.name; };   // method on the prototype
const u = new User('Ann');
u.hi();                       // 'Ann' — found hi in User.prototype along the chain
u.__proto__ === User.prototype;   // true
```
Why put a method on `prototype` rather than in the constructor: then it's stored
**in one place** and shared by all instances, not copied into each — saving memory.

---

## `class` — sugar over prototypes

`class` in JS doesn't introduce a new model — it's **syntactic sugar** over the same
prototypal mechanism:
```ts
class User {
  constructor(public name: string) {}
  hi() { return this.name; }       // effectively put on User.prototype
}
class Admin extends User {          // extends links the prototype chain
  constructor(name: string) { super(name); }
}
```
`hi` lives on `User.prototype`; `extends` sets `Admin.prototype.__proto__ ===
User.prototype`; `super` calls the parent's method along the chain. Understanding
this, you're not surprised why `this` in a method can be "lost" (a method is an
ordinary function on the prototype) and why `instanceof` is a check of "is
`X.prototype` in the chain".

---

## Interview phrasing

> "`this` in JS is determined by the call site, not the declaration: a plain call →
> undefined/global, `obj.method()` → this = obj, `call/apply/bind` set it
> explicitly, `new` → a new object. Arrow functions have no `this` of their own —
> they take it lexically, and can't be rebound, which makes them handy for fixing
> 'lost this' in callbacks. Inheritance in JS is prototypal: an object has a
> reference to a prototype object, properties are looked up up the prototype chain
> to null. `prototype` is a constructor's property (the future prototype of
> instances), `__proto__` is an object's own reference. Methods go on the prototype
> to be shared. `class` is sugar over all of this."

---

See [05-this-prototypes-inheritance.tasks.md](./05-this-prototypes-inheritance.tasks.md)
— tasks. Solutions in [05-this-prototypes-inheritance.answers.md](./05-this-prototypes-inheritance.answers.md).
