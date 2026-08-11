# Interview prep menu (senior fullstack JS)

Navigation by topic. Each topic is a folder with a conspect, tasks and answers.
Click a topic title to jump to its conspect.

> 🌐 Russian version: [ru.PLAN.md](./ru.PLAN.md)

---

## 🔁 [Event Loop (Node.js)](./event-loop/en.README.md)
How Node decides what to run next: libuv phases, micro/macrotasks,
`nextTick`, the `setTimeout(0)` vs `setImmediate` race.
→ [Conspect](./event-loop/en.README.md) · [Tasks](./event-loop/en.tasks.md) · [Answers](./event-loop/en.answers.md)

## 📦 [CommonJS vs ESM](./cjs-esm/en.README.md)
Two module systems: synchronous dynamic loading (CJS) versus asynchronous
static loading (ESM), live bindings, interop, `import.meta`, top-level await.
→ [Conspect](./cjs-esm/en.README.md) · [Tasks](./cjs-esm/en.tasks.md) · [Answers](./cjs-esm/en.answers.md)

## 🧠 [Advanced TypeScript](./typescript-advanced/en.README.md)
Types as a second language: generics, mapped/conditional types, `infer`, `never`,
key remapping, distributivity, a walkthrough of the utility types.
→ [Conspect](./typescript-advanced/en.README.md) · [Tasks](./typescript-advanced/en.tasks.md) · [Answers](./typescript-advanced/en.answers.md)

## 🏗️ [System Design](./system-design/en.README.md)
A reliable event-driven backend and scaling. One topic = one file.
- [01 — Foundations](./system-design/en.01-foundations.md) — CAP/PACELC, trade-off thinking
- [02 — Event-driven messaging](./system-design/en.02-event-driven-messaging.md) — producer/broker/consumer, delivery semantics
- [03 — Reliability patterns](./system-design/en.03-reliability-patterns.md) — ack/nack, idempotency, outbox, retries, DLQ
- [04 — Kafka vs RabbitMQ](./system-design/en.04-kafka-vs-rabbitmq.md) — log vs queue
- [05 — Scaling](./system-design/en.05-scaling.md) — vertical/horizontal, cluster/PM2, worker_threads
- [06 — Scaling data](./system-design/en.06-scaling-data.md) — load balancing, replication, sharding

## 🗄️ [Databases](./databases/en.README.md)
About a single DB: transactions, isolation, locks. One topic = one file.
- [01 — Isolation levels](./databases/en.01-isolation-levels.md) · [Tasks](./databases/en.01-isolation-levels.tasks.md) · [Answers](./databases/en.01-isolation-levels.answers.md)
