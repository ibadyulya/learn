# Scaling Node applications

Opens block B (scalability). Node specifics: everything follows from Node being
single-threaded (see the event-loop module). Builds on the stateless idea, which
later ties into load balancing and replication.

> 🌐 Russian version: [ru.05-scaling.md](./ru.05-scaling.md)

The through-line:

> **Node is single-threaded by nature: one process = one event loop = one CPU
> core busy with your JS. No matter how many cores the machine has, one Node
> process uses only one. Everything about scaling Node follows from this — and
> why "give it a beefier machine" works worse here than you'd expect.**

---

## The limit: Node sees only one core

Node has **one thread** running your JavaScript. That's both a strength (no thread
races) and a limit: **your JS executes on one core.** A 32-core machine + one
`node app.js` process = **one** core loaded, the rest idle.

> Caveat: libuv keeps a background thread pool for some I/O, but **your JS**
> (callbacks, request handling, computation) is one thread. For scaling it's
> precisely that one that matters.

Hence two ways to grow.

## Vertical scaling (scale up) — "a beefier machine"

Make **one machine more powerful**: more CPU/RAM, a faster disk. You don't add
servers — you fatten one up.

Plus: **as simple as it gets** — moved to a fatter instance, same architecture,
no balancing, no synchronization.

**The catch for Node:** adding cores to one process is **useless** — it used one
and keeps using one. From vertical scaling a Node process is helped only by a
**faster core** and **more RAM**, but not the number of cores.

General downsides of vertical:
- **there's a ceiling** — the most powerful machine doesn't exist;
- **a single point of failure** — the machine went down → everything is down;
- the price of top-tier hardware grows non-linearly.

## cluster — occupy all the cores of one machine

Since process = core, the solution: **run several Node processes, one per core.**
That's the **`cluster`** module: a master process spawns **workers** (≈ as many
as cores), all listen on **one port**, connections are spread among them. Each
worker — **its own process, its own event loop, its own core**:

```
                   ┌── worker 1 (core 1)
clients → :3000 → master ── worker 2 (core 2)
                   └── worker 3 (core 3)
```

In practice, instead of manual `cluster` people take **PM2** (`pm2 start app.js
-i max` — a worker per core + auto-restart of crashed ones) or hand it to an
orchestrator.

**Important: workers do NOT share memory** — separate processes, each with its
own heap. State in one worker's memory is invisible to others → the bridge to
stateless.

## Horizontal scaling (scale out) — "more machines"

Add **servers** and spread the load. `cluster` is the same trick within one
machine; horizontal is across several machines. In front of the copies — a **load
balancer**:

```
                    ┌── instance A (machine 1)
clients → load balancer ── instance B (machine 2)
                    └── instance C (machine 3)
```

Algorithms: **round-robin**, **least connections**, by IP hash. Plus
**health-checks** — an instance isn't responding → traffic is steered away from
it.

Pluses (which vertical didn't have):
- **almost no ceiling** — need more → added machines (cloud autoscaling);
- **fault tolerance** — an instance went down → the balancer steered traffic
  away, the service is alive;
- cheaper — many ordinary machines beat one top-tier one.

The minus — **complexity** (a balancer, fleet management → Docker/Kubernetes,
harder deploy/debug) and one hard requirement below.

## Why horizontal requires stateless

The balancer sends requests to **any** instance. If an instance **remembers
something in its memory** — it breaks:
```
request 1 → instance A → the session landed in A's memory
request 2 → instance C → C's memory has NO session → "you're not authorized"
```
Rule: **instances are stateless** — any of them handles any request the same way.
State goes **outside, into shared storage**:
- sessions/cache → **Redis**;
- data → **DB**;
- files → **S3**, not the local disk.

> **A stateless instance + state in shared storage = any request to any instance
> works the same.** (The same problem is already there with `cluster` — workers
> don't share memory either.)

**Sticky sessions** — a crutch: the balancer always sends a user to one instance,
letting it keep state in memory. Breaks fault tolerance (its instance went down →
state lost) and evenness. The right way — **stateless + Redis**, sticky
temporarily.

## worker_threads — don't confuse with cluster (CPU-bound)

`cluster` and `worker_threads` solve **different** problems.

Node is strong at **I/O-bound** load — operations that mostly **wait** (DB,
network, files). One event loop juggles thousands of these — waiting costs almost
no CPU.

The trouble is with **CPU-bound** work — heavy computation (video, hashes,
images, a giant JSON). It **doesn't wait, it loads the processor**, and the
single-threaded event loop **blocks**: while you compute, other requests stand
still. One calculation freezes the whole server.

**`worker_threads`** — real threads **inside the process**, to which you hand the
heavy computation so as **not to block the main event loop**.

| | `cluster` | `worker_threads` |
|---|---|---|
| What | several **processes** | several **threads** in one process |
| Memory | don't share | can share (SharedArrayBuffer) |
| For | scaling **I/O throughput** across cores | not blocking the loop on **CPU-bound** |
| Analogy | many copies of the server | move the heavy calc off to the side |

In short: **`cluster` — serve more requests (occupy cores for I/O),
`worker_threads` — so a heavy computation doesn't freeze the server.**

## The whole picture

```
one machine, one process    →  1 event loop = 1 core (Node's ceiling)
        ↓ occupy all cores of the machine
cluster / PM2               →  several processes on one machine
        ↓ go beyond one machine
horizontal scale-out        →  many machines behind a balancer
                               ⚠ requires STATELESS (state → Redis/DB/S3)

special CPU-bound case      →  worker_threads (don't block the loop)
```

> Phrasing: "Node is single-threaded, one process is capped at one core. Vertical
> scaling for Node is limited — extra cores go unused, `cluster`/PM2 helps (a
> process per core). Next — horizontal: many instances behind a balancer with
> health-checks, almost no ceiling and fault-tolerant, but requires stateless —
> state in Redis/DB/S3, otherwise a request to another instance won't find the
> data. For CPU-bound there's worker_threads, so computation doesn't block the
> event loop. Usually combined: cluster within a machine + scale-out of machines,
> often via Kubernetes."
