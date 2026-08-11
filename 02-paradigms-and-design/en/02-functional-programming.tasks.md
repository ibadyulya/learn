# Functional programming — tasks

Answer in your own words, then check against [02-functional-programming.answers.md](./02-functional-programming.answers.md).

> 🌐 Russian version: [02-functional-programming.tasks.md](../ru/02-functional-programming.tasks.md)

---

## Question 1
Define a pure function (two conditions). Why do testability, cacheability and safe
parallelism follow from purity?

## Question 2
Which of these functions are pure and which aren't, and why?
```ts
const a = (x: number) => x * 2;
const b = (arr: number[]) => arr.sort();
const c = () => Date.now();
const d = (arr: number[]) => [...arr].sort();
```

## Question 3
Why is immutability important for React/Redux reactivity? What breaks if you mutate
state directly (`state.items.push(x)`)?

## Question 4
Rewrite the imperative code declaratively (no loop, no mutation):
```ts
let result = [];
for (const u of users) {
  if (u.active) result.push(u.name.toUpperCase());
}
```

## Question 5
What are currying and partial application? Give an example where it's genuinely
convenient.

## Question 6
"FP forbids side effects" — why is that wrong? How does FP actually deal with
effects?

## Question 7
Implement `compose(f, g, h)` that applies functions right to left:
`compose(f, g, h)(x) === f(g(h(x)))`.
