# Vue — solutions

> 🌐 Russian version: [02-vue.answers.md](../ru/02-vue.answers.md)

---

## Question 1
**Reactivity** — data marked reactive (`ref`/`reactive`) that Vue **watches**. On render
Vue **records dependencies** (which template spot/effect read which reactive source). When
the data changes, Vue **notifies** exactly those dependencies and **surgically** re-renders
only them. No manual "tell it what to update": you assign — it updates.

## Question 2
- **React** — you change `state` (setState) → the component **re-renders wholly**, React
  builds a new VDOM and **diffs** it against the old, applying the differences.
- **Vue** — you change reactive data → by the **dependency graph** Vue updates **only** the
  affected nodes, without a full component re-render.
React re-renders and compares, Vue updates targetedly by tracking.

## Question 3
- **Options API** — logic split across `data`/`methods`/`computed`/`watch` options; simple,
  but one feature is smeared across different sections.
- **Composition API** — all logic in `setup` (`ref`/`computed`/`watch`); it appeared to
  **group logic by feature** and **reuse** it via composables (in Options this was done
  with mixins — implicit and conflict-prone).

## Question 4
- **`computed`** — for **derived values**: cached, recomputed only when dependencies
  change, returns the cache on re-read. Example: `fullName` from `first`+`last`.
- **`watch`** — for **side effects** on change: a request, a write, a log. "X changed → do
  Y".
`computed` is more efficient for deriving data because it **doesn't recompute needlessly**
(cache) and is declarative; `watch` is an imperative reaction.

## Question 5
A **composable** — a function encapsulating **reusable reactive logic** (e.g., `useMouse()`,
`useFetch()`), built on the Composition API. The direct analog — a **custom React hook**
(`useSomething`): the same idea of extracting stateful/effectful logic into a reusable
function.

## Question 6
In common: both are **component-based**, **declarative** (UI follows data), with **one-way
data flow** and reactive updating; both encourage composition from reusable pieces
(hooks/composables). The fundamental difference — the **reactivity mechanism**: React
re-renders the component and diffs the VDOM (reactivity is "manual", via state/deps), Vue
automatically tracks dependencies and updates targetedly.

## Question 7
`ref(0)` wraps the value in a **reactive object** with a `.value` field — in JS code you
access/change it via `count.value` (otherwise you lose reactivity working with the bare
number). In the **template** Vue **auto-unwraps** a top-level ref, so you write just
`count` — no `.value` needed. It's a template syntactic convenience.
