# Common algorithms — solutions

> 🌐 Russian version: [04-common-algorithms.answers.md](../ru/04-common-algorithms.answers.md)

---

## Question 1
**Two pointers** — two indices move over a structure (toward each other or at one pace),
replacing pair enumeration. The class of problems: a **sorted array**, a pair/triple with a
condition, a palindrome, a merge. Example — a pair with sum `target` on a sorted array:
pointers from the ends, sum too small → move the left one right, too big → the right one left.
Instead of O(n²) enumeration — **O(n)**.

## Question 2
A **sliding window** — maintain a subrange (window), moving the borders and **updating** the
result incrementally, instead of recomputing from scratch. It saves: naive recomputation of
each window is **O(n·k)** or O(n²), the window is **O(n)**. The signal: "a contiguous
subarray/substring", "a window of length k", "the longest substring with a condition".

## Question 3
- **BFS** — a **queue**, traversal **level by level** (nearest first). Natural for the
  **shortest path** in an unweighted graph, "minimum steps".
- **DFS** — a **stack/recursion**, traversal **in depth**. Natural for "traverse everything",
  connected components, path search, topological sort.
BFS finds the shortest path because it expands **in a wave by levels**: it reaches a node at
distance d **earlier** than a node at d+1 → the first time you reach the target, it's the
minimum number of edges.

## Question 4
**Dynamic programming** applies when: (1) **overlapping subproblems** — the same subproblems
are solved repeatedly (there's something to cache); (2) **optimal substructure** — the
problem's optimum is built from subproblems' optima. Then you remember subproblem solutions
(memoization/tabulation) and don't recompute.

## Question 5
Naive `fib(n) = fib(n-1) + fib(n-2)` spawns a **call tree** where the same `fib(k)` are
computed **many times** (exponentially many) → **O(2ⁿ)**. **Memoization** caches each
`fib(k)`'s result on the first computation; repeat calls take it from the cache in O(1). Each
value is computed **once** → **O(n)** computations total.

## Question 6
A **greedy** algorithm takes the locally best choice at each step. It doesn't always work —
only if the local optimum is guaranteed to lead to the global one (a property you must prove).
It breaks, for example, on **coin change with a non-standard set**: coins {1, 3, 4}, sum 6.
Greedy: 4 + 1 + 1 = 3 coins. Optimal: 3 + 3 = **2 coins**. Here you need **DP**, greed gives a
non-optimum.

## Question 7
The technique — a **sliding window** + a set/hash of characters in the window. Move the right
border, adding characters; on a repeat move the left border, removing characters until the
repeat is gone; the max window length is the answer. Each character is added and removed at
most once → **O(n)** time, O(k) memory (k — alphabet size).
