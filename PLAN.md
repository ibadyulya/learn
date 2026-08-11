# План подготовки к собесу (senior fullstack JS)

Контекст: заказчик уже собеседовал друга (Andrew Zenkow), он прошёл и передал
фидбэк. Мы готовимся к собесу с тем же заказчиком по этому фидбэку. Формат
материалов: на каждую тему — подпапка с конспектом (`README.md`), задачами
(`tasks.md`) и разборами (`answers.md`). Объяснения — **развёрнуто, с механикой
и «зачем», а не тезисно.**

## Исходный фидбэк (что просят подтянуть)

- **Node.js Event Loop** — глубокие детали: `process.nextTick` vs Promise
  microtasks vs timers/check, `setTimeout(0)` vs `setImmediate`, разница
  CommonJS/ESM.
- **Advanced TypeScript** — mapped types, conditional types, key remapping,
  `infer`, `never`, utility types, свои type-level хелперы.
- **System design / production reliability** — transactional outbox,
  idempotency, retries with backoff, DLQ, rate limiting, ack/nack semantics,
  crash recovery, ограничения exactly-once.
- Сильные стороны (не трогаем): практический fullstack/backend на
  Node.js/NestJS, event-driven, очереди, CI/CD, инфра, коммуникация.

Источники из фидбэка: TypeScript Handbook (Advanced/Mapped Types),
Matt Pocock / Total TypeScript, Node.js docs (Event Loop), статьи по
Transactional Outbox, «Designing Data-Intensive Applications».

---

## Модуль 1 — Node.js Event Loop ✅ ПРОЙДЕНО

Папка: [event-loop/](./event-loop/). Конспект + 8 задач-трассировок + разборы.

Ключевое, что подтвердил фидбэк друга:
- «Момент краски» на собесе: `setTimeout(fn, 0)` клампится до **1 мс** → на
  верхнем уровне гонка `setTimeout` vs `setImmediate` **недетерминирована**,
  зависит от скорости старта («от железа»). Оба собеседника могут быть правы.
- Внутри I/O-колбэка `setImmediate` **гарантированно** раньше (`check` идёт
  сразу за `poll`).

Статус: закрыто. При повторе — только прогнать задачи вслух на скорость.

### Модуль 1.5 — CommonJS vs ESM ✅ ПРОЙДЕНО

Папка: [cjs-esm/](./cjs-esm/). Хвост из фидбэка про «CommonJS/ESM сценарии».
Что такое модули, синхронная (CJS) vs асинхронная/статическая (ESM) загрузка,
живые связи, интероп, `import.meta`, top-level await и как всё это сдвигает
момент старта таймеров в event loop. Конспект + 6 вопросов + разборы.

---

## Модуль 2 — Advanced TypeScript (следующий)

Цель: уметь с нуля написать type-level хелпер под диктовку и объяснить каждый
шаг. Не «знаю про infer», а «вывожу тип вслух, как код».

Подтемы (в порядке изучения):
1. **Generics + constraints** — база: `<T>`, `T extends U`, дефолты, вывод
   типов из аргументов.
2. **Conditional types** — `T extends U ? X : Y`; дистрибутивность по
   юнионам (почему `T extends U` «размазывается» по членам юниона и как это
   отключить через `[T]`).
3. **`infer`** — вытаскивание типа из позиции (возвращаемое значение, элемент
   массива, аргументы функции, `Awaited`).
4. **`never`** — почему это «пустой» тип, как он исчезает из юнионов, зачем в
   фильтрации ключей, exhaustiveness-проверки.
5. **Mapped types** — `{ [K in keyof T]: ... }`, модификаторы `?`/`readonly`
   и снятие через `-?`/`-readonly`.
6. **Key remapping** — `{ [K in keyof T as NewKey]: ... }`; фильтрация ключей
   через `as ... extends ... ? K : never`.
7. **Utility types изнутри** — переписать своими руками `Pick`, `Omit`,
   `Partial`, `Record`, `Exclude`, `ReturnType`, `Parameters` (понять, а не
   запомнить).
8. **Сборка своих хелперов** — комбинируем всё выше.

**Якорная задача с собеса** (её задавали живьём):
```ts
type SomeType = {
  f1: number;
  f2: boolean;
};
// Достать только поля, чьё значение имеет заданный тип U.
type ExtractByType<T, U> = /* ... */;
// ExtractByType<SomeType, boolean>  →  { f2: boolean }
```
Решается через mapped type + key remapping + conditional + `never`. Разобрать
до автоматизма и уметь объяснить, почему `as ... : never` выкидывает ключ.
(На собесе была и вторая задача по типам — друг не вспомнил; готовимся к классу
задач, а не к конкретной.)

Артефакты: [typescript-advanced/](./typescript-advanced/) — README + tasks (10) +
answers. Статус: ✅ **пройдено полностью** — generics, keyof/`T[K]`, mapped
types, conditional, `never`, key remapping, якорная задача `ExtractByType`,
`infer`, дистрибутивность, разбор утилитных типов (Pick/Omit/Partial/Exclude/
ReturnType и т.д. как обычные типы поверх механизмов).

---

## Модуль 3 — System Design: reliability & очереди

