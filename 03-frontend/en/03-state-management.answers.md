# State management — solutions

> 🌐 Russian version: [03-state-management.answers.md](../ru/03-state-management.answers.md)

---

## Question 1
- **Local** — belongs to one component and isn't needed by others (whether a modal is open,
  input text).
- **Global** — shared by many unrelated components (user, theme, cart).
Rule: **start local**, lift it up/into a store **only when data is really needed by
several** components. Don't move it to global "just in case".

## Question 2
**Prop drilling** — passing props through many intermediate components that don't need the
data, just to deliver it to a deep consumer. Bad: noise, coupling, fragility. Avoid it:
**Context** (React) / **provide-inject** (Vue) — pass it deep without manual forwarding; or
move shared state into a **store**.

## Question 3
**Unidirectional flow (flux):** user → **action** → a handler changes the **store** → the
**view** reads the store → UI (and back to the user). The store changes **only** via
explicit actions, not "from the side" anywhere. This gives **predictability** (clear what
changed and why), convenient debugging (action log, time-travel), and a single source of
truth.

## Question 4
A reducer `(state, action) => newState` is **pure** (no side effects, deterministic) and
returns a **new** object rather than mutating the old. This is needed because change
detection goes **by reference**: a new object → the reference changed → subscribers know to
update; a mutation in place keeps the same reference → the change "goes unnoticed".
Immutability also underpins time-travel (keeping past states as snapshots).

## Question 5
Server data is a **cache of remote state**, not ordinary local state: it has its own
concerns (loading/error, caching, revalidation, staleness, request dedup). Putting it in
Redux by hand is a lot of boilerplate and bugs. Better — **React Query / SWR / TanStack
Query** (or Pinia Colada): they cache, track status, and invalidate. The store then holds
only client UI state.

## Question 6
- **Justified:** complex client state shared by many components with frequent interactions
  (cart, a multi-step form, an editor with undo).
- **Over-engineering:** setting up Redux for a "is the dropdown open" flag or local input
  text — that's local state, global only adds layers and boilerplate.

## Question 7
**Pinia** is simpler because it leans on **Vue's built-in reactivity**: no pure reducers or
immutable updates needed — state can be **mutated directly** (`state.count++`), reactivity
notices it and updates subscribers. Less boilerplate (no explicit action types/dispatch),
an API closer to ordinary objects.
