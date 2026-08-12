# Complexity and Big-O

> 🌐 Russian version: [02-complexity-big-o.md](../ru/02-complexity-big-o.md)

How to assess an algorithm's efficiency independent of hardware. Reads as a story.

The through-line of the topic:

> **Big-O describes how time (or memory) **grows** with the input size `n` —
> asymptotically, dropping constants and lower-order terms. This lets you compare
> algorithms by **scalability**, not by absolute seconds on a specific machine.**

---

## Why not measure in seconds

Seconds depend on the CPU, language, load — incomparable. We care about something else:
**how will the algorithm behave when the data grows 10, 100, 1000 times?** Big-O answers
that by describing the **shape of growth**, not specific numbers. An O(n²) algorithm on a
large input will lose to O(n log n) on any hardware — it's only a question of at what `n`.

---

## What Big-O is

An **upper bound on growth** of a function as `n → ∞`. Two simplification rules:
- **drop constants**: O(2n) → **O(n)**, O(n/2) → O(n);
- **keep the highest term**: O(n² + n) → **O(n²)** (at large `n` the square dominates).

Big-O is about the **trend**, not exact time: constants and small terms don't matter over
distance.

---

## The main classes (ascending)

| Class | Name | Example |
|---|---|---|
| **O(1)** | constant | index access, hash table insertion |
| **O(log n)** | logarithmic | binary search, search in a balanced tree |
| **O(n)** | linear | a pass over an array, linear search |
| **O(n log n)** | linearithmic | good sorts (merge/quick/heap) |
| **O(n²)** | quadratic | a nested loop over all pairs, bubble sort |
| **O(2ⁿ)** | exponential | naive subset enumeration, Fibonacci recursion |

A guide: **O(log n)** and **O(1)** — excellent; **O(n)**, **O(n log n)** — normal;
**O(n²)** — tolerable on small data, bad on large; **O(2ⁿ)** — practically inapplicable.

---

## How to reason about complexity

- **sequential** blocks add up, the highest wins: O(n) + O(n²) = O(n²);
- **nested** loops multiply: a loop within a loop over `n` → O(n²);
- **halving the problem** each step → O(log n) (binary search);
- **a pass + halving** → O(n log n).
You count how many times the "expensive" operation runs as a function of `n`.

---

## Time vs memory

Complexity can be **by time** (how many operations) and **by memory** (how much extra
space). Often it's a **trade-off**: you can speed up at the cost of memory (a cache, a hash
table) or save memory at the cost of time. Always clarify which complexity you mean.

---

## Best / average / worst case and amortization

- one algorithm can have different complexity in the **best/average/worst** case (quicksort
  — O(n log n) average, O(n²) worst);
- **amortized** complexity — the average over a sequence of operations: `push` into a
  dynamic array is usually O(1), but sometimes O(n) (resize) — **amortized O(1)**, because
  the expensive resizes are rare.
In interviews you usually name the **worst** case (a guarantee) unless stated otherwise.

---

## Interview phrasing

> "Big-O describes the growth of time/memory with the input asymptotically: drop constants
> and lower-order terms, look at the trend, to compare algorithms independent of hardware.
> Classes ascending: O(1), O(log n), O(n), O(n log n), O(n²), O(2ⁿ) — logarithm and constant
> excellent, square bad on large data, exponent inapplicable. I reason by structure:
> sequential adds up (the highest wins), nested multiplies, halving gives a logarithm. I
> distinguish time and memory (often a trade-off) and best/average/worst case; amortized is
> the average over a series of operations, like push into a dynamic array. By default I name
> the worst case."

---

See [02-complexity-big-o.tasks.md](./02-complexity-big-o.tasks.md) — tasks. Solutions in
[02-complexity-big-o.answers.md](./02-complexity-big-o.answers.md).
