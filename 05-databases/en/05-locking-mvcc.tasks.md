# Locking and MVCC — tasks

Answer in your own words, then check against [05-locking-mvcc.answers.md](./05-locking-mvcc.answers.md).

> 🌐 Russian version: [05-locking-mvcc.tasks.md](../ru/05-locking-mvcc.tasks.md)

---

## Question 1
What's the difference between pessimistic and optimistic locking? On what assumption
is each built?

## Question 2
How is optimistic locking implemented via a version column? What happens on a conflict?

## Question 3
When do you choose pessimistic vs optimistic locking? What does it depend on?

## Question 4
What is a deadlock? How does the DB detect and resolve it? How do you lower the
likelihood?

## Question 5
Explain MVCC in your own words. What main rule does it provide?

## Question 6
How does MVCC store changes if it doesn't overwrite the row in place? What's the price
(and what does `VACUUM` do in Postgres)?

## Question 7
How are isolation levels (Read Committed, Serializable) related to locking and MVCC?
