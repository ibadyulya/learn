# Validation and error handling — tasks

Answer in your own words, then check against [05-validation-error-handling.answers.md](./05-validation-error-handling.answers.md).

> 🌐 Russian version: [05-validation-error-handling.tasks.md](../ru/05-validation-error-handling.tasks.md)

---

## Question 1
Why is validation done "at the boundary" rather than in business logic? Where does it
happen in Nest?

## Question 2
How do expected (operational) errors differ from unexpected (programmer errors)?
Classify: "email taken", "no rights", "Cannot read property of undefined", "DB
timeout".

## Question 3
Why is centralized handling better than a `try/catch` in every handler? How is it done
in Nest and in Express?

## Question 4
What must you not return to the client in an error response and why? Where do those
details go?

## Question 5
What is "fail fast" and why is it better than dragging incorrect data deep?

## Question 6
The client sent a `role: "admin"` field in the body during registration. Why can't you
trust it and what's the right way?

## Question 7
What HTTP status and shape do you return on a DTO validation failure? What do you put
in the body?
