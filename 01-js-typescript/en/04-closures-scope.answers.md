# Closures and scope — solutions

> 🌐 Russian version: [04-closures-scope.answers.md](../ru/04-closures-scope.answers.md)

---

## Question 1
**Lexical** scope — determined by where the function is **written** in the code. A
function takes variables from the **declaration site**, not the call site: not
found in itself — it walks up the chain to the function it's declared inside, and so
on to the global one.

## Question 2
A closure is a **function together with the environment it was created in**: it
keeps access to outer variables even after the producing function finished. `count`
doesn't disappear because the returned function **holds a reference** to it — with a
reference present, the garbage collector won't reclaim the variable, even though
`makeCounter`'s frame is already popped.

## Question 3
It prints **`3, 3, 3`**. `var i` is one function-scoped variable for the whole loop;
all three callbacks closed over the same `i`, and by the time the timers fire the
loop has finished and `i === 3`. To get `0,1,2` without touching `setTimeout`:
- replace `var` with **`let`** (per-iteration binding), or
- wrap in an IIFE passing `i` by copy: `(j => setTimeout(() => console.log(j)))(i)`.

## Question 4
```ts
function once<T>(fn: () => T): () => T {
  let called = false, result: T;
  return () => {
    if (!called) { called = true; result = fn(); }
    return result;
  };
}
```
The closure is the inner function: it remembers `called` and `result` between calls,
even though `once` returned long ago. That's private persistent state.

## Question 5
```ts
const bank = (() => {
  let balance = 0;                       // private, not visible outside
  return {
    deposit: (n: number) => (balance += n),
    getBalance: () => balance,
  };
})();
bank.deposit(100);
bank.getBalance(); // 100
// bank.balance === undefined — no direct access
```
`balance` lives only in the closure; only methods are exposed. This is the module
pattern — encapsulation via IIFE + closure.

## Question 6
Each call to `makeCounter()` creates a **new frame** with its own `count` variable
and a new closure over it. The returned functions close over **different** `count`
variables, so the counters are independent. They'd share state only if `count` were
declared outside, common to all.

## Question 7
A closure holds outer variables from garbage collection. Example:
```ts
function attach() {
  const huge = new Array(1_000_000).fill('x');  // a heavy object
  document.getElementById('btn')!.addEventListener('click', () => {
    console.log(huge.length);   // the callback closed over huge
  });
}
```
As long as the listener is attached, the closure holds a reference to `huge`, and
the million elements aren't collected — even if no longer needed. The cure: remove
the listener (`removeEventListener`) or don't close over the heavy thing.
