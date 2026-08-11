# GoF patterns — tasks

Answer in your own words, then check against [05-design-patterns-gof.answers.md](./05-design-patterns-gof.answers.md).

> 🌐 Russian version: [05-design-patterns-gof.tasks.md](../ru/05-design-patterns-gof.tasks.md)

---

## Question 1 — three families
Name the three families of GoF patterns and, in one sentence each, what problem
each solves.

## Question 2 — recognize the pattern
"I need to add behavior to an object in various combinations, without breeding a
class per combination, keeping the original interface." Which pattern? Give a mini
example.

## Question 3 — tell the similar ones apart
What's the difference between **Adapter**, **Facade** and **Decorator**? All three
"wrap" an object — by what criterion do you tell them apart?

## Question 4 — recognize the pattern
A third-party library logs via `logger.write(level, msg)`, but all your code
expects the interface `Logger.log(msg)`. Which pattern do you apply and how?

## Question 5 — Observer
Describe the Observer pattern in your own words. Give three places in the
JS/Node/frontend ecosystem where it's already implemented out of the box.

## Question 6 — Strategy vs State
How does Strategy differ from State? Both are "a behavior object behind an
interface".

## Question 7 — Command
Why pack an action into a command object? Name two capabilities it unlocks that are
hard to get by calling a function directly.

## Question 8 — Singleton and proportion
Why is Singleton often called an anti-pattern? And why is it rarely needed by hand
in Node/Nest — what replaces it?

## Question 9 — "simpler than the pattern"
For which classic patterns does JS have a simpler idiomatic expression (without
classes)? Give 2–3 examples.
