# Authentication and authorization — tasks

Answer in your own words, then check against [04-auth.answers.md](./04-auth.answers.md).

> 🌐 Russian version: [04-auth.tasks.md](../ru/04-auth.tasks.md)

---

## Question 1
What's the difference between authentication and authorization? Which status codes
correspond to each one's failure?

## Question 2
Server sessions vs JWT: name one key pro and con of each approach.

## Question 3
What three parts make up a JWT? Is the payload encrypted or signed? What follows from
that in practice?

## Question 4
Why do you need an access + refresh pair if you could issue one long-lived token?
What exactly does refresh solve?

## Question 5
Where do you store the token on the client: `localStorage` or an httpOnly cookie?
Which attack threatens each option?

## Question 6
What is OAuth2 and how does OIDC differ from it? What happens on "log in with Google"?

## Question 7
How do you store passwords correctly? Why not plaintext, nor just a fast hash like
SHA-256? What's the salt for?
