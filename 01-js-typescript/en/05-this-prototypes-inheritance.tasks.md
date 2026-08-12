# `this`, prototypes and inheritance — tasks

Answer in your own words, then check against [05-this-prototypes-inheritance.answers.md](./05-this-prototypes-inheritance.answers.md).

> 🌐 Russian version: [05-this-prototypes-inheritance.tasks.md](../ru/05-this-prototypes-inheritance.tasks.md)

---

## Question 1
What does `this` depend on — the declaration site or the call site? Name the four
rules for determining `this`.

## Question 2
What does this print and why? How do you fix it to output `'Ann'`?
```ts
const user = { name: 'Ann', hi() { return this.name; } };
const fn = user.hi;
console.log(fn());
```

## Question 3
How does an arrow function differ from a regular one regarding `this`? Why can't an
arrow be rebound with `bind`?

## Question 4
What's the difference between `prototype` and `__proto__`? Which belongs to what?

## Question 5
Describe what happens on `obj.method()` if `obj` has no own `method`. What is the
prototype chain?

## Question 6
Why are methods put on `Constructor.prototype` rather than assigned in the
constructor (`this.method = ...`)? What's the benefit?

## Question 7
Is `class` in JS a fundamentally new thing or sugar? What do `extends` and `super`
do under the hood?

## Question 8
What does this print and why?
```ts
class Timer {
  seconds = 0;
  start() { setInterval(function () { this.seconds++; }, 1000); }
}
new Timer().start();
```
How do you fix it?
