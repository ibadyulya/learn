# Caching and CDN — tasks

Answer in your own words, then check against [07-caching-cdn.answers.md](./07-caching-cdn.answers.md).

> 🌐 Russian version: [07-caching-cdn.tasks.md](../ru/07-caching-cdn.tasks.md)

---

## Question 1
Name the cache layers along the request path from browser to DB. Why is it beneficial
to cache closer to the user?

## Question 2
What is a CDN and what are the two main benefits it provides?

## Question 3
Pull CDN vs push CDN: what's the difference? Which is more common?

## Question 4
How do HTTP headers control the cache? Explain `Cache-Control: max-age` and `ETag` →
`304`.

## Question 5
How do you update static content if the CDN cached the old version with a long TTL? (Two
approaches.)

## Question 6
What can be cached aggressively and what can't? Give an example each.

## Question 7
How does a cache stampede manifest at the CDN/origin level and why is it dangerous?
