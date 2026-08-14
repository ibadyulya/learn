# Rendering, SSR and Next.js — tasks

Answer in your own words, then check against [08-rendering-and-nextjs.answers.md](./08-rendering-and-nextjs.answers.md).

> 🌐 Russian version: [08-rendering-and-nextjs.tasks.md](../ru/08-rendering-and-nextjs.tasks.md)

---

## Question 1
What problems does pure CSR (a plain SPA) have? Why is it bad for SEO and the first paint?

## Question 2
Compare CSR, SSR, SSG and ISR: where the HTML renders, when data is taken, what about the
first-paint speed.

## Question 3
What is hydration? Describe step by step what happens after the server has sent ready HTML.

## Question 4
What is a hydration mismatch and what causes it? How do you avoid it?

## Question 5
Why does SSR "speed up the paint but not remove the bundle"? How do RSC and partial
hydration address this?

## Question 6
What are React Server Components? How does a server component differ from a client one
(`'use client'`) and what's the benefit?

## Question 7
What does streaming SSR + Suspense give compared to plain SSR?

## Question 8
For three scenarios choose a rendering strategy and explain: (a) a docs blog; (b) a
personalized recommendations feed with SEO; (c) a dashboard editor behind a login.
