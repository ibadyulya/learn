# Iterators and generators — tasks

Answer in your own words, then check against [08-iterators-generators.answers.md](./08-iterators-generators.answers.md).

> 🌐 Russian version: [08-iterators-generators.tasks.md](../ru/08-iterators-generators.tasks.md)

---

## Question 1
Describe the iteration protocol: how does an "iterable" differ from an "iterator"?
What does `next()` return?

## Question 2
Which language constructs use this protocol "under the hood"? Name at least four.

## Question 3
Why can't you traverse a plain object `{a:1, b:2}` with `for...of`, while an array
you can? How do you iterate an object anyway?

## Question 4
What do `function*` and `yield` do? How does a generator's execution differ from an
ordinary function's?

## Question 5
Implement a generator `range(from, to, step)` that yields numbers with a step.

## Question 6
How does a generator let you describe an **infinite** sequence without hanging the
runtime? Give an example and explain why it doesn't loop forever.

## Question 7
Make an object `range = {from, to}` iterable **via a generator** in
`[Symbol.iterator]`.

## Question 8
What are async iterators and when do you need them? Which loop iterates them?
