# Уровни изоляции — задачи

> 🌐 English version: [01-isolation-levels.tasks.en.md](./01-isolation-levels.tasks.en.md)

Отвечай своими словами, потом сверяйся с
[01-isolation-levels.answers.ru.md](./01-isolation-levels.answers.ru.md).

---

## Вопрос 1 — назвать аномалию
Определи, какая это аномалия, и почему:
```
T1: SELECT price FROM items WHERE id=7;      -- 100
T2:   UPDATE items SET price=150 WHERE id=7; COMMIT;
T1: SELECT price FROM items WHERE id=7;      -- 150
```

## Вопрос 2 — назвать аномалию
```
T1: UPDATE stock SET qty=0 WHERE id=3;   -- НЕ commit
T2:   SELECT qty FROM stock WHERE id=3;  -- 0
T1: ROLLBACK;
```
Какая аномалия? На каком минимальном уровне изоляции она исчезнет?

## Вопрос 3 — dirty vs non-repeatable
В чём принципиальная разница между dirty read и non-repeatable read? Обе ведь про
«прочитал не то».

## Вопрос 4 — non-repeatable vs phantom
Чем phantom read отличается от non-repeatable read? Через какие SQL-операции
возникает каждая?

## Вопрос 5 — уровень под задачу
Какой уровень изоляции минимально достаточен, чтобы: (а) никогда не читать
незакоммиченные данные; (б) внутри транзакции дважды прочитать одну строку и
гарантированно получить то же значение?

## Вопрос 6 — подвох про дефолты
Собеседующий: «Какой уровень изоляции по умолчанию в PostgreSQL и в MySQL? И
правда ли, что на дефолтном уровне MySQL возможны фантомы?» Ответь.

## Вопрос 7 — цена Serializable
Почему Serializable «дорогой»? Что конкретно приходится делать приложению,
работающему на этом уровне в PostgreSQL, чего не нужно на Read Committed?
