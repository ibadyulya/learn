# ACID and transactions — tasks

Answer in your own words, then check against [04-acid-transactions.answers.md](./04-acid-transactions.answers.md).

> 🌐 Russian version: [04-acid-transactions.tasks.md](../ru/04-acid-transactions.tasks.md)

---

## Question 1
What is a transaction? Give an example where breaking atomicity leads to trouble.

## Question 2
Spell out ACID: what does each letter guarantee?

## Question 3
How does atomicity work at the mechanism level? What happens on `ROLLBACK`?

## Question 4
What is durability and how does WAL (write-ahead log) provide it? Why write to the
journal BEFORE applying to the data?

## Question 5
How does the "C" in ACID differ from the "C" in the CAP theorem?

## Question 6
What is a savepoint and what's it for?

## Question 7
Why does ACID "break" when an operation touches the DB + an external broker? What's
used instead of a simple transaction?
