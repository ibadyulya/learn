# Backend testing — solutions

> 🌐 Russian version: [08-backend-testing.answers.md](../ru/08-backend-testing.answers.md)

---

## Question 1
Tests give the **confidence to change code safely**: on a refactor/new feature,
regressions are caught right away, not in production. Plus they're **executable
documentation** of behavior and **design pressure**: code that's hard to test is usually
poorly designed (tightly coupled, hidden dependencies) — tests highlight that early.

## Question 2
The pyramid: many **fast unit** at the bottom, fewer **integration** in the middle, very
few **e2e** at the top. There are more unit tests because they're **fast, precise, and
cheap** — dense logic coverage in seconds; e2e are slow and brittle, kept few for key
scenarios. The **"ice cream cone"** is the inverted pyramid: tons of slow e2e and few
unit → long runs, brittleness, hard to localize a break.

## Question 3
- **Unit** — one function/class **in isolation**, dependencies mocked; **no** real DB;
  checks logic and edge cases.
- **Integration** — several parts together, often with a **real DB** (a test
  container); checks the seams: a request reached the DB and back, modules assembled.
- **E2E** — a whole scenario from outside over HTTP on a **real** app and DB; checks the
  user path/contract end to end.

## Question 4
- **Stub** — returns predefined answers (data for the test).
- **Mock** — a stub + a check that it was **called** as expected (with which arguments,
  how many times).
- **Spy** — a wrapper around the **real** object, records calls without replacing
  behavior.
- **Fake** — a simplified **working** implementation (an in-memory repository instead of
  a DB).

## Question 5
DIP: a class depends on an **abstraction** and receives the dependency **from outside**
(DI), rather than creating `new` inside. So in a test it's easy to plug in a double:
```ts
const repo: OrderRepository = new InMemoryOrderRepo();   // a fake instead of the DB
const service = new OrderService(repo);                  // injected
```
In Nest — `Test.createTestingModule({...}).overrideProvider(OrderRepository)
.useValue(fakeRepo)`. Without DI you'd have to spin up a real DB even for a unit test.

## Question 6
Testing **behavior** — checking the observable result ("for this input, this
output/effect"), not **how** it's achieved (which private methods were called, the
internal structure). Coupling to the implementation is bad because a **refactor** (that
doesn't change behavior) breaks tests for no reason — they become a drag, not a support.

## Question 7
A **flaky test** — passes sometimes, fails sometimes **with no code change**. It's
dangerous because it erodes trust: the team stops reacting to a red CI ("it always
flickers") and misses real bugs. Typical causes: dependence on **time/timers**,
**network/external services**, **test order** or shared state, races/asynchrony, random
data without a fixed seed.
