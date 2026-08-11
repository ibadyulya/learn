# Меню подготовки к собесу (senior fullstack JS)

> 🌐 English version: [en.PLAN.md](./en.PLAN.md)

Навигация по темам. Каждая тема — папка с конспектом, задачами и разборами.
Кликай по названию темы, чтобы перейти к конспекту.

---

## 🔁 [Event Loop (Node.js)](./event-loop/ru.README.md)
Как Node решает, что выполнять следующим: фазы libuv, микро/макротаски,
`nextTick`, гонка `setTimeout(0)` vs `setImmediate`.
→ [Конспект](./event-loop/ru.README.md) · [Задачи](./event-loop/ru.tasks.md) · [Разборы](./event-loop/ru.answers.md)

## 📦 [CommonJS vs ESM](./cjs-esm/ru.README.md)
Две системы модулей: синхронная динамическая загрузка (CJS) против асинхронной
статической (ESM), живые связи, интероп, `import.meta`, top-level await.
→ [Конспект](./cjs-esm/ru.README.md) · [Задачи](./cjs-esm/ru.tasks.md) · [Разборы](./cjs-esm/ru.answers.md)

## 🧠 [Advanced TypeScript](./typescript-advanced/ru.README.md)
Типы как второй язык: generics, mapped/conditional types, `infer`, `never`,
key remapping, дистрибутивность, разбор утилитных типов.
→ [Конспект](./typescript-advanced/ru.README.md) · [Задачи](./typescript-advanced/ru.tasks.md) · [Разборы](./typescript-advanced/ru.answers.md)

## 🏗️ [System Design](./system-design/ru.README.md)
Надёжный event-driven backend и масштабирование. Один топик = один файл.
- [01 — Foundations](./system-design/ru.01-foundations.md) — CAP/PACELC, трейд-офф-мышление
- [02 — Event-driven messaging](./system-design/ru.02-event-driven-messaging.md) — продюсер/брокер/консьюмер, семантики доставки
- [03 — Reliability patterns](./system-design/ru.03-reliability-patterns.md) — ack/nack, идемпотентность, outbox, retries, DLQ
- [04 — Kafka vs RabbitMQ](./system-design/ru.04-kafka-vs-rabbitmq.md) — лог vs очередь
- [05 — Scaling](./system-design/ru.05-scaling.md) — вертикаль/горизонталь, cluster/PM2, worker_threads
- [06 — Scaling data](./system-design/ru.06-scaling-data.md) — балансировка, репликация, шардинг

## 🗄️ [Databases](./databases/ru.README.md)
Про одну БД: транзакции, изоляция, блокировки. Один топик = один файл.
- [01 — Уровни изоляции](./databases/ru.01-isolation-levels.md) · [Задачи](./databases/ru.01-isolation-levels.tasks.md) · [Разборы](./databases/ru.01-isolation-levels.answers.md)
