# Frontend testing

> 🌐 Russian version: [07-frontend-testing.md](../ru/07-frontend-testing.md)

How to test the UI so tests help rather than hinder. Reads as a story; for general test
principles see [backend-testing](../../04-backend/en/08-backend-testing.md), here it's
the UI specifics.

The through-line of the topic:

> **Test the UI the way a person uses it: check **behavior** (what the user sees and
> does), not implementation details (internal state, method names). The pyramid is the
> same: many fast unit, fewer integration, very few e2e in a real browser.**

---

## The pyramid on the frontend

As on the backend (see [backend-testing](../../04-backend/en/08-backend-testing.md)):
- **Unit** — pure functions, hooks, individual components in isolation; fast, the
  majority.
- **Integration** — several components together + a mocked API: check that a feature works
  as a whole (a form submits data, a list renders the response).
- **E2E** — a whole scenario in a **real browser** through the actual UI (Playwright,
  Cypress): register → log in → order. Maximum confidence, but slow and brittle — keep them
  few.

Most of the value on the frontend is in **integration** tests of components: they're close
to real usage and not as brittle as e2e.

---

## Tools

- **Runner**: **Jest** or **Vitest** (running, assertions, mocks).
- **Components**: **React Testing Library** / **Vue Test Utils** — render a component and
  interact with it.
- **E2E**: **Playwright** / **Cypress** — drive a real browser.
- **Network mocks**: **MSW (Mock Service Worker)** — intercepts HTTP at the network level,
  closer to reality than mocking the request function.

---

## The Testing Library philosophy — "as a user"

The key idea: **find elements the way a user sees them**, not by implementation details.
```tsx
// good: by role/text, as a person perceives it
screen.getByRole('button', { name: /submit/i });
screen.getByLabelText('Email');
await userEvent.click(button);

// bad: by classes/test-ids/internal state
container.querySelector('.btn-primary');   // brittle to markup refactoring
```
Why: a test tied to structure/classes/state **breaks on refactoring** even though behavior
didn't change. A "by the user" test survives refactoring and also checks accessibility
(roles, labels).

---

## What and how to test

- **Behavior and interactions**: clicked — saw the result; entered invalid — saw the error.
  Not "the state became such-and-such".
- **Not implementation details**: internal function names, private state, DOM structure.
- **Mock the external**: the API (via MSW), time, randomness — so tests are
  **deterministic**.
- **Snapshot tests** — carefully: big snapshots are brittle and get "updated without
  looking"; useful for small stable pieces.

The link to the general rule (see [backend-testing](../../04-backend/en/08-backend-testing.md)):
**test behavior, not implementation** — on the frontend that's literally "as a user".

---

## Interview phrasing

> "I test the frontend by the pyramid: many unit (functions, hooks, components in
> isolation), most value in integration tests of components with a mocked API, and very few
> e2e in a real browser (Playwright/Cypress). Tools: Jest/Vitest as the runner, React
> Testing Library / Vue Test Utils for components, MSW for network mocks. The main Testing
> Library principle — find elements and interact **as a user** (by role/text/label), not by
> classes/test-ids/internal state, so tests don't break on refactoring and also check
> accessibility. I test behavior, not implementation; I mock the external (API, time) for
> determinism; snapshots carefully."

---

See [07-frontend-testing.tasks.md](./07-frontend-testing.tasks.md) — tasks. Solutions in
[07-frontend-testing.answers.md](./07-frontend-testing.answers.md).
