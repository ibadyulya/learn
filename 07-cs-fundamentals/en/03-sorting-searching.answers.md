# Sorting and searching — solutions

> 🌐 Russian version: [03-sorting-searching.answers.md](../ru/03-sorting-searching.answers.md)

---

## Question 1
Simple sorts, for each element, pass over the rest (a nested loop) → **n × n = O(n²)**. The
efficient ones use **"divide and conquer"**: they split the array in half — that's **log n**
levels (how many times to halve `n` down to 1), and at each level they process all elements —
**n**. Total **n × log n = O(n log n)**. The `log n` comes from **halving**.

## Question 2
- **Merge sort:** average and worst — **O(n log n)** (consistently fast), **stable**, but
  needs **O(n) extra memory** (a merge buffer).
- **Quick sort:** average — **O(n log n)**, **worst — O(n²)** (a bad pivot); **in-place**
  (O(log n) memory for the recursion stack), **not stable**.
Merge is more predictable and stable at the cost of memory; quick saves memory and is fast in
practice.

## Question 3
**Stability** — preserving the relative order of elements with **equal keys**. Matters for
**sorting by multiple criteria**: sort by age, then stably by city — within one city the
age order is preserved. Without stability the second sort "shuffles" equal keys.

## Question 4
Binary search at each step compares with the middle and **discards half** the remaining
range; the number of steps down to one element = **log₂n** → **O(log n)**. The mandatory
condition — the data is **sorted** (otherwise "discard half" is incorrect); if not — sort
first (O(n log n)).

## Question 5
Quick sort's worst **O(n²)** occurs with a **bad pivot choice** — when it regularly turns out
to be the min/max (e.g., on an **already-sorted** array with pivot = the first element):
partitioning degenerates into "1 and n-1", recursion depth `n`. Avoid it: a **random pivot**,
"median of three", shuffle the input, or use introsort (switch to heap sort on deep
recursion).

## Question 6
No, in practice you **don't write your own** — the built-in is almost always better and
tested. `Array.prototype.sort` in V8 uses **TimSort** — a hybrid of merge sort and insertion
sort: stable, **O(n log n)**, very efficient on real (partially-ordered) data. Your own
implementations — for study/interviews/special requirements.

## Question 7
`[10, 2, 1].sort()` without a comparator compares elements **as strings** (lexicographically):
`"10" < "2"` because comparison is character-by-character and `"1" < "2"`. Fix — pass a
**numeric comparator**:
```js
[10, 2, 1].sort((a, b) => a - b);   // [1, 2, 10]
```
