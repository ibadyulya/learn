# Caching and CDN — solutions

> 🌐 Russian version: [07-caching-cdn.answers.md](../ru/07-caching-cdn.answers.md)

---

## Question 1
Levels: **browser** → **CDN (edge)** → **reverse proxy/gateway** → **application
(Redis/in-memory)** → **DB buffer cache**. Caching closer to the user is beneficial
because the request **doesn't travel the whole path** to the source: shorter latency and
less work at each lower level (the browser cache doesn't hit the network at all).

## Question 2
A **CDN** is a network of edge servers worldwide serving content (usually static) from
the node nearest the user. Benefits: (1) **lower latency** — data is physically closer;
(2) **origin offload** — most traffic settles on the edge, a small part reaches your
server (+ absorbing spikes/some DDoS).

## Question 3
- **Pull** — the edge **fetches** content from the origin on the first miss and caches it;
  easy to set up, the cache fills on demand.
- **Push** — you **upload** content to the edge nodes in advance; more control, but more
  work.
**Pull** is more common (simpler).

## Question 4
- **`Cache-Control: max-age=N`** — how many seconds the response is considered **fresh**;
  while fresh, it's taken from the cache without a request.
- **`ETag`** — a version "fingerprint" of the resource. On revalidation the browser sends
  `If-None-Match: <etag>`; unchanged → the server replies **`304 Not Modified` without a
  body** (saved traffic, the cache stays valid).

## Question 5
1. **Filename versioning** — `app.a1b2.js` (a content hash in the name): content changed
   → a new name → the browser/CDN see a new URL, the old cache doesn't get in the way.
   The best trick for static (you can set a huge max-age).
2. **Purge/invalidation** — explicitly flush the object in the CDN via an API so the edge
   refetches the fresh one from the origin.

## Question 6
- **Aggressively cacheable:** static assets (JS/CSS/images with a version in the name),
  rarely-changing public content (catalog, articles).
- **Not/carefully:** personal data (balance, account page), critically-fresh data,
  responses depending on the user/authorization → `no-store`/`private` or a short TTL.

## Question 7
A popular object with an expired TTL: many requests **miss simultaneously** at the edge
and **hit the origin at once** for the fresh version, creating a load spike that can crash
the origin (see cache stampede/thundering herd). Fixed by: single-flight at the edge,
stale-while-revalidate, TTL jitter.
