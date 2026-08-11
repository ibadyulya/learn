# Isolation levels — tasks

Answer in your own words, then check against
[en.01-isolation-levels.answers.md](./en.01-isolation-levels.answers.md).

> 🌐 Russian version: [ru.01-isolation-levels.tasks.md](./ru.01-isolation-levels.tasks.md)

---

## Question 1 — name the anomaly
Determine which anomaly this is, and why:
```
T1: SELECT price FROM items WHERE id=7;      -- 100
T2:   UPDATE items SET price=150 WHERE id=7; COMMIT;
T1: SELECT price FROM items WHERE id=7;      -- 150
```

## Question 2 — name the anomaly
```
T1: UPDATE stock SET qty=0 WHERE id=3;   -- NOT committed
T2:   SELECT qty FROM stock WHERE id=3;  -- 0
T1: ROLLBACK;
```
Which anomaly? At what minimum isolation level does it disappear?

## Question 3 — dirty vs non-repeatable
What's the fundamental difference between a dirty read and a non-repeatable read?
Both are about "read the wrong thing", after all.

## Question 4 — non-repeatable vs phantom
How does a phantom read differ from a non-repeatable read? Through which SQL
operations does each arise?

## Question 5 — a level for the task
What isolation level is the minimum sufficient to: (a) never read uncommitted
data; (b) read one row twice within a transaction and be guaranteed the same
value?

## Question 6 — the defaults gotcha
The interviewer: "What is the default isolation level in PostgreSQL and in MySQL?
And is it true that at MySQL's default level phantoms are possible?" Answer.

## Question 7 — the cost of Serializable
Why is Serializable "expensive"? What specifically must an application running at
this level in PostgreSQL do that it doesn't need to at Read Committed?
