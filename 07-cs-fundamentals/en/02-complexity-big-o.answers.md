# Complexity and Big-O — solutions

> 🌐 Russian version: [02-complexity-big-o.answers.md](../ru/02-complexity-big-o.answers.md)

---

## Question 1
Seconds depend on the **hardware, language, load** — incomparable and changeable. Big-O
describes **how** time grows with the input (the shape of growth), which lets you compare
algorithms **by scalability** independent of the machine: O(n²) at large `n` loses to
O(n log n) on any hardware.

## Question 2
- O(2n + 5) → **O(n)** (constants and addends dropped);
- O(n² + n) → **O(n²)** (the highest term);
- O(n/2) → **O(n)**;
- O(100) → **O(1)** (a constant).

## Question 3
Ascending: **O(1)** (index access) < **O(log n)** (binary search) < **O(n)** (a pass over an
array) < **O(n log n)** (merge sort) < **O(n²)** (a nested loop over pairs) < **O(2ⁿ)**
(enumerating all subsets).

## Question 4
**O(n²)** — quadratic. The outer loop runs `n` times, the inner for each also `n` times, so
`doSomething` is called `n × n = n²` times. Nested loops **multiply**.

## Question 5
Binary search works on a **sorted** array: at each step it compares with the middle and
**discards half** of the remaining range. The number of steps = how many times you can halve
`n` down to 1 = **log₂n**, hence **O(log n)**. Each step shrinks the problem by half — the
classic sign of a logarithm.

## Question 6
**Amortized** complexity — the average cost of an operation **over a long series**. A dynamic
array **doubles** its capacity when full: that's a one-time copy at O(n), but it happens
**rarely** (after many cheap O(1) insertions). If you spread the cost of resizes over all
insertions, each gets a **constant** → **amortized O(1)**, even though an individual insertion
is occasionally O(n).

## Question 7
- **Time** complexity — how many **operations** (time) as a function of `n`.
- **Space** — how much **extra memory** beyond the input.
Often a **trade-off**: e.g., caching results in a hash table — speeds up from O(n²) to O(n) in
time, but spends O(n) extra memory. Or traversal without extra memory (O(1) space) at the cost
of more time.
