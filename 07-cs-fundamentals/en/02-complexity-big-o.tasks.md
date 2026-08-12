# Complexity and Big-O — tasks

Answer in your own words, then check against [02-complexity-big-o.answers.md](./02-complexity-big-o.answers.md).

> 🌐 Russian version: [02-complexity-big-o.tasks.md](../ru/02-complexity-big-o.tasks.md)

---

## Question 1
Why assess an algorithm in Big-O rather than in seconds?

## Question 2
Simplify by the Big-O rules: O(2n + 5), O(n² + n), O(n/2), O(100).

## Question 3
Order ascending: O(n log n), O(1), O(2ⁿ), O(n), O(log n), O(n²). Give an example of each.

## Question 4
What's the complexity of this code and why?
```js
for (let i = 0; i < n; i++)
  for (let j = 0; j < n; j++)
    doSomething(i, j);
```

## Question 5
Why is binary search O(log n)? What exactly happens at each step?

## Question 6
What is amortized complexity? Why is `push` into a dynamic array amortized O(1) even though
it's sometimes O(n)?

## Question 7
What's the difference between time and space complexity? Give a trade-off example.
