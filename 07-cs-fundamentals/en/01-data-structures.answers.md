# Data structures — solutions

> 🌐 Russian version: [01-data-structures.answers.md](../ru/01-data-structures.answers.md)

---

## Question 1
- **Array:** index access **O(1)** (the address is computed), but insertion/deletion in the
  middle **O(n)** (shifting elements).
- **Linked list:** access to the i-th **O(n)** (follow references), but insertion/deletion at
  a known node **O(1)** (relink references).
The choice: need fast access by position/mostly read → **array**; many insertions/deletions
at known spots and no index needed → **list**.

## Question 2
- **Stack — LIFO** (last in — first out). Task: **undo/redo**, the call stack, depth-first
  traversal (DFS), balanced-parentheses checking.
- **Queue — FIFO** (first in — first out). Task: **task scheduling**, a message buffer,
  breadth-first traversal (BFS).

## Question 3
A hash function turns a key into an **array position** directly, without scanning →
access/insertion is O(1) on average. A **collision** is when two different keys give one
position; they're resolved with a list in the slot or probing. With a **bad** hash
function/collision overflow, there are many collisions, and in the **worst case** it all
degenerates into one list → **O(n)**. In practice with a good hash and resizing — O(1)
amortized.

## Question 4
A **BST** stores elements ordered (left subtree < node < right), so search proceeds by
halving — **O(log n)** in a **balanced** tree. If the tree is **unbalanced** (you inserted
already-sorted data), it degenerates into a chain — effectively a linked list, and search
degrades to **O(n)**. So self-balancing trees are used (AVL, red-black, B-tree).

## Question 5
A **heap** — a structure with fast access to the **minimum or maximum** (at the root); the
basis of a **priority queue**. Inserting a new element and extracting the extreme are
**O(log n)** (sifting up/down). Needed where you constantly take the "highest priority":
schedulers, Dijkstra's algorithm, top-K.

## Question 6
A **hash table** (`Map`): the key is the word, the value the counter. Walk the text, for each
word `count[word] = (count[word] ?? 0) + 1` — O(1) average per word, O(n) total for the whole
text. Order doesn't matter, you need fast access by key — ideal for frequency counting.

## Question 7
A plain array is bad: removal from the front (`shift`) is **O(n)** (shifting all elements).
Better — a **queue** based on a linked list or a ring buffer — enqueue at the end and dequeue
from the front in **O(1)**. (In JS for large volumes take a queue structure, not
`array.shift()`.)
