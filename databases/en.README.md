# Databases — navigation

The database area. One topic = one file (`NN-topic.md`), with `.tasks.md` and
`.answers.md` alongside for self-checks. The overall plan — [../en.PLAN.md](../en.PLAN.md).

> 🌐 Russian version: [ru.README.md](./ru.README.md)

A topic about **a single DB** (transactions, isolation, locks), in contrast to
the distributed queues in [../system-design/](../system-design/). Replication and
sharding sit on the boundary — we cover them once and cross-reference.

---

| # | Topic | Tasks | Status |
|---|---|---|---|
| 01 | [Isolation levels](./en.01-isolation-levels.md) | [tasks](./en.01-isolation-levels.tasks.md) · [answers](./en.01-isolation-levels.answers.md) | ✅ |

## Planned (as needed)
- **Locks** — pessimistic vs optimistic, `SELECT ... FOR UPDATE`, deadlocks.
- **MVCC** — how Postgres/InnoDB give snapshots without blocking reads.
- **Indexes** — B-tree, covering indexes, when an index isn't used.
- **Replication / sharding** — on the boundary with system-design (data scaling).

The through-line of the whole area: transactions from
[system-design/03-reliability-patterns](../system-design/en.03-reliability-patterns.md)
(atomic commit, rollback) — that's the foundation on which isolation sits here.
