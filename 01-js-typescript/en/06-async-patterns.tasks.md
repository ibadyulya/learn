# Async — tasks

Answer in your own words, then check against [06-async-patterns.answers.md](./06-async-patterns.answers.md).

> 🌐 Russian version: [06-async-patterns.tasks.md](../ru/06-async-patterns.tasks.md)

---

## Question 1
Why does single-threaded JS need async at all? What happens if you wait for a
network response synchronously?

## Question 2
Name the three promise states. How many times and how can it change state?

## Question 3
Why is a `.then().then().catch()` chain better than nested callbacks? What does the
`.catch` at the end of the chain catch?

## Question 4
Is `async/await` a replacement for promises or a layer on top? What does an async
function return? How is `await` related to `.then` and microtasks?

## Question 5
Speed up this code without changing the fact that the `fetch`es are independent:
```ts
const results = [];
for (const url of urls) {
  results.push(await fetch(url));
}
```

## Question 6
What's the difference between `Promise.all`, `Promise.allSettled`, `Promise.race`
and `Promise.any`? One scenario for each.

## Question 7
Why doesn't this work as expected and how do you fix it?
```ts
items.forEach(async (item) => { await save(item); });
console.log('all saved');
```

## Question 8
What does this print? (link to the event loop)
```ts
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
```
