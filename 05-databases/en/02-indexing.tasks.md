# Indexing — tasks

Answer in your own words, then check against [02-indexing.answers.md](./02-indexing.answers.md).

> 🌐 Russian version: [02-indexing.tasks.md](../ru/02-indexing.tasks.md)

---

## Question 1
Why do you need an index? How does it change search complexity and by what means?

## Question 2
Why is a B-tree index better than a hash index? What can one do that the other can't?

## Question 3
Given `INDEX(country, city)`. Will it work for `WHERE city='NY'`? Explain the
leftmost-prefix rule.

## Question 4
What is a covering index (index-only scan) and why is it faster than an ordinary
index lookup?

## Question 5
Indexes aren't free. Name two costs and explain why "an index on every column" is bad.

## Question 6
Why won't an index on `email` help the query `WHERE LOWER(email) = 'a@b.com'`? How do
you fix it?

## Question 7
Name three more situations where an index exists but the DB does a full scan.
