# Recursion and backtracking

> 🌐 Russian version: [05-recursion.md](../ru/05-recursion.md)

How to solve a problem by reducing it to itself. Reads as a story; on the call stack see
the [event loop](../../01-js-typescript/en/01-event-loop.md).

The through-line of the topic:

> **Recursion — a function that calls itself, reducing the problem to a **smaller version
> of the same problem** until it reaches a trivial case. Two things are needed: a **base
> case** (when to stop) and a **recursive step** (how to shrink the problem). It works via
> the call stack; backtracking is recursion that tries options and undoes them.**

---

## The two mandatory elements

Any correct recursion consists of:
1. A **base case** — a stopping condition where the answer is known without recursion.
2. A **recursive step** — reduce to a **smaller** instance of the same problem.
```js
function factorial(n) {
  if (n <= 1) return 1;            // base case — stop
  return n * factorial(n - 1);    // step — the problem is smaller by 1
}
```
**Without a base case** (or if the step doesn't shrink the problem) — infinite recursion →
stack overflow.

---

## The call stack

Each function call pushes a **frame** onto the call stack (see the
[event loop](../../01-js-typescript/en/01-event-loop.md)); on return the frame is popped.
Recursion grows the stack for each level of nesting:
```
factorial(3) → factorial(2) → factorial(1)   ← stack of depth 3
             ←     2        ←     1           ← unwinding back
```
Hence the limit: **too deep recursion → stack overflow** (the stack is finite). For very
deep/large `n` you sometimes rewrite to iteration or use tail recursion (where the engine may
not grow the stack — but in JS that's unreliably supported).

---

## Recursion vs iteration

Any recursion can be expressed as a loop and vice versa (a loop + an explicit stack). What to
choose:
- **recursion** is cleaner for **tree-like/divisible** problems: tree traversal,
  "divide and conquer" (merge sort), DFS (see [algorithms](./04-common-algorithms.md)) — the
  code mirrors the problem's structure;
- **iteration** is simpler and without overflow risk for linear problems (array sum, a simple
  counter).
Rule: **the problem's structure is tree-like/recursive → recursion; linear → a loop.**

---

## Backtracking

A recursive technique of "try — go deeper — undo if a dead end". You build the solution step
by step; at each step you try options, recurse further, and if a branch doesn't lead to a
solution — you **undo the choice** (backtrack) and try the next.
```
for each option at the current step:
  choose the option
  if the solution is complete → record it
  else → recurse to the next step
  undo the choice (backtrack)
```
The classics: permutations/combinations, sudoku, N queens, maze pathfinding. Essentially it's
**DFS over the decision tree** with undo. Often exponential in time (brute force), but with
branch cutting (pruning) it's practical.

---

## The link to dynamic programming

Recursion with **overlapping subproblems** + **memoization** = top-down dynamic programming
(see [algorithms](./04-common-algorithms.md)). Naive Fibonacci recursion recomputes the same
thing (**O(2ⁿ)**); add a cache and it becomes **O(n)**. So recursion is the skeleton,
memoization turns it from brute-force into efficient.

---

## Interview phrasing

> "Recursion — a function reducing the problem to a smaller version of itself; a base case
> (stop) and a recursive step (shrinking) are mandatory, or you get a stack overflow. Each
> call is a frame on the call stack, so too deep recursion crashes with a stack overflow —
> for linear problems I take iteration, recursion for tree-like/divisible ones (tree
> traversal, divide-and-conquer, DFS). Backtracking — recursive brute force "chose → went
> deeper → undid on a dead end" (DFS over the decision tree): permutations, N queens, sudoku.
> Recursion with overlapping subproblems + memoization = top-down DP, turning an exponent into
> a polynomial (Fibonacci 2ⁿ → n)."

---

See [05-recursion.tasks.md](./05-recursion.tasks.md) — tasks. Solutions in
[05-recursion.answers.md](./05-recursion.answers.md).
