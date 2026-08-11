# Coupling, cohesion & principles — tasks

Answer in your own words, then check against [06-coupling-cohesion-principles.answers.md](./06-coupling-cohesion-principles.answers.md).

> 🌐 Russian version: [06-coupling-cohesion-principles.tasks.md](../ru/06-coupling-cohesion-principles.tasks.md)

---

## Question 1
Define cohesion and coupling. Which one is maximized, which minimized?

## Question 2
Why are these two metrics almost always named as a pair? How does high cohesion
help low coupling?

## Question 3
A `Utils` class contains `formatDate`, `sendEmail`, `parseCsv`, `hashPassword`.
What problem is this and how do you name it in the topic's terms? How do you fix it?

## Question 4
What is the Law of Demeter? What's wrong with the chain
`order.getCustomer().getAddress().getCity()` and how do you rewrite it?

## Question 5
DRY is often misunderstood. Why is "remove any duplication" dangerous advice? What
does "the wrong abstraction is worse than duplication" mean?

## Question 6
How are YAGNI and over-engineering related? Give an example where a premature
abstraction raises coupling instead of helping.

## Question 7
List 3–4 signs of tight coupling in code that are visible "at a glance".
