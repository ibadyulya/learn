# Load balancing, replication, sharding

Block B, a continuation of [05-scaling](./05-scaling.en.md). How to scale beyond
the stateless layer — traffic and, most importantly, **data**.

> 🌐 Russian version: [06-scaling-data.ru.md](./06-scaling-data.ru.md)

The through-line:

> **A (stateless) application is easy to scale — instances behind a balancer. The
> real pain is the database: it's stateful, you can't just multiply it.
> Replication and sharding are two different answers to "how to scale a DB". Key:
> replication = the same data in many copies; sharding = different data on
> different nodes.**

---

## Where the bottleneck is

The application layer scales **easily**: instances are stateless (see 05), add
copies behind a balancer — done, they're interchangeable. But we moved state
**into the DB** — and it's **stateful**: it holds data that can't be lost and
must be kept consistent. You can't multiply it like stateless instances — right
away the question arises "how to keep the copies in sync". **That's why scaling
the DB is a separate hard topic, and the whole system runs up against it.**

## Load balancing — spread the traffic

A deeper look than [05](./05-scaling.en.md). At what level it works (they love to
ask):
- **L4 (transport)** — by IP/ports, without looking into the request. Fast, dumb.
- **L7 (application)** — understands HTTP (URL, headers, cookies), routes smartly
  (`/api/*` → some, `/images/*` → others). Flexible, but more expensive.

Algorithms: **round-robin**, **least connections**, **weighted**, **IP hash**.
**Health-checks** — a dead instance is automatically pulled out of rotation
(that's what makes horizontal self-healing).

Keep in mind: **the balancer = a distributor in front of the stateless layer.**
It scales traffic intake, but not the DB behind it. For the DB — the tools below.

## Replication — "one dataset, many copies"

Keep **full copies of the same data** on several nodes. The **leader/follower**
scheme (master/replica, primary/secondary):

```
       writes
client ──────────────► LEADER
                         │ replicates changes
              ┌──────────┼──────────┐
              ▼          ▼          ▼
          FOLLOWER   FOLLOWER   FOLLOWER   ◄── reads
```
- **all writes → one leader**;
- the leader **broadcasts changes** to followers;
- **reads are spread across the replicas.**

Two benefits (don't mix them up):
1. **Read scaling** — reads are usually far more numerous than writes; spreading
   `SELECT`s across replicas → smeared the read load. (This does NOT scale
   writes.)
2. **Fault tolerance (HA)** — the leader went down → a replica is **promoted** to
   the new leader (failover), data isn't lost.

The problem — **replication lag.** A change reaches the replicas not instantly.
The same CAP/PACELC trade-off:
- **synchronous** — the leader waits for the replicas before "written":
  reliable/consistent, but **slow**;
- **asynchronous** — the leader responded immediately, the replicas will catch
  up: **fast**, but a read from a replica may return **stale** → **eventual
  consistency**.

A classic bug — **read-your-writes**: changed the profile (in the leader),
immediately re-read (from a lagging replica) → you see the old value. The fix:
send reads of just-written data to the leader.

> **Replication scales READS and provides HA, at the cost of possible replica lag
> (eventual consistency). It doesn't scale writes — all go to one leader.**

## Sharding — "different data on different nodes"

Replication hits a wall: all writes still go to one leader, and the data must fit
on one machine. **Sharding (partitioning) — cut the data into pieces and put
DIFFERENT pieces on DIFFERENT nodes:**

```
Replication:  node1=[A,B,C]  node2=[A,B,C]  node3=[A,B,C]   ← the same thing
Sharding:     node1=[A,B]    node2=[C,D]    node3=[E,F]     ← different parts
```
Each **shard** holds its own piece and is responsible for writes/reads only on
it. It gives:
1. **write scaling** — spread across shards, don't press into one leader;
2. **volume scaling** — the data no longer has to fit on one machine.

**Shard key — the most important decision.** The field we cut by (`user_id`):
- **range** — 1–1000 → shard1, etc. Simple, range queries are good, but skew is
  easy (the last shard heats up);
- **hash** — `hash(key) % N`. Even, but range queries hit all shards;
- **directory** — a lookup "key → shard". Flexible, but itself a bottleneck.

The pain with a bad key:
- **hotspot** — the load piles into one shard (sharded by country, 80% from one);
- **cross-shard queries / JOIN** — data is scattered → you must query all shards
  and stitch together; a JOIN between shards is nearly impossible. So related
  data goes on one shard (a user's orders — with the user);
- **rebalancing** — adding a shard = reshuffling data (softened by consistent
  hashing).

> **Sharding scales WRITES and VOLUME. Replication is about availability, sharding
> is about capacity.**

## In reality they're combined

Not either-or, but together — they solve different problems:
- **sharding** cuts the data → scales writes and volume;
- **replicating each shard** → scales reads on it + HA.

```
Shard 1: leader + 2 replicas
Shard 2: leader + 2 replicas
Shard 3: leader + 2 replicas
```
Each shard is a mini-cluster "leader + followers". That's how MongoDB sharded
clusters, Cassandra, Vitess (MySQL) are built.

## The whole picture

```
traffic → BALANCER → stateless instances   (easy: copies)
                          ↓
                      DB (stateful — that's where it's hard):
  REPLICATION: copies of the same data → scales READS + HA (at the cost of lag/eventual)
  SHARDING:    different data per node  → scales WRITES + volume (at the cost of shard key)
  TOGETHER:    sharding + replication of each shard → both
```

> Phrasing: "The application layer scales easily — stateless behind a balancer.
> The difficulty is in the DB (stateful). Replication — copies of the same data:
> writes to the leader, reads from replicas, scales reads and gives failover, but
> replicas lag (async → eventual consistency, the same CAP trade-off). Sharding —
> different data per node by shard key: scales writes and volume, but the key is
> critical (else a hotspot), cross-shard queries/JOINs are expensive. In reality
> they're combined: shard, and replicate each shard."

**A tricky closer:** does replication scale writes? **No** — all writes go to one
leader. Only **sharding** scales writes. Confusing these two is the most common
mistake on the topic.
