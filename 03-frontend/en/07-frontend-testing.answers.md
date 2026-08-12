# Frontend testing — solutions

> 🌐 Russian version: [07-frontend-testing.answers.md](../ru/07-frontend-testing.answers.md)

---

## Question 1
The pyramid: many **unit** at the bottom (pure functions, hooks, components in isolation),
**integration** in the middle (several components + a mocked API), few **e2e** in a real
browser at the top. Most value — in **integration tests of components**: they're close to
real usage (checking a feature as a whole) while **not as slow and brittle** as e2e.

## Question 2
The principle: **find elements and interact with them the way a user does** — by visible
role, text, label (`getByRole`, `getByLabelText`), not by internal classes or
`data-testid`. By role/text is better because it's **what a person sees and works with**:
the test doesn't depend on markup structure and also **checks accessibility** (is there a
role/label). Classes/test-ids are an implementation detail that's brittle to couple to.

## Question 3
Testing behavior = checking the **observable result of an interaction** ("clicked Submit
with an empty email → I see an error"), not the internals.
```tsx
// bad (implementation): reaching into internal state/structure
expect(wrapper.state('isValid')).toBe(false);

// good (behavior): as a user
await userEvent.click(screen.getByRole('button', { name: /submit/i }));
expect(screen.getByText(/email is required/i)).toBeInTheDocument();
```

## Question 4
- **Running/assertions/mocks:** Jest or Vitest.
- **Components:** React Testing Library / Vue Test Utils.
- **E2E (real browser):** Playwright / Cypress.
- **Network mocks:** MSW (Mock Service Worker) — HTTP interception at the network level.

## Question 5
Because such tests check **how** it's done (a private method name, a state field, a class,
DOM structure), not **what** resulted. Refactoring changes the implementation **while
preserving behavior** — but the test sees the changed internals and **fails for no reason**.
Such tests create false positives and slow down refactoring instead of giving confidence.

## Question 6
You should mock the **external and non-deterministic**: the **API/network** (via MSW),
**time/timers** (fake timers), **randomness** (`Math.random`, uuid). Why — so the test is
**deterministic** (the same result each run) and **fast/isolated** (doesn't hit the real
network). Otherwise tests become flaky and slow.

## Question 7
A big **snapshot** fixes the whole component output as the "reference". The danger: (1) it's
**brittle** — breaks on any small markup change, even a harmless one; (2) on failure
developers often **update the snapshot without looking** (`--u`), and it stops catching
anything — it loses meaning. Only **small** snapshots of stable pieces are useful; behavior
is better checked with explicit assertions.
