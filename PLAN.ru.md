# Меню подготовки к собесу (senior fullstack JS)

> 🌐 English version: [PLAN.en.md](./PLAN.en.md)

Навигация по темам. Каждая тема — папка с конспектом, задачами и разборами.
Кликай по названию темы, чтобы перейти к конспекту.

---

## 🔁 [Event Loop (Node.js)](./event-loop/README.ru.md)
Как Node решает, что выполнять следующим: фазы libuv, микро/макротаски,
`nextTick`, гонка `setTimeout(0)` vs `setImmediate`.
→ [Конспект](./event-loop/README.ru.md) · [Задачи](./event-loop/tasks.ru.md) · [Разборы](./event-loop/answers.ru.md)

## 📦 [CommonJS vs ESM](./cjs-esm/README.ru.md)
Две системы модулей: синхронная динамическая загрузка (CJS) против асинхронной
статической (ESM), живые связи, интероп, `import.meta`, top-level await.
→ [Конспект](./cjs-esm/README.ru.md) · [Задачи](./cjs-esm/tasks.ru.md) · [Разборы](./cjs-esm/answers.ru.md)

## 🧠 [Advanced TypeScript](./typescript-advanced/README.ru.md)
Типы как второй язык: generics, mapped/conditional types, `infer`, `never`,
key remapping, дистрибутивность, разбор утилитных типов.
→ [Конспект](./typescript-advanced/README.ru.md) · [Задачи](./typescript-advanced/tasks.ru.md) · [Разборы](./typescript-advanced/answers.ru.md)

## 🏗️ [System Design](./system-design/README.ru.md)
Надёжный event-driven backend и масштабирование. Один топик = один файл.
- [01 — Foundations](./system-design/01-foundations.ru.md) — CAP/PACELC, трейд-офф-мышление
- [02 — Event-driven messaging](./system-design/02-event-driven-messaging.ru.md) — продюсер/брокер/консьюмер, семантики доставки
- [03 — Reliability patterns](./system-design/03-reliability-patterns.ru.md) — ack/nack, идемпотентность, outbox, retries, DLQ
- [04 — Kafka vs RabbitMQ](./system-design/04-kafka-vs-rabbitmq.ru.md) — лог vs очередь
- [05 — Scaling](./system-design/05-scaling.ru.md) — вертикаль/горизонталь, cluster/PM2, worker_threads
- [06 — Scaling data](./system-design/06-scaling-data.ru.md) — балансировка, репликация, шардинг

## 🗄️ [Databases](./databases/README.ru.md)
Про одну БД: транзакции, изоляция, блокировки. Один топик = один файл.
- [01 — Уровни изоляции](./databases/01-isolation-levels.ru.md) · [Задачи](./databases/01-isolation-levels.tasks.ru.md) · [Разборы](./databases/01-isolation-levels.answers.ru.md)
