# GRASP — tasks

Answer in your own words, then check against [04-grasp.answers.md](./04-grasp.answers.md).

> 🌐 Russian version: [04-grasp.tasks.md](../ru/04-grasp.tasks.md)

---

## Question 1
What single question do all GRASP principles answer? How does GRASP differ from GoF
patterns in terms of level?

## Question 2
By Information Expert, who should be assigned computing an order's total, and why?
What's wrong with extracting it into a separate `OrderCalculator` that pulls all
the data out?

## Question 3
What is Pure Fabrication? Give a typical backend example and explain which two
metrics it's introduced for.

## Question 4
Given code with `switch (payment.type)` in three places for three payment methods.
Which GRASP principle suggests a solution, and which solution?

## Question 5
What is Protected Variations? How does it relate to DIP and OCP? Give an example of
a "seam".

## Question 6
The Controller role: why shouldn't the UI itself or a domain object receive a
business operation? Where's the Controller in a NestJS app?

## Question 7
How do Information Expert and High Cohesion together echo the SRP principle from
SOLID?
