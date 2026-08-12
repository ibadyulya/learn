# Sorting and searching

> 🌐 Russian version: [03-sorting-searching.md](../ru/03-sorting-searching.md)

Basic ordering and searching algorithms. Reads as a story; costs are in terms of
[Big-O](./02-complexity-big-o.md).

The through-line of the topic:

> **Sorting orders data, searching finds what's needed. The key — know that good sorts
> run in **O(n log n)** (faster than naive quadratic ones), and binary search gives
> **O(log n)** on **sorted** data. And understand when a built-in sort is enough rather
> than reinventing your own.**

---

## Simple sorts — O(n²)

Naive, for understanding the principle (rarely used in practice):
- **Bubble** — repeatedly "bubble up" adjacent pairs.
- **Insertion** — insert each element into the sorted part; good on nearly-sorted/small
  data.
- **Selection** — find the minimum and put it at the front.
All — **O(n²)** (a nested loop), slow on large inputs.

---

## Efficient sorts — O(n log n)

- **Merge sort** — "divide and conquer": split in half down to singletons, then **merge**
  the sorted halves. Always **O(n log n)**, **stable**, but needs **O(n) extra memory**.
- **Quick sort** — pick a pivot, partition into "less/greater", recursively sort the parts.
  Average **O(n log n)**, **in-place** (little memory), but **worst case O(n²)** (a bad pivot
  on already-sorted data) — fixed with a random pivot.
- **Heap sort** — via a heap; **O(n log n)**, in-place, not stable.

The common O(n log n) technique — **halving the problem** (log n levels) × **a pass** (n) at
each.

---

## Stability

A sort is **stable** if it preserves the relative order of elements with **equal keys**.
Matters when sorting by multiple criteria (first by name, then stably by age — names within
an age keep their order). Merge is stable, quick/heap aren't.

---

## Binary search — O(log n)

On a **sorted** array: compare with the middle, discard half, repeat.
```
searching for 7 in [1,3,5,7,9,11]: middle 5 < 7 → right half → middle 9 > 7 → ... → found
```
The number of steps = log₂n → **O(log n)**. The requirement — the data **must be sorted**
(otherwise sort first). DB indexes stand on binary search (B-tree, see
[indexing](../../05-databases/en/02-indexing.md)).

---

## When built-in is enough

In practice **don't rewrite sorting by hand** — `Array.prototype.sort` (in V8 — **TimSort**,
a merge+insertion hybrid, stable, O(n log n)) is almost always better. You need to know the
algorithms to: understand complexity, choose a structure, and for interviews. A JS caveat:
`sort()` by default compares **as strings** — for numbers pass a comparator (`a - b`).

---

## Interview phrasing

> "Simple sorts (bubble/insertion/selection) — O(n²), for understanding. The working ones —
> O(n log n): merge (divide-and-conquer, stable, but O(n) memory), quick (in-place, average
> n log n, worst n² with a bad pivot — I take a random one), heap. Stability — preserving the
> order of equal keys, matters for multi-criteria sorting. Binary search — O(log n) on sorted
> data (discard half each step), DB indexes stand on it. In practice I don't rewrite sorting
> — the built-in (TimSort) is stable and O(n log n); in JS I remember the comparator for
> numbers."

---

See [03-sorting-searching.tasks.md](./03-sorting-searching.tasks.md) — tasks. Solutions in
[03-sorting-searching.answers.md](./03-sorting-searching.answers.md).
