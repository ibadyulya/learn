# Databases — навигация

> 🌐 English version: [README.en.md](./README.en.md)

Область про базы данных. Один топик = один файл (`NN-topic.md`), рядом
`.tasks.md` и `.answers.md` для самопроверки. Общий план — [../PLAN.ru.md](../PLAN.ru.md).

Тема про **одну БД** (транзакции, изоляция, блокировки), в отличие от
распределённых очередей в [../system-design/](../system-design/). Репликация и
шардинг сидят на стыке — разберём один раз и сошлёмся.

---

| # | Топик | Задачи | Статус |
|---|---|---|---|
| 01 | [Уровни изоляции](./01-isolation-levels.ru.md) | [tasks](./01-isolation-levels.tasks.ru.md) · [answers](./01-isolation-levels.answers.ru.md) | ✅ |

## В планах (по мере надобности)
- **Блокировки** — pessimistic vs optimistic, `SELECT ... FOR UPDATE`, deadlocks.
- **MVCC** — как Postgres/InnoDB дают снимки без блокировки чтений.
- **Индексы** — B-tree, покрывающие индексы, когда индекс не используется.
- **Репликация / шардинг** — на стыке с system-design (масштабирование данных).

Сквозная нить всей области: транзакции из
[system-design/03-reliability-patterns](../system-design/03-reliability-patterns.ru.md)
(атомарный commit, откат) — это фундамент, на который здесь ложится изоляция.
