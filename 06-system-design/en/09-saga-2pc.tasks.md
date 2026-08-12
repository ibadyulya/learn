# Saga and 2PC — tasks

Answer in your own words, then check against [09-saga-2pc.answers.md](./09-saga-2pc.answers.md).

> 🌐 Russian version: [09-saga-2pc.tasks.md](../ru/09-saga-2pc.tasks.md)

---

## Question 1
Why can't you just wrap the "payment + reserve + order" operation across three services
in an ACID transaction?

## Question 2
Describe the two phases of 2PC. What are its three main downsides?

## Question 3
How does a saga work? What happens on the failure of one of the steps?

## Question 4
Why is a compensation not the same as a rollback? Give a money example.

## Question 5
Orchestration vs choreography of a saga: what's the difference and the trade-off?

## Question 6
What consistency does a saga give? What does that mean for intermediate states visible
to the user?

## Question 7
Why must saga steps be idempotent?
