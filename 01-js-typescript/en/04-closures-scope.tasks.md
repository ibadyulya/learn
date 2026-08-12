# Closures and scope — tasks

Answer in your own words, then check against [04-closures-scope.answers.md](./04-closures-scope.answers.md).

> 🌐 Russian version: [04-closures-scope.tasks.md](../ru/04-closures-scope.tasks.md)

---

## Question 1
What is lexical scope? Does a function take variables from the call site or the
declaration site?

## Question 2
Define a closure in one or two sentences. Why doesn't `count` in `makeCounter`
disappear after the function returns?

## Question 3
What does this print and why? How do you get `0, 1, 2` without changing the
`setTimeout`?
```ts
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);
}
```

## Question 4
Implement `once(fn)` — a wrapper that calls `fn` only on the first call and then
returns the cached result. Where's the closure here?

## Question 5
How do you make a "private" variable inaccessible from outside using a closure?
Give a mini module-pattern example.

## Question 6
Each call to `makeCounter()` gives an independent counter. Why don't they share
`count`?

## Question 7
How are closures related to memory leaks? Give an example where a closure
accidentally keeps a large object alive.
