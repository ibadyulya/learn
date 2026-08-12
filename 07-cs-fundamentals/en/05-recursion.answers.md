# Recursion and backtracking — solutions

> 🌐 Russian version: [05-recursion.answers.md](../ru/05-recursion.answers.md)

---

## Question 1
1. A **base case** — a stopping condition where the answer is known without recursion
   (`n <= 1`).
2. A **recursive step** — reduction to a **smaller** instance of the same problem.
Without a base case (or if the step doesn't shrink the problem) the recursion **never stops**
→ infinite calls → **stack overflow**.

## Question 2
Each call pushes a **frame** onto the call stack (arguments, local variables, the return
point); a nested call pushes another frame — the stack grows with the recursion depth. On
return the frames are popped, and the results "unwind" back. The **stack is finite**, so too
deep recursion (e.g., `n` = hundreds of thousands) exhausts it → **stack overflow**.

## Question 3
Yes, any recursion can be expressed iteratively (with an explicit stack if needed) and vice
versa. The choice:
- **Recursion** — when the problem's structure is **tree-like/divisible** (tree/graph DFS
  traversal, divide-and-conquer): the code mirrors the structure and reads more clearly.
- **Iteration** — for **linear** problems (sum, counter) and when protection from stack
  overflow / maximum performance matters.

## Question 4
**Backtracking** — recursive brute force: build the solution step by step, at each step try an
option, go deeper; if a branch hits a dead end — **undo the last choice** and try the next.
```
choose an option → if a complete solution, record it; else recurse further → undo the choice
```
It's **DFS over the decision tree** with backtracking. Problems: permutations, N queens,
sudoku, maze path. With cutting off obviously-bad branches (pruning) it's practical, though in
the general case exponential.

## Question 5
Recursion sets up the **skeleton** (a problem via subproblems). If the subproblems **overlap**
(the same ones are computed many times), we add **memoization** (a result cache) — that's
**top-down dynamic programming**. It turns brute-force recursion into efficient: Fibonacci with
naive recursion is **O(2ⁿ)**, with memoization — **O(n)**.

## Question 6
```js
function sumTree(node) {
  if (!node) return 0;                                   // base case
  return node.value + sumTree(node.left) + sumTree(node.right);  // step: subtrees
}
```
Recursion is natural because **a tree is itself recursive**: a node = value + left subtree +
right subtree, and the function literally mirrors this structure. Iteratively you'd have to
maintain a stack by hand.

## Question 7
The cause — **overlapping subproblems**: `fib(n)` recomputes the same `fib(k)` exponentially
many times (**O(2ⁿ)**), and for `fib(50)` that's billions of calls. Fix:
1. **Memoization** (top-down): cache each `fib(k)`'s result → **O(n)**;
2. **Tabulation/iteration** (bottom-up): compute `fib` in a loop, keeping the last two values →
   **O(n)** time, **O(1)** memory.
