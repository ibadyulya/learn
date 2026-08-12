# React — solutions

> 🌐 Russian version: [01-react.answers.md](../ru/01-react.answers.md)

---

## Question 1
"UI = f(state)" — the interface is a **function of state**: for a given state there's an
unambiguous markup. You describe **what to show**, and React decides how to update the DOM.
Imperatively (`el.textContent = x`, `el.classList.add`) you manually poke specific nodes
step by step — easy to desync UI and data. Declaratively — you change only the state, and
the view follows it.

## Question 2
The **virtual DOM** — a lightweight copy of the UI tree in memory. **Reconciliation** — on
a state change React builds a new virtual tree, **diffs** it against the old one, and
applies **only the differences** to the real DOM (the real DOM is expensive to update). A
**`key`** — a stable identifier of a list element so React knows who's "the same" and
who's new/removed. **The index is bad** if the list changes order/content: on
insert/delete the indices shift → React thinks the "wrong" elements changed → excess
re-renders and mixed-up state (e.g., input text sticks to the wrong row).

## Question 3
`useEffect` runs **side effects after render** (requests, subscriptions, timers, DOM work).
The **dependency array** decides when to re-run the effect: `[]` — once on mount; `[x]` —
when `x` changes; no array — after every render. The **cleanup function** (return) removes
subscriptions/timers before the next run/unmount. A leak without cleanup:
```tsx
useEffect(() => { const id = setInterval(tick, 1000); }, []);   // forgot clearInterval
```
The interval lives after unmount, holding closed-over data — a leak.

## Question 4
- **`useMemo`** — caches a **value** (the result of an expensive computation) between
  renders, recomputes when dependencies change.
- **`useCallback`** — caches a **function** (a stable reference) so a new one isn't created
  each render (matters for props in `React.memo`).
- **`useRef`** — a **mutable reference** that **doesn't trigger a re-render**; for accessing
  a DOM node or holding a value between renders (timers, "previous value").

## Question 5
`setCount(count + 1)` uses `count` **closed over in this render** — with several updates in
a row (or in a callback closure) the value goes stale and updates "eat" each other.
`setCount(c => c + 1)` takes the **current previous** value from React's update queue —
reliable under batching and in closures. The closure link: a callback remembers the
`count` from its creation moment (stale closure), the functional form sidesteps it.

## Question 6
Ways: **`React.memo`** (skip a re-render when props are unchanged),
**`useMemo`/`useCallback`** (stabilize object/function props), correct **`key`s**,
splitting components, lifting/localizing state. `React.memo` needs **immutability** because
it compares props **by reference**: if you mutate an object in place, the reference is the
same → memo thinks "unchanged" and skips a needed re-render; a new object (`{...obj}`) → a
new reference → a correct update.

## Question 7
A **stale closure** — a callback/effect "closed over" the values of **the render** it was
created in, and on later changes it sees the **outdated** value (e.g., `count` always 0 in
an interval set up with `[]`). Avoid it: put the value in the effect's **dependencies**,
use the **functional update** of the setter (`setX(prev => ...)`), or `useRef` for an
"always fresh" value.
