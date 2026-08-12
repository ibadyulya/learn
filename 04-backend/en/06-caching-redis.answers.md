# Caching (Redis) — solutions

> 🌐 Russian version: [06-caching-redis.answers.md](../ru/06-caching-redis.answers.md)

---

## Question 1
A cache is a fast layer in front of a slow/expensive source: the result is computed
once and served from fast memory while valid. The trade-off — **freshness vs speed**:
data may be slightly stale, but it's fast and easier on the source. It works thanks to
**locality** — the same data is requested often.

## Question 2
**Cache-aside:**
- read: in cache → serve; miss → fetch from DB, put in cache, serve;
- write: update the DB, then **invalidate** (delete) the key.
**Write-through** differs on write: we write **both to the cache and synchronously to
the DB** at once (the cache is always consistent with the DB, but writes are slower).
Cache-aside populates the cache lazily (on read), write-through on write.

## Question 3
Because it's hard to remove the stale **in time and precisely**: too early — lose the
benefit, too late — serve wrong data; easy to forget **dependent** keys. Approaches:
- **TTL** — simple, but has a staleness window;
- **explicit deletion on write** — more precise, but you must remember all related
  keys;
- **by event** (subscribe to changes) — fresh, but more complex.
The trade-off: a short TTL is simpler and safer for freshness, a long one more
efficient.

## Question 4
**Cache stampede** — a popular key expired, and many requests **simultaneously** miss
and hit the DB at once, overloading it (a special case of thundering herd). Fight it:
1. **lock / single-flight** — only the first recomputes, the rest wait for its result;
2. **stale-while-revalidate** — serve the stale value while refreshing in the
   background;
3. **TTL jitter** — spread out the expiries so keys don't expire in sync.

## Question 5
Besides caching, Redis is used for: **sessions** (a shared store for stateless
instances), **rate limiting** (counters with TTL), **distributed locks**,
**queues/pub-sub**, ratings/leaderboards (sorted set), throttling. Rich structures
(hash/list/set/zset) + TTL + atomicity of simple commands.

## Question 6
No, not the same way. A product list tolerates slight staleness (eventual consistency)
— showing yesterday's catalog is no disaster. An **account balance** is critical:
showing a stale value → wrong decisions (double spend, overdraft). There either read
**from the source**, or invalidate **hard and immediately** on any change, or don't
cache. Rule: the more critical the freshness, the more careful the cache.

## Question 7
**TTL** — a key's lifetime: on expiry Redis deletes it itself (freshness without manual
cleanup). **Eviction** — what to do when **memory is full**: Redis drops keys by a
policy. **LRU** (least recently used) evicts long-unused ones — assuming they won't be
asked for again; there's also LFU (by frequency), random, "only keys with TTL", etc.
