# React

> 🌐 Russian version: [01-react.md](../ru/01-react.md)

A library for building UIs from components. Reads as a story; builds on
[functional programming](../../02-paradigms-and-design/en/02-functional-programming.md)
and [closures](../../01-js-typescript/en/04-closures-scope.md).

The through-line of the topic:

> **React is `UI = f(state)`: you describe how the interface looks for a given state,
> not how to change it step by step. Change the state → React recomputes the markup and
> **efficiently** updates the real DOM via diffing (reconciliation). Hooks give state
> and effects to function components.**

---

## The component model and declarativeness

The UI is assembled from **components** — functions returning markup (JSX) from their
input. The key shift is **declarativeness**: you describe the **result** for the current
state, not imperatively poke the DOM (`el.textContent = ...`). "How to update" is React's
job.
```tsx
function Counter({ count }: { count: number }) {
  return <div>Count: {count}</div>;   // what to show for this count
}
```
Data flows **top-down** (props), one-way — predictable.

---

## Virtual DOM and reconciliation

The real DOM is slow to update. React keeps a **lightweight copy in memory (virtual
DOM)**. On a state change it builds a new virtual tree, **diffs** it against the old one,
and applies **only the differences** to the real DOM — this is **reconciliation**.
- the diff goes by position and element **type**;
- in lists you need a **`key`** — a stable identifier so React knows which elements are
  the same and which are new/removed (otherwise it re-renders extra or mixes up state).
  **Not the array index** if the list changes.

---

## Hooks — state and effects in functions

- **`useState`** — local state; a change via the setter **triggers a re-render**.
  ```tsx
  const [count, setCount] = useState(0);
  setCount(c => c + 1);   // update from the previous, not the closed-over value
  ```
- **`useEffect`** — side effects (requests, subscriptions, timers) **after** render. The
  dependency array controls when it runs; the cleanup function removes
  subscriptions/timers (see [memory/leaks](../../01-js-typescript/en/09-memory-garbage-collection.md)).
  ```tsx
  useEffect(() => {
    const id = setInterval(tick, 1000);
    return () => clearInterval(id);      // cleanup
  }, []);                                 // [] = once on mount
  ```
- **`useMemo`/`useCallback`** — memoize an expensive value/function between renders.
- **`useContext`** — pass data without "prop drilling".
- **`useRef`** — a mutable reference that doesn't trigger a re-render (DOM access, holding
  a value between renders).

The rules of hooks: call them **at the top level** of the component, always in the same
order (React matches them by call order).

---

## The re-render model and performance

State changed → the component (and its subtree) **re-renders** → reconciliation →
targeted DOM update. The re-render is **virtual and cheap**, but excess ones aren't free.
Optimization:
- **`React.memo`** — don't re-render a component if its props didn't change (by reference
  → hence [immutability](../../02-paradigms-and-design/en/02-functional-programming.md)
  matters);
- **`useMemo`/`useCallback`** — stabilize object/function props;
- correct **`key`s** in lists.
Don't optimize prematurely — measure first (React DevTools Profiler).

A subtlety with closures: a callback in a hook "closes over" the values of the render it
was created in — hence a "stale closure" (an outdated value), fixed with dependencies and
the functional setter update.

---

## Interview phrasing

> "React — a declarative UI as a function of state: I describe what to show for a given
> state, not how to change the DOM. It keeps a virtual DOM; on a state change it builds a
> new tree, diffs it against the old (reconciliation), and applies only the differences to
> the real DOM; lists need a stable key. Hooks give function components state (useState →
> re-render), effects (useEffect with dependencies and cleanup), memoization
> (useMemo/useCallback), context, ref. Data flows top-down. I optimize excess re-renders
> with React.memo + stable props (immutability matters), but by measurement, not
> prematurely. I keep the stale closure in hooks in mind."

---

See [01-react.tasks.md](./01-react.tasks.md) — tasks. Solutions in
[01-react.answers.md](./01-react.answers.md).
