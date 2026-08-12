# Caching and CDN

> 🌐 Russian version: [07-caching-cdn.md](../ru/07-caching-cdn.md)

Caching at the system level: where to put it along the request path and what a CDN
is. Reads as a story; for the backend cache mechanics see
[caching-redis](../../04-backend/en/06-caching-redis.md), here it's about **layers**
and **geography**.

The through-line of the topic:

> **A request passes through several levels, and at each you can put a cache — closer
> to the user means faster and less load on the source. A CDN is a geographically
> distributed cache of static content at the "edge" of the network, near the user.
> The price is the same — freshness and invalidation.**

---

## Cache layers along the request path

From client to source — each level can cache:
```
Browser → CDN (edge) → reverse proxy/API gateway → application → DB
  cache     cache            cache                     cache     (the source itself)
```
- **browser** — caches responses by HTTP headers (don't go to the network at all);
- **CDN** — static content (images, JS/CSS, video) on servers near the user;
- **reverse proxy** (nginx, Varnish) — caches responses in front of the app;
- **application** — Redis/in-memory for expensive computations/queries;
- **DB** — its own buffer cache.
Rule: **cache as close to the user as possible** — the shorter the path and the less
work.

---

## CDN — a cache at the edge

A **CDN (Content Delivery Network)** is a network of **edge servers** worldwide. The
user gets static content from the **nearest** node, not the far-off origin.
- ➕ **lower latency** (data is physically closer → less ping);
- ➕ **origin offload** — most traffic settles on the edge, a small fraction reaches your
  server;
- ➕ absorbing spikes/protecting from some DDoS.
The classic — static assets; modern CDNs also cache API responses and do edge compute.

**Pull vs push:** *pull* — the edge fetches from the origin on the first miss and caches;
*push* — you upload content to the edge in advance. Pull is simpler and more common.

---

## HTTP caching (briefly)

The browser and CDN decide whether to cache by headers:
- **`Cache-Control: max-age=3600`** — how many seconds to consider it fresh;
- **`ETag`/`If-None-Match`** — a version validator: unchanged → `304 Not Modified`
  without a body (saved traffic);
- **`no-store`** — don't cache (private).
For static — a long `max-age` + **versioning in the filename** (`app.a1b2.js`): content
changed → a new name → the cache doesn't get in the way.

---

## Invalidation and difficulties

The same two troubles as with any cache (see
[caching-redis](../../04-backend/en/06-caching-redis.md)):
- **staleness** — you posted a new version to the CDN, but the edge serves the old one
  until TTL expiry; fixed by **purge/invalidation** and versioned names;
- **cache stampede** — a popular object expired, a crowd of misses hits the origin.
And the general question: **what's cacheable?** Static and rarely-changing — yes;
personal/critically-fresh — carefully or `no-store`.

---

## Interview phrasing

> "I put a cache at every level of the path — browser, CDN, reverse proxy, application
> (Redis), DB — closer to the user means faster and less load on the source. A CDN is a
> network of edge servers serving static from the nearest node: lower latency and origin
> offload, usually a pull model. I control it with HTTP headers (`Cache-Control`, `ETag`
> → 304), for static a long max-age + versioned filenames. The main difficulties are the
> same — invalidation (purge/versions) and cache stampede; personal and critically-fresh
> data I don't cache aggressively."

---

See [07-caching-cdn.tasks.md](./07-caching-cdn.tasks.md) — tasks. Solutions in
[07-caching-cdn.answers.md](./07-caching-cdn.answers.md).
