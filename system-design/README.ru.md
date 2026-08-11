# System Design — навигация

> 🌐 English version: [README.en.md](./README.en.md)

Область system design. Один топик = один файл, читать по порядку номеров. Общий
конспект-план всей подготовки — в [../PLAN.ru.md](../PLAN.ru.md).

Сквозная идея области: **сеть ненадёжна, узлы падают. Почти каждый паттерн — это
ответ на «нельзя гарантировать доставку/обработку ровно один раз».** Мета-правило
собеса: рассуждай про **трейд-оффы**, начинай ответ с «зависит от…».

---

## Блок A — Messaging & reliability ✅

| # | Файл | О чём | Статус |
|---|---|---|---|
| 01 | [foundations](./01-foundations.ru.md) | распределённые системы, CAP/PACELC, трейд-офф-мышление | ✅ |
| 02 | [event-driven-messaging](./02-event-driven-messaging.ru.md) | зачем event-driven, продюсер/брокер/консьюмер, семантики доставки | ✅ |
| 03 | [reliability-patterns](./03-reliability-patterns.ru.md) | ack/nack, идемпотентность, outbox, retries+backoff, DLQ, rate limiting, crash recovery | ✅ |
| 04 | [kafka-vs-rabbitmq](./04-kafka-vs-rabbitmq.ru.md) | лог vs очередь, offset/партиции vs exchange/competing consumers | ✅ |

Стержень блока: `at-least-once (доставка) + идемпотентность (обработка) =
effectively-once`. Честного exactly-once на уровне доставки не существует.

## Блок B — Scalability & инфраструктура

| # | Файл | О чём | Статус |
|---|---|---|---|
| 05 | [scaling](./05-scaling.ru.md) | вертикаль/горизонталь Node, cluster/PM2, stateless, worker_threads | ✅ |
| 06 | [scaling-data](./06-scaling-data.ru.md) | балансировка (L4/L7), репликация (leader/follower, lag), шардинг (shard key) | ✅ |

Ещё в планах (файлы появятся по мере прохождения):
- кэширование и CDN (стратегии, инвалидация);
- идемпотентные HTTP API, saga/2PC (блок C).

## Блок C — прочее ⏳

- идемпотентные HTTP API (Idempotency-Key на REST);
- консистентность между сервисами (saga, 2PC vs eventual).

> Смежное: транзакции и **уровни изоляции** — в отдельном модуле `databases/`
> (это про одну БД, не про распределённость). Репликация/шардинг живут на стыке
> — разберём один раз и сошлёмся.
