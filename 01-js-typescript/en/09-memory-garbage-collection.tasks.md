# Memory and garbage collection — tasks

Answer in your own words, then check against [09-memory-garbage-collection.answers.md](./09-memory-garbage-collection.answers.md).

> 🌐 Russian version: [09-memory-garbage-collection.tasks.md](../ru/09-memory-garbage-collection.tasks.md)

---

## Question 1
By what criterion does the GC decide an object can be collected? What are the "roots"
(GC roots)?

## Question 2
Describe the mark-and-sweep algorithm in two steps.

## Question 3
Will a pair of objects that reference each other but are tied to nothing else be
collected? Why would reference counting give the wrong answer here?

## Question 4
What is a memory leak in terms of reachability? Give three typical sources of leaks
in JS.

## Question 5
Why is this a leak and how do you fix it?
```ts
const cache = {};
function remember(id, data) { cache[id] = data; }   // and nobody clears it
```

## Question 6
How does `WeakMap` differ from `Map` from the GC's standpoint? In what scenario does
it save you from a leak?

## Question 7
Find the leak and fix it:
```ts
class Widget {
  constructor() {
    this.timer = setInterval(() => this.tick(), 1000);
  }
  tick() { /* ... */ }
  destroy() { /* remove the widget */ }
}
```
