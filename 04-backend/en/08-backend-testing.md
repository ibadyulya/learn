# Backend testing

> 🌐 Russian version: [08-backend-testing.md](../ru/08-backend-testing.md)

Why and how to test server code. Reads as a story.

The through-line of the topic:

> **Tests give you the confidence to change code without fear of breaking
> everything. The pyramid: many fast unit tests at the bottom, fewer integration,
> even fewer slow e2e at the top. Test **behavior**, not implementation, and isolate
> dependencies with test doubles — and the ease of isolation comes from
> [DI/DIP](../../02-paradigms-and-design/en/03-solid.md).**

---

## Why test

Not "for show" but to **change code safely**: you refactor/add a feature — tests catch
regressions right away, not in production. Tests are also behavior documentation and
design pressure (hard-to-test code is usually poorly designed — tightly coupled).

---

## The test pyramid

Bottom to top: more — faster — cheaper → fewer — slower — costlier.
```
        /\
       /e2e\        few: the whole path over HTTP, a real DB (slow, brittle)
      /------\
     / integ. \     medium: modules together, DB/queue
    /----------\
   /   unit     \   many: one function/class in isolation (fast)
  /--------------\
```
- **Unit** — one unit of logic **in isolation**, dependencies mocked. Fast, precise,
  the majority.
- **Integration** — several parts together (a service + a real DB, a Nest module);
  they check the seams.
- **E2E** — a whole scenario from outside: an HTTP request → the real app → the
  response. Maximum confidence, but slow and brittle — keep them few.

The anti-pattern is the "ice cream cone" (inverted pyramid): tons of slow e2e and few
unit.

---

## Test doubles — isolating dependencies

To test a unit in isolation, its dependencies are replaced by "doubles":
- **Stub** — returns predefined answers (`repo.find → a fixed user`).
- **Mock** — a stub + a check that it was **called** as expected (`expect(mailer.send)
  .toHaveBeenCalledWith(...)`).
- **Spy** — a wrapper around the real one, watches calls.
- **Fake** — a working simplified implementation (an in-memory repository instead of a
  DB).

This is where **DIP/DI** pays off: since the service depends on the `OrderRepository`
abstraction and receives it via the constructor, in a test you plug in a fake/mock with
no DB at all (see [nestjs](./02-nestjs.md) — `Test.createTestingModule` with provider
overrides).

---

## What to test where

- **Unit** — business logic/calculations in pure form, branches, edge cases; DB and
  network mocked.
- **Integration** — that a request really reaches the DB and back, that modules
  assemble, that migrations/schemas match. Often against a **test container** of the
  real DB.
- **E2E** — key user scenarios end-to-end (register → log in → order), API contract
  checks.
- **Contract tests** — that the API matches consumers' expectations (between services).

---

## Signs of a good test

- **Tests behavior, not implementation** — "for this input, this output", not "called
  this private method". Otherwise refactoring breaks tests for no reason.
- **Deterministic** — doesn't depend on time/network/order; "flaky" tests are worse
  than none.
- **Fast and isolated** — doesn't drag in half the system to check one function.
- **AAA** — Arrange (set up) → Act (call) → Assert (check); one test — one idea.

---

## Interview phrasing

> "Tests exist to change code without fear of regressions, and they also pressure the
> design. I keep the pyramid: many fast unit tests (a unit in isolation, dependencies
> mocked), fewer integration (a service + a real DB, the seams), very few e2e (the
> whole path over HTTP). I isolate dependencies with test doubles (stub/mock/spy/fake),
> which is easy thanks to DI/DIP — I plug a fake repository in for the DB (in Nest — a
> TestingModule with provider overrides). I test behavior, not implementation, so
> refactoring doesn't break tests; I keep tests deterministic and fast, in the
> Arrange-Act-Assert shape."

---

See [08-backend-testing.tasks.md](./08-backend-testing.tasks.md) — tasks. Solutions in
[08-backend-testing.answers.md](./08-backend-testing.answers.md).
