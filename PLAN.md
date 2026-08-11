# Меню подготовки к собесу (senior fullstack JS)

Навигация по темам. Каждая тема — папка с конспектом, задачами и разборами.
Кликай по названию темы, чтобы перейти к конспекту.

---

## 🔁 [Event Loop (Node.js)](./event-loop/README.md)
Как Node решает, что выполнять следующим: фазы libuv, микро/макротаски,
`nextTick`, гонка `setTimeout(0)` vs `setImmediate`.
→ [Конспект](./event-loop/README.md) · [Задачи](./event-loop/tasks.md) · [Разборы](./event-loop/answers.md)

## 📦 [CommonJS vs ESM](./cjs-esm/README.md)
Две системы модулей: синхронная динамическая загрузка (CJS) против асинхронной
статической (ESM), живые связи, интероп, `import.meta`, top-level await.
→ [Конспект](./cjs-esm/README.md) · [Задачи](./cjs-esm/tasks.md) · [Разборы](./cjs-esm/answers.md)

## 🧠 [Advanced TypeScript](./typescript-advanced/README.md)
Типы как второй язык: generics, mapped/conditional types, `infer`, `never`,
key remapping, дистрибутивность, разбор утилитных типов.
→ [Конспект](./typescript-advanced/README.md) · [Задачи](./typescript-advanced/tasks.md) · [Разборы](./typescript-advanced/answers.md)

## 🏗️ [System Design](./system-design/README.md)
Надёжный event-driven backend и масштабирование. Один топик = один файл.
- [01 — Foundations](./system-design/01-foundations.md) — CAP/PACELC, трейд-офф-мышление
- [02 — Event-driven messaging](./system-design/02-event-driven-messaging.md) — продюсер/брокер/консьюмер, семантики доставки
- [03 — Reliability patterns](./system-design/03-reliability-patterns.md) — ack/nack, идемпотентность, outbox, retries, DLQ
- [04 — Kafka vs RabbitMQ](./system-design/04-kafka-vs-rabbitmq.md) — лог vs очередь
- [05 — Scaling](./system-design/05-scaling.md) — вертикаль/горизонталь, cluster/PM2, worker_threads
- [06 — Scaling data](./system-design/06-scaling-data.md) — балансировка, репликация, шардинг

## 🗄️ [Databases](./databases/README.md)
Про одну БД: транзакции, изоляция, блокировки. Один топик = один файл.
- [01 — Уровни изоляции](./databases/01-isolation-levels.md) · [Задачи](./databases/01-isolation-levels.tasks.md) · [Разборы](./databases/01-isolation-levels.answers.md)
