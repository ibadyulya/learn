# SOLID — solutions

> 🌐 Russian version: [03-solid.answers.md](../ru/03-solid.answers.md)

---

## Question 1
"One reason to change" is **one customer / one axis along which requirements may
change**. "Does one thing" looks at the action; SRP looks at the source of change.

Example of "does one thing but violates SRP": a function that **reads a config and
from it builds and sends an HTTP request**. The action seems singular ("initialize
the client"), but there are two reasons to change — the config format and the
communication protocol; they should be separated. Key: count not actions but **who
might come with a change**.

## Question 2
Violates **SRP**: four reasons to change (calculation rules, PDF format, DB
schema, email delivery) — four different customers. Break it apart:
```ts
class InvoiceCalculator { total(inv: Invoice): Money {} }
class InvoicePdf        { render(inv: Invoice): Buffer {} }
class InvoiceRepository { save(inv: Invoice): void {} }
class InvoiceMailer     { send(inv: Invoice): void {} }
```
`Invoice` stays data; each behavior lives where its changes come from.

## Question 3
Violates **OCP**: a new product category = editing `getPrice`. Polymorphic
solution:
```ts
interface PricingStrategy { price(base: number): number }
class BookPricing   implements PricingStrategy { price(b: number) { return b * 0.9 } }
class FoodPricing   implements PricingStrategy { price(b: number) { return b * 1.1 } }
class GadgetPricing implements PricingStrategy { price(b: number) { return b * 1.2 } }
// a new category = a new strategy class; getPrice/the dispatcher don't change
```
(This is the Strategy pattern — a direct application of OCP.)

## Question 4
Violates **LSP**: `Penguin` is a subtype of `Bird` but can't honestly substitute
where a flying bird is expected (it throws on `fly()`). The code
`birds.forEach(b => b.fly())` fails on the penguin. The essence — the subclass
**narrowed** the parent's contract.

Redesign — model by behavior, not by "a penguin is a bird":
```ts
interface Bird {}
interface Flying { fly(): void }
class Sparrow implements Bird, Flying { fly() {} }
class Penguin implements Bird { /* no fly */ }
```
The ability to fly is a separate role, not a property of every bird.

## Question 5
Violates **ISP**: the fat `Machine` forces `OldPrinter` to implement `scan`/`fax`
as stubs. Split by capability:
```ts
interface Printer { print(doc: Doc): void }
interface Scanner { scan(doc: Doc): void }
interface Fax     { fax(doc: Doc): void }

class OldPrinter implements Printer { print(doc: Doc) {} }
class Mfp implements Printer, Scanner, Fax { print(){} scan(){} fax(){} }
```
A client that only needs printing depends only on `Printer`.

## Question 6
Violates **DIP**: the high-level `NotificationService` is nailed to the concrete
`SmtpEmailSender`. Invert via an abstraction + injection:
```ts
interface Notifier { send(to: string, text: string): void }

class NotificationService {
  constructor(private notifier: Notifier) {}
  notify(user: User, text: string) { this.notifier.send(user.email, text) }
}
class SmtpNotifier implements Notifier { send(to: string, text: string) {} }
class SmsNotifier  implements Notifier { send(to: string, text: string) {} }
```
The difference:
- **DIP** — the principle: depend on the abstraction `Notifier`, not on SMTP.
- **DI** — how we did it: passed `notifier` into the constructor from outside.
- **IoC** — the general principle "hand creation/wiring control outward" (to a
  container/framework); DI is a special case of it.

## Question 7
OCP wants to add behavior through polymorphism: the code calls `shape.area()` /
`strategy.price()` without knowing the concrete type, and a new type just
substitutes in. This works **only if any subtype honestly behaves like the base**
— i.e. under LSP. If a subtype lies (throws, changes semantics), the polymorphic
call breaks on that implementation, and you're forced back to `if (instanceof)` —
exactly what OCP removed. So extensibility-without-modification rests on
substitutability.

## Question 8
Over-engineering: introducing an `interface` + factory + strategy for code that
has **one** implementation with no sign a second will appear — that's abstraction
"just in case" that only adds layers and indirection. Or splitting a class by SRP
so finely that a simple operation smears across five files.

The guide — **the rule of three / YAGNI**: close for extension (OCP) and introduce
an abstraction (DIP) when a change/duplicate has **actually repeated**, not on a
hunch. The first `if` is fine; the third identical one is the signal for
polymorphism. The principles serve readability and changeability; if applying them
makes the code more complex with no payoff, you're using them against their own
goal.
