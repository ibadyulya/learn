# REST API design — tasks

Answer in your own words, then check against [03-rest-api-design.answers.md](./03-rest-api-design.answers.md).

> 🌐 Russian version: [03-rest-api-design.tasks.md](../ru/03-rest-api-design.tasks.md)

---

## Question 1
Why is `/getUser?id=5` bad REST design? What's the right way?

## Question 2
What's the difference between `401` and `403`? Give a scenario for each.

## Question 3
What do "safe" and "idempotent" mean for an HTTP method? Fill the table for GET, POST,
PUT, PATCH, DELETE.

## Question 4
Why isn't `POST` idempotent and why is that dangerous for payments? How do you solve
the double-charge problem at the API level?

## Question 5
What does "REST is stateless" mean and how does it relate to horizontal scaling?

## Question 6
What status code do you return: (a) a resource successfully created; (b) the body
failed validation; (c) deleted, no body; (d) a record with this email already exists?

## Question 7
Offset pagination vs cursor-based: what's the difference and when is cursor better?
