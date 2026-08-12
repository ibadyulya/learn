# Data structures

> 🌐 Russian version: [01-data-structures.md](../ru/01-data-structures.md)

How to organize data so operations are fast. Reads as a story; operation costs are
formalized by [complexity/Big-O](./02-complexity-big-o.md).

The through-line of the topic:

> **A data structure is a way to organize data for the operations you need. There's no
> universal "best": each structure is fast at one thing and slow at another. Choosing a
> structure = choosing a **trade-off** on the cost of access, search, insertion,
> deletion.**

---

## Array

Elements contiguous in memory, access by index.
- **access by index — O(1)** (the address is computed directly);
- **search by value — O(n)** (a scan);
- **insertion/deletion in the middle — O(n)** (shifting elements);
- insertion at the end — amortized O(1) (a dynamic array).
Good when you need fast access by position and mostly read.

## Linked list

Nodes, each holding a value and a **reference** to the next (no adjacency in memory).
- **access to the i-th — O(n)** (follow the references);
- **insertion/deletion at a known node — O(1)** (relink references);
- no shifting, but poor memory locality.
Good for many insertions/deletions at known spots and no need for index access.

## Stack and queue

Restricted by access discipline:
- **Stack** — **LIFO** (last in — first out): push/pop from one end. Undo history, the
  call stack, depth-first traversal.
- **Queue** — **FIFO** (first in — first out): enqueue/dequeue from different ends. Task
  scheduling, breadth-first traversal.
Both — O(1) on their operations.

## Hash table (hash map)

Key → value via a **hash function** giving a position.
- **insertion/search/deletion — O(1) on average**;
- **no order**; on **collisions** (two keys into one slot) — lists/probing, in the worst
  case degrades to O(n).
The workhorse: dictionaries, caches, deduplication, frequency counting. In JS —
`Map`/`Object`/`Set`.

## Trees and heap

- **Tree** — a hierarchy of nodes; a **binary search tree (BST)** keeps order (left < node
  < right) → search/insertion **O(log n)** when **balanced** (an unbalanced one degenerates
  into a list, O(n)). DB indexes stand on trees (B-tree) (see
  [indexing](../../05-databases/en/02-indexing.md)).
- **Heap** — a tree with priority (min/max at the root): insertion/extraction of the
  extreme — O(log n). The basis of a **priority queue**.

## Graph

Nodes (vertices) + connections (edges). Models networks: social graphs, routes,
dependencies. Stored as an adjacency list (usually) or a matrix. Traversed with BFS/DFS
(see [algorithms](./04-common-algorithms.md)).

---

## Choosing = a trade-off

| Structure | Access | Search | Insert/delete |
|---|---|---|---|
| Array | O(1) by index | O(n) | O(n) in the middle |
| Linked list | O(n) | O(n) | O(1) at a known node |
| Hash table | — | O(1) avg | O(1) avg |
| BST (balanced) | — | O(log n) | O(log n) |
| Heap | O(1) extreme | O(n) | O(log n) |

There's no "best" — there's the right one for the task's operations.

---

## Interview phrasing

> "A data structure is a way to organize data for operations; the choice is a trade-off.
> Array: index access O(1), but insert/search O(n). Linked list: insert/delete O(1) at a
> known node, access O(n). Stack — LIFO, queue — FIFO, O(1). Hash table — O(1) average for
> insert/search, but no order and degradation on collisions — the workhorse for
> dictionaries/caches. A BST keeps order with O(log n) when balanced (DB indexes stand on
> trees), a heap gives O(log n) for priority. A graph — nodes and edges, traversed with
> BFS/DFS. I pick a structure for the task's most frequent operations."

---

See [01-data-structures.tasks.md](./01-data-structures.tasks.md) — tasks. Solutions in
[01-data-structures.answers.md](./01-data-structures.answers.md).
