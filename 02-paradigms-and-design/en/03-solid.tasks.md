# SOLID — tasks

Answer in your own words, then check against [03-solid.answers.md](./03-solid.answers.md).
For each snippet: name the violated principle, explain *why*, and show how to fix
it.

> 🌐 Russian version: [03-solid.tasks.md](../ru/03-solid.tasks.md)

---

## Question 1 — theory
What does "one reason to change" mean in SRP? Why isn't it the same as "a class
does one thing"? Give an example of a class that "does one thing" yet violates SRP.

## Question 2 — spot and fix
```ts
class Invoice {
  calculateTotal() { /* business rules */ }
  printToPdf()     { /* PDF layout */ }
  saveToDatabase() { /* SQL */ }
  sendByEmail()    { /* SMTP */ }
}
```
Which principle is violated and how do you break this apart?

## Question 3 — spot and fix
```ts
function getPrice(product: any): number {
  if (product.type === 'book')    return product.base * 0.9;
  if (product.type === 'food')    return product.base * 1.1;
  if (product.type === 'gadget')  return product.base * 1.2;
  // every new category = editing this function
}
```
Which principle is violated? Rewrite it so a new category doesn't require changing
existing code.

## Question 4 — LSP
```ts
class Bird { fly() { /* ... */ } }
class Penguin extends Bird { fly() { throw new Error('penguins cannot fly') } }
```
Which principle is violated and what exactly is the violation? How do you redesign
the hierarchy?

## Question 5 — ISP
```ts
interface Machine {
  print(doc: Doc): void;
  scan(doc: Doc): void;
  fax(doc: Doc): void;
}
class OldPrinter implements Machine {
  print(doc: Doc) { /* ... */ }
  scan(doc: Doc)  { throw new Error('no scanner') }
  fax(doc: Doc)   { throw new Error('no fax') }
}
```
Which principle is violated? How do you split the interface?

## Question 6 — DIP
```ts
class NotificationService {
  private sender = new SmtpEmailSender();
  notify(user: User, text: string) { this.sender.send(user.email, text) }
}
```
Which principle is violated? Apply DIP. And explain the difference between DIP, DI
and IoC.

## Question 7 — links between principles
Why is OCP said to "rest on" LSP? What breaks in an OCP design (polymorphic
dispatch) if you violate LSP?

## Question 8 — sense of proportion
Give an example where literally following SRP or OCP makes the code **worse**
(over-engineering). How do you know it's time to stop?
