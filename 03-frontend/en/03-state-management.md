# State management

> 🌐 Russian version: [03-state-management.md](../ru/03-state-management.md)

Where and how to store UI data. Reads as a story; adjacent to [React](./01-react.md)
and [Vue](./02-vue.md).

The through-line of the topic:

> **State — the data the interface shows and changes. The key question isn't "which
> store to take" but **where** the state should live: locally in a component or
> globally for many. A global store is needed when data is shared by distant parts of
> the UI — but not everything goes there; an unnecessary global store adds
> complexity.**

---

## Local vs global state

- **Local** — lives in a component (`useState`/`ref`): whether a dropdown is open, input
  text. Doesn't concern others — keep it close.
- **Global** — shared by many unrelated components: the current user, the theme, the
  cart. Lives in a **store** or context.

Rule: **start local**, lift it up only when data is really needed by several. A premature
global store is over-engineering.

---

## The prop drilling problem

If state is high but needed deep, you have to **pass props through many layers** (prop
drilling) that don't need the data — noise and coupling. Solutions:
- **Context** (React) / provide-inject (Vue) — pass data deep without manual forwarding;
- **a store** — move shared state outside the components.

---

## Flux — unidirectional flow

The idea Redux/Pinia stand on: data flows **one way**, changes are predictable.
```
action → a handler changes the store → the view reads the store → UI
   ▲                                                                │
   └──────────────────── the user interacts ◄──────────────────────┘
```
Nobody changes the store "from the side": only via explicit actions. Hence
predictability, time-travel debugging, loggability.

---

## Redux / Pinia

- **Redux** (React): a single **store**, changes only via **actions** → **reducers** —
  **pure functions** `(state, action) => newState` returning a **new** state
  ([immutability](../../02-paradigms-and-design/en/02-functional-programming.md) matters;
  change detection and time-travel stand on it). Modern — Redux Toolkit (less
  boilerplate).
- **Pinia** (Vue): simpler, uses Vue's reactivity — a store with `state`/`getters`/`actions`,
  you can mutate state directly (reactivity notices it).

Both implement a "single source of truth" + unidirectional flow; subscribers update on
change (the observer pattern).

---

## Client state ≠ server state

A common mistake — dragging **everything** into Redux, including data from the server. But
server data is essentially a **cache** of remote state with its own concerns (loading,
caching, revalidation, staleness). For it, dedicated tools are better — **React Query / SWR
/ TanStack Query** (or Pinia Colada): they cache responses, track request status, and
invalidate. Then the global store holds only **client** UI state.

---

## When to use what

- **Local** — by default.
- **Context/provide** — theme, locale, current user: rarely changes, needed by many.
- **A store (Redux/Pinia)** — complex shared client state, many interactions.
- **A server-state library** — data from the backend (lists, entities).

---

## Interview phrasing

> "I split state into local (in a component) and global (shared by many); by default I
> keep it local and lift only when really needed. Against prop drilling —
> context/provide or a store. Global stores (Redux/Pinia) implement flux: a
> unidirectional flow action → store change → view, a single source of truth; in Redux
> reducers are pure functions returning a new state (immutability is needed, change
> detection and time-travel stand on it). I don't keep server data in Redux — it's a cache
> of remote state, for it React Query/SWR with its own cache and revalidation. I don't
> move everything into global — that adds complexity."

---

See [03-state-management.tasks.md](./03-state-management.tasks.md) — tasks. Solutions in
[03-state-management.answers.md](./03-state-management.answers.md).
