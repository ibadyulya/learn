# NestJS — tasks

Answer in your own words, then check against [02-nestjs.answers.md](./02-nestjs.answers.md).

> 🌐 Russian version: [02-nestjs.tasks.md](../ru/02-nestjs.tasks.md)

---

## Question 1
What does Nest add over bare Express and why? Name 3–4 things.

## Question 2
What is a provider and how does dependency injection work in Nest? How does it relate
to the DIP principle from SOLID?

## Question 3
What does `@Module` consist of (imports/controllers/providers/exports)? What does it
mean that a provider isn't exported?

## Question 4
What scope do providers have by default? When do you need `REQUEST` scope and why is
it more expensive?

## Question 5
Describe the request-handling pipeline in order. What is each layer responsible for?

## Question 6
Where do you put the logic: (a) check the user has the admin role; (b) validate the
request body; (c) log the execution time; (d) turn a thrown exception into a 400 JSON
response?

## Question 7
Why should a controller be "thin"? What should be in it and what shouldn't?
