# Common algorithms and techniques

> 🌐 Russian version: [04-common-algorithms.md](../ru/04-common-algorithms.md)

A set of techniques that cover whole classes of interview problems. Reads as a story.

The through-line of the topic:

> **Most algorithmic problems are solved by a few **typical techniques**. The skill isn't
> memorizing solutions but **recognizing the pattern** of a problem and applying the right
> technique: two pointers, sliding window, BFS/DFS, dynamic programming, greedy.**

---

## Two pointers

Two indices move over a structure (toward each other or at one pace), replacing a nested
loop. On a **sorted** array, find a pair summing to `target`:
```js
let l = 0, r = arr.length - 1;
while (l < r) {
  const sum = arr[l] + arr[r];
  if (sum === target) return [l, r];
  sum < target ? l++ : r--;      // move the needed pointer
}
```
It would be O(n²) by enumerating pairs — now it's **O(n)**. The sign: a sorted array, a
pair/triple, a palindrome, a merge.

## Sliding window

Maintain a "window" (a subrange), moving the borders instead of recomputing from scratch.
For **substring/subarray** problems: the longest substring without repeats, the max sum of a
window of length k.
```js
// max sum of a window of length k — O(n) instead of O(n*k)
let sum = 0, max = -Infinity;
for (let i = 0; i < arr.length; i++) {
  sum += arr[i];
  if (i >= k) sum -= arr[i - k];   // slid the window: added the new, removed the old
  if (i >= k - 1) max = Math.max(max, sum);
}
```
The sign: "a contiguous subrange", "a window", "a substring with a condition".

## BFS and DFS — traversals

Traversing a graph/tree (see [data structures](./01-data-structures.md)):
- **BFS (breadth-first)** — level by level, via a **queue**. Finds the **shortest path** in
  an unweighted graph. "Nearest first".
- **DFS (depth-first)** — deep until stuck, via a **stack**/recursion. Traverse everything,
  find connected components, topological sort, path search.
Both — **O(V + E)** (vertices + edges). The sign: a graph, tree, maze, "is it reachable",
"the shortest number of steps".

## Dynamic programming (DP)

When a problem breaks into **overlapping subproblems** and has **optimal substructure** (the
whole optimum is built from the optima of parts). The technique — **remember** subproblem
solutions:
- **memoization** (top-down): recursion + a cache;
- **tabulation** (bottom-up): fill a table iteratively.
The classic — Fibonacci: naive recursion **O(2ⁿ)** (recomputes the same thing), with
memoization — **O(n)**. Also: knapsack, coin change, edit distance, grid paths. The sign:
"how many ways", "min/max under choices", "optimally".

## Greedy algorithms

At each step take the **locally best** choice hoping for a global optimum. Fast and simple,
**but doesn't always work** — only when the local optimum leads to the global one (you need
to prove/check it). An example where it works: coin change with a canonical system (take the
largest ≤ the remainder); where it **doesn't**: some non-standard coin sets — there you need
DP. The sign: "maximize/minimize", "scheduling", intervals.

---

## How to recognize the technique

| Signal in the problem | Likely technique |
|---|---|
| a sorted array, a pair/triple | two pointers |
| a contiguous subrange/substring with a condition | sliding window |
| a graph/tree, the shortest path (unweighted) | BFS |
| traverse everything, components, paths | DFS |
| "how many ways", optimum under choices, overlaps | dynamic programming |
| a locally-greedy choice leads to the optimum | greedy |

---

## Interview phrasing

> "I solve problems by recognizing the pattern. Two pointers — replace a nested loop on
> sorted data (a pair with a sum) → O(n). Sliding window — for contiguous subranges/
> substrings, move the borders instead of recomputing. BFS (queue) — the shortest path in an
> unweighted graph, DFS (stack/recursion) — traverse everything/find components, both O(V+E).
> Dynamic programming — when there are overlapping subproblems and optimal substructure:
> memoization or tabulation turn an exponent into a polynomial (Fibonacci 2ⁿ → n). Greedy —
> the locally best choice, fast, but only when it leads to the global optimum. First I
> recognize the problem's signal, then apply the technique."

---

See [04-common-algorithms.tasks.md](./04-common-algorithms.tasks.md) — tasks. Solutions in
[04-common-algorithms.answers.md](./04-common-algorithms.answers.md).
