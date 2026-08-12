# Query optimization — tasks

Answer in your own words, then check against [06-query-optimization.answers.md](./06-query-optimization.answers.md).

> 🌐 Russian version: [06-query-optimization.tasks.md](../ru/06-query-optimization.tasks.md)

---

## Question 1
What does the query planner do and based on what does it pick a plan? Why do stale
statistics hurt?

## Question 2
What's the difference between `EXPLAIN` and `EXPLAIN ANALYZE`? What does a `Seq Scan`
of a big table mean?

## Question 3
Name three things you look for in the plan when diagnosing a slow query.

## Question 4
What is N+1 queries from the app side? How do you remove it?

## Question 5
Why can `SELECT *` hinder optimization? What's better?

## Question 6
Describe the correct order of actions when optimizing. Why is "add indexes at random"
bad?

## Question 7
A query with `ORDER BY created_at LIMIT 20 OFFSET 100000` gets slower toward the end.
Why and how do you fix it?
