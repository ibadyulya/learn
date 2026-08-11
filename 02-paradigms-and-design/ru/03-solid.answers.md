# SOLID — разборы

> 🌐 English version: [03-solid.answers.md](../en/03-solid.answers.md)

---

## Вопрос 1
«Одна причина для изменения» — это **один заказчик / одна ось, по которой
требования могут поменяться**. «Делает одну вещь» смотрит на действие, SRP — на
источник изменений.

Пример «делает одно, но нарушает SRP»: функция, которая **читает конфиг и из него
же собирает и шлёт HTTP-запрос**. Действие вроде одно («инициализировать клиент»),
но причин меняться две — формат конфига и протокол общения; их стоит разделить.
Ключ: считаем не действия, а **кто может прийти с изменением**.

## Вопрос 2
Нарушен **SRP**: четыре причины меняться (правила расчёта, формат PDF, схема БД,
доставка почты) — четыре разных заказчика. Разложить:
```ts
class InvoiceCalculator { total(inv: Invoice): Money {} }
class InvoicePdf        { render(inv: Invoice): Buffer {} }
class InvoiceRepository { save(inv: Invoice): void {} }
class InvoiceMailer     { send(inv: Invoice): void {} }
```
`Invoice` остаётся данными; каждое поведение живёт там, откуда приходят его
изменения.

## Вопрос 3
Нарушен **OCP**: новая категория товара = правка `getPrice`. Полиморфное решение:
```ts
interface PricingStrategy { price(base: number): number }
class BookPricing   implements PricingStrategy { price(b: number) { return b * 0.9 } }
class FoodPricing   implements PricingStrategy { price(b: number) { return b * 1.1 } }
class GadgetPricing implements PricingStrategy { price(b: number) { return b * 1.2 } }
// новая категория = новый класс-стратегия; getPrice/диспетчер не меняются
```
(Это паттерн «Стратегия» — прямое применение OCP.)

## Вопрос 4
Нарушен **LSP**: `Penguin` — подтип `Bird`, но не может честно подставиться туда,
где ждут летающую птицу (бросает на `fly()`). Код `birds.forEach(b => b.fly())`
падает на пингвине. Суть — наследник **сузил контракт** предка.

Перепроектирование — моделировать по поведению, а не по «пингвин это птица»:
```ts
interface Bird {}
interface Flying { fly(): void }
class Sparrow implements Bird, Flying { fly() {} }
class Penguin implements Bird { /* без fly */ }
```
Способность летать — отдельная роль, а не свойство всякой птицы.

## Вопрос 5
Нарушен **ISP**: толстый `Machine` заставляет `OldPrinter` реализовывать
`scan`/`fax` заглушками. Разбить по возможностям:
```ts
interface Printer { print(doc: Doc): void }
interface Scanner { scan(doc: Doc): void }
interface Fax     { fax(doc: Doc): void }

class OldPrinter implements Printer { print(doc: Doc) {} }
class Mfp implements Printer, Scanner, Fax { print(){} scan(){} fax(){} }
```
Клиент, которому нужна только печать, зависит только от `Printer`.

## Вопрос 6
Нарушен **DIP**: высокоуровневый `NotificationService` прибит к конкретному
`SmtpEmailSender`. Инверсия через абстракцию + внедрение:
```ts
interface Notifier { send(to: string, text: string): void }

class NotificationService {
  constructor(private notifier: Notifier) {}
  notify(user: User, text: string) { this.notifier.send(user.email, text) }
}
class SmtpNotifier implements Notifier { send(to: string, text: string) {} }
class SmsNotifier  implements Notifier { send(to: string, text: string) {} }
```
Разница:
- **DIP** — принцип: зависеть от абстракции `Notifier`, а не от SMTP.
- **DI** — как мы это сделали: передали `notifier` в конструктор снаружи.
- **IoC** — общий принцип «управление созданием/связыванием отдаём наружу»
  (контейнеру/фреймворку); DI — его частный случай.

## Вопрос 7
OCP хочет добавлять поведение через полиморфизм: код зовёт `shape.area()` /
`strategy.price()`, не зная конкретного типа, и новый тип просто подставляется.
Это работает, **только если любой подтип честно ведёт себя как базовый** — то есть
при LSP. Если подтип врёт (бросает, меняет семантику), полиморфный вызов
ломается на конкретной реализации, и приходится возвращать `if (instanceof)` —
ровно то, что OCP убирал. Значит расширяемость без модификации держится на
подставляемости.

## Вопрос 8
Оверинжиниринг: заводить `interface` + фабрику + стратегию для кода, у которого
**одна** реализация и нет признаков, что появится вторая — это абстракция «на
всякий случай», которая только добавляет слоёв и косвенности. Или дробить класс по
SRP так мелко, что простая операция размазывается по пяти файлам.

Ориентир — **правило трёх / YAGNI**: закрывай под расширение (OCP) и вводи
абстракцию (DIP) тогда, когда изменение/дубликат **реально повторился**, а не по
предчувствию. Первый `if` — норма, третий такой же — сигнал к полиморфизму.
Принципы служат читаемости и изменяемости; если применение делает код сложнее без
выгоды — ты применяешь их против их же цели.
