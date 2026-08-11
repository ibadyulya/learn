# SOLID — задачи

Отвечай своими словами, потом сверяйся с [03-solid.answers.md](./03-solid.answers.md).
Для каждого фрагмента: назови нарушенный принцип, объясни *почему*, и покажи, как
починить.

> 🌐 English version: [03-solid.tasks.md](../en/03-solid.tasks.md)

---

## Вопрос 1 — теория
Что значит «одна причина для изменения» в SRP? Почему это не то же самое, что
«класс делает одну вещь»? Приведи пример класса, который «делает одно», но
нарушает SRP.

## Вопрос 2 — распознать и починить
```ts
class Invoice {
  calculateTotal() { /* бизнес-правила */ }
  printToPdf()     { /* вёрстка PDF */ }
  saveToDatabase() { /* SQL */ }
  sendByEmail()    { /* SMTP */ }
}
```
Какой принцип нарушен и как это разложить?

## Вопрос 3 — распознать и починить
```ts
function getPrice(product: any): number {
  if (product.type === 'book')    return product.base * 0.9;
  if (product.type === 'food')    return product.base * 1.1;
  if (product.type === 'gadget')  return product.base * 1.2;
  // каждая новая категория — правка этой функции
}
```
Какой принцип нарушен? Перепиши так, чтобы новая категория не требовала менять
существующий код.

## Вопрос 4 — LSP
```ts
class Bird { fly() { /* ... */ } }
class Penguin extends Bird { fly() { throw new Error('пингвины не летают') } }
```
Какой принцип нарушен и в чём именно суть нарушения? Как перепроектировать иерархию?

## Вопрос 5 — ISP
```ts
interface Machine {
  print(doc: Doc): void;
  scan(doc: Doc): void;
  fax(doc: Doc): void;
}
class OldPrinter implements Machine {
  print(doc: Doc) { /* ... */ }
  scan(doc: Doc)  { throw new Error('нет сканера') }
  fax(doc: Doc)   { throw new Error('нет факса') }
}
```
Какой принцип нарушен? Как разбить интерфейс?

## Вопрос 6 — DIP
```ts
class NotificationService {
  private sender = new SmtpEmailSender();
  notify(user: User, text: string) { this.sender.send(user.email, text) }
}
```
Какой принцип нарушен? Примени DIP. И объясни разницу между DIP, DI и IoC.

## Вопрос 7 — связи между принципами
Почему говорят, что OCP «держится на» LSP? Что сломается в OCP-дизайне (полиморфная
диспетчеризация), если нарушить LSP?

## Вопрос 8 — чувство меры
Приведи пример, где буквальное следование SRP или OCP делает код **хуже**
(оверинжиниринг). Как понять, что пора остановиться?
