# Interview prep menu (senior fullstack JS)

Navigation by topic. Each topic is a folder with a conspect, tasks and answers.
Click a topic title to jump to its conspect.

> 🌐 Russian version: [PLAN.ru.md](./PLAN.ru.md)

---

## 🔁 [Event Loop (Node.js)](./event-loop/README.en.md)
How Node decides what to run next: libuv phases, micro/macrotasks,
`nextTick`, the `setTimeout(0)` vs `setImmediate` race.
→ [Conspect](./event-loop/README.en.md) · [Tasks](./event-loop/tasks.en.md) · [Answers](./event-loop/answers.en.md)

## 📦 [CommonJS vs ESM](./cjs-esm/README.en.md)
Two module systems: synchronous dynamic loading (CJS) versus asynchronous
static loading (ESM), live bindings, interop, `import.meta`, top-level await.
→ [Conspect](./cjs-esm/README.en.md) · [Tasks](./cjs-esm/tasks.en.md) · [Answers](./cjs-esm/answers.en.md)

## 🧠 [Advanced TypeScript](./typescript-advanced/README.en.md)
Types as a second language: generics, mapped/conditional types, `infer`, `never`,
key remapping, distributivity, a walkthrough of the utility types.
→ [Conspect](./typescript-advanced/README.en.md) · [Tasks](./typescript-advanced/tasks.en.md) · [Answers](./typescript-advanced/answers.en.md)

## 🏗️ [System Design](./system-design/README.en.md)
A reliable event-driven backend and scaling. One topic = one file.
- [01 — Foundations](./system-design/01-foundations.en.md) — CAP/PACELC, trade-off thinking
- [02 — Event-driven messaging](./system-design/02-event-driven-messaging.en.md) — producer/broker/consumer, delivery semantics
- [03 — Reliability patterns](./system-design/03-reliability-patterns.en.md) — ack/nack, idempotency, outbox, retries, DLQ
- [04 — Kafka vs RabbitMQ](./system-design/04-kafka-vs-rabbitmq.en.md) — log vs queue
- [05 — Scaling](./system-design/05-scaling.en.md) — vertical/horizontal, cluster/PM2, worker_threads
- [06 — Scaling data](./system-design/06-scaling-data.en.md) — load balancing, replication, sharding

## 🗄️ [Databases](./databases/README.en.md)
About a single DB: transactions, isolation, locks. One topic = one file.
- [01 — Isolation levels](./databases/01-isolation-levels.en.md) · [Tasks](./databases/01-isolation-levels.tasks.en.md) · [Answers](./databases/01-isolation-levels.answers.en.md)
