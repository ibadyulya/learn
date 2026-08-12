# Caching (Redis)

> 🌐 Russian version: [06-caching-redis.md](../ru/06-caching-redis.md)

How to speed up a system with a cache and why it's harder than it looks. Reads as a
story.

The through-line of the topic:

> **A cache is a fast layer in front of a slow source: it trades data freshness for
> speed. It's easy to put things in the cache — hard to remove them in time
> (invalidation). Redis is an in-memory store used for caching, sessions, counters,
> queues, and distributed locks.**

---

## Why cache

Something slow/expensive (a DB query, an external API call, a heavy computation) is
computed **once**, the result is put in a fast layer (memory/Redis), and served from
there while it's still valid. The benefit: lower latency and less load on the source.
It works thanks to **locality** — the same data is requested often.

---

## Caching strategies

- **Cache-aside (lazy)** — the most common: the app manages the cache itself.
  ```
  read:  in cache? yes → serve; no → fetch from DB, put in cache, serve
  write: update the DB + invalidate (delete) the key
  ```
- **Read-through** — always read through the cache layer, which fetches from the DB on
  a miss.
- **Write-through** — write to the cache and synchronously to the DB (consistent but
  slower writes).
- **Write-behind (write-back)** — write to the cache, to the DB asynchronously later
  (fast, but risk of loss on a crash).

On the backend the default is **cache-aside**.

---

## TTL, eviction, and the two hard problems

- **TTL** — a key's lifetime: stale — it disappears on its own. Balances freshness and
  load.
- **Eviction** — when memory runs out, Redis drops keys by a policy (**LRU** —
  least-recently-used, LFU, etc.).

> "There are two hard things in computer science: cache invalidation and naming
> things." **Invalidation** is the main pain: when and how to remove/refresh the
> stale.

Invalidation approaches: by **TTL** (simple, but a staleness window), **explicitly**
on write (delete the key — more precise, but easy to forget dependent keys), by
**event** (subscribe to changes). The trade-off: a short TTL is simpler, a long one
more efficient but riskier for freshness.

---

## Cache stampede (thundering herd)

The danger: a popular key's TTL expired — and **thousands** of requests miss at once
and **simultaneously** hit the DB, finishing it off (see
[reliability-patterns](../../06-system-design/en/03-reliability-patterns.md)).
Solutions:
- **lock/single-flight** — the first to miss recomputes, the rest wait for its result;
- **stale-while-revalidate** — serve the stale value while refreshing in the
  background;
- **TTL jitter** — so keys don't expire in sync.

---

## Redis — not just a cache

An in-memory store with rich structures:
- **data types:** string, hash, list, set, sorted set (zset), + a TTL per key;
- **sessions** (a fast shared store for stateless instances, see
  [scaling](../../06-system-design/en/05-scaling.md));
- **rate limiting** (counters with TTL, token bucket);
- **distributed locks** (one worker per task; carefully — there are correctness
  subtleties);
- **queues / pub-sub** (simple jobs, event streaming).

Redis is single-threaded per command → atomicity of simple operations out of the box.

---

## Careful with consistency

A cache is a **second copy** of the data; it can diverge from the source. That's a
deliberate trade-off (**eventual consistency**): fine for avatars and catalogs, not for
an account balance (there read from the source or invalidate hard). Always ask: "how
critical is freshness?"

---

## Interview phrasing

> "A cache is a fast layer in front of a slow source, trading freshness for speed,
> working on locality. Usually cache-aside: a miss → fetch from the DB and store, a
> write → invalidate the key. I manage TTL and eviction (LRU). The main difficulty is
> invalidation: TTL is simple but has a staleness window; explicit deletion is more
> precise but easy to forget dependent keys. I watch for cache stampede — when a
> popular key expires and a crowd hits the DB: I fix it with a lock/single-flight,
> stale-while-revalidate, TTL jitter. I use Redis not only for caching but for
> sessions, rate limiting, locks, queues. I remember a cache is a second copy: where
> freshness is critical, I cache carefully."

---

See [06-caching-redis.tasks.md](./06-caching-redis.tasks.md) — tasks. Solutions in
[06-caching-redis.answers.md](./06-caching-redis.answers.md).