Цель: объяснить, как строить надёжный event-driven backend, и грамотно
отвечать на «почему сообщение перевыполнилось / потерялось / как ретраится».
Друга гоняли именно по этому (Kafka/RabbitMQ, ретраи, зафейленные сообщения).

Подтемы:
1. **Семантика доставки** — at-most-once / at-least-once / exactly-once;
   почему настоящего exactly-once в распределёнке нет, и что есть на деле
   (at-least-once + идемпотентность = «эффективно один раз»).
2. **ack/nack** — как брокер понимает, что сообщение обработано; что бывает при
   краше между обработкой и ack (отсюда дубли → нужна идемпотентность).
3. **Idempotency** — идемпотентные обработчики, ключи идемпотентности,
   дедупликация, идемпотентные HTTP-эндпоинты.
4. **Transactional Outbox** — та самая «outbox», название которой друг забыл.
   Проблема двойной записи (БД + брокер) и как outbox её решает; relay/CDC.
5. **Retries + backoff** — экспоненциальный backoff, jitter, ограничение
   попыток, почему «тупой ретрай» опасен.
6. **DLQ (dead letter queue)** — куда уходят зафейленные сообщения, как
   разбирать и переигрывать.
7. **Rate limiting** — token bucket / leaky bucket, где применять.
8. **Crash recovery** — что происходит при падении консьюмера, переигрывание,
   offset-и в Kafka vs ack в RabbitMQ.
9. **Kafka vs RabbitMQ** — модель (лог с offset-ами vs брокер очередей),
   как в каждом реализуются ретраи и «мёртвые» сообщения.

Артефакты: папка [system-design/](./system-design/) — один топик = один файл,
[README.md](./system-design/README.md) = меню навигации.
Статус: ✅ **блок A (messaging/reliability) пройден целиком**, разложен по
файлам:
- [01-foundations](./system-design/01-foundations.md) — CAP/PACELC, трейд-оффы;
- [02-event-driven-messaging](./system-design/02-event-driven-messaging.md) —
  event-driven, семантики доставки;
- [03-reliability-patterns](./system-design/03-reliability-patterns.md) —
  ack/nack, идемпотентность, outbox, retries+backoff, DLQ, rate limiting, crash
  recovery;
- [04-kafka-vs-rabbitmq](./system-design/04-kafka-vs-rabbitmq.md).

Задач для самопроверки по блоку A пока нет — можно добавить.

### Блок B — Scalability & инфраструктура
1. ✅ **[05-scaling](./system-design/05-scaling.md)** — вертикаль/горизонталь
   Node, cluster/PM2, worker_threads (CPU vs I/O-bound), stateless + Redis/БД/S3,
   балансировка/health-checks. Сквозная идея: Node однопоточный → 1 процесс = 1
   ядро.
2. ✅ **[06-scaling-data](./system-design/06-scaling-data.md)** — балансировка
   (L4/L7, алгоритмы, health-checks), репликация (leader/follower, масштаб
   чтений + HA, replication lag, sync/async = CAP-трейд-офф), шардинг (shard key,
   range/hash/directory, hotspot, кросс-шард боль), комбинирование. Ключ:
   репликация масштабирует чтения, шардинг — записи/объём.
3. **Кэширование и CDN** (стратегии, инвалидация) — TODO.

### Блок C — прочее (при желании)
Идемпотентные HTTP API (Idempotency-Key на REST), saga/2PC vs eventual
consistency между сервисами.

---

## Модуль 4 — Databases (транзакции и не только)

Новый модуль. Повод — уровни изоляции (запрос пользователя) плюс естественное
продолжение разговора про транзакции из outbox/идемпотентности. Живёт в
отдельной папке `databases/`, НЕ в system-design (изоляция — про одну БД, не про
распределённость; мешать нельзя).

Многотопиковая область (папка + меню + файлы-топики).
[databases/README.md](./databases/README.md) = меню.

Топики:
1. ✅ **[Уровни изоляции](./databases/01-isolation-levels.md)** (+ ACID и
   аномалии внутри) — RU/RC/RR/Serializable, dirty/non-repeatable/phantom,
   дефолты Postgres (RC) / MySQL (RR), трейд-офф. tasks + answers готовы.
2. **Блокировки и MVCC** — как изоляция реализуется под капотом (TODO).
3. (дальше) индексы, репликация, шардинг.

---

## Порядок работы

1. Модуль 1 (Event Loop) — ✅.
2. Модуль 1.5 (CJS/ESM) — ✅.
3. Модуль 2 (Advanced TS) — ✅.
4. Модуль 3 (System design: messaging/reliability) — ✅ (блок из фидбэка).
5. Модуль 4 (Databases: уровни изоляции) — ✅.
6. System design блок B — масштабирование Node (05) + балансировка/репликация/
   шардинг (06) — ✅.
7. **NestJS — основные моменты** — **следующий** (запрошено). Отдельный модуль
   `nestjs/`.

Дальше по желанию: кэш/CDN, идемпотентные HTTP API, saga/2PC, блокировки/MVCC.
Пользователь накидывает топики — складываем сюда, разбираем по готовности.

По каждому: конспект-рассказ (одна сквозная идея, не тезисно) → задачи на
самопроверку → разборы. Проговариваем вслух (собес устный).
