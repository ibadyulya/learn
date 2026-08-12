# Caching (Redis) — tasks

Answer in your own words, then check against [06-caching-redis.answers.md](./06-caching-redis.answers.md).

> 🌐 Russian version: [06-caching-redis.tasks.md](../ru/06-caching-redis.tasks.md)

---

## Question 1
What is a cache and what's the main trade-off it offers? Why does it work at all?

## Question 2
Describe the cache-aside strategy on read and on write. How does write-through differ
from it?

## Question 3
Why is cache invalidation called one of the "two hard problems"? Name invalidation
approaches and their trade-offs.

## Question 4
What is a cache stampede (thundering herd)? How does it arise and three ways to fight
it.

## Question 5
What besides caching is Redis used for? Name at least four scenarios.

## Question 6
Account balance data — should you cache it the same way as a product list? Why is
freshness critical here?

## Question 7
What are TTL and eviction? What does Redis do when memory runs out (the LRU policy)?
