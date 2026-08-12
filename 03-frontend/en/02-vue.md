# Vue

> 🌐 Russian version: [02-vue.md](../ru/02-vue.md)

A framework with reactivity "out of the box". Reads as a story; the comparison with
[React](./01-react.md) runs throughout.

The through-line of the topic:

> **Vue is built on reactivity: you declare data, and the framework **tracks
> dependencies** and updates only the parts of the view that depend on the changed
> data. Unlike React, where you trigger a re-render by replacing state, Vue notices a
> change to reactive data automatically.**

---

## Reactivity — the core of Vue

You mark data as reactive (`ref`, `reactive`), and Vue **watches who reads it**. When the
data changes, Vue **surgically** updates only the template spots that depend on it. No
"call setState": you assign — the view updates.
```vue
<script setup>
import { ref } from 'vue';
const count = ref(0);           // a reactive cell
function inc() { count.value++; }  // change it — the template updates itself
</script>
<template>
  <button @click="inc">Count: {{ count }}</button>
</template>
```
The mechanics: on render Vue **records dependencies** (which effect read which `ref`); on a
`ref` change it **notifies** only those effects (the observer pattern). That's the
difference from React: React re-renders the whole component and diffs the VDOM, Vue updates
targetedly by the dependency graph.

---

## Composition API vs Options API

- **Options API** (the old style) — the `data`, `methods`, `computed`, `watch` options in
  the component object. Simple for small components, but one feature's logic is smeared
  across different options.
- **Composition API** (modern, `<script setup>`) — all logic in the `setup` function via
  `ref`/`reactive`/`computed`/`watch`. A feature's logic can be gathered together and
  **reused** via **composables** (the analog of React hooks).

---

## computed vs watch

- **`computed`** — a **derived** value that's **cached** and recomputed only when its
  dependencies change. For "derive from data" (a full name from first+last).
  ```js
  const fullName = computed(() => first.value + ' ' + last.value);
  ```
- **`watch`** — a **side effect in response to a change** (a request, a store write,
  logging). For "do something when it changed".
Rule: **compute a value → `computed`; perform an action → `watch`.**

---

## Comparison with React

| | React | Vue |
|---|---|---|
| Update | replace state → component re-render + VDOM diff | change reactive data → targeted update by dependencies |
| Reactivity | "manual" (setState, hook deps) | automatic (tracking) |
| Template | JSX (it's JS) | template syntax (+ optionally JSX) |
| Logic | hooks | Composition API (`ref`/`computed`/`watch`), composables |
| Common | components, one-way data flow, reactive UI | the same |

Both are component-based, declarative, with one-way data flow. The difference is in the
**update mechanism** and style.

---

## Interview phrasing

> "Vue — a framework with reactivity out of the box: I mark data reactive
> (`ref`/`reactive`), Vue tracks dependencies on render and, on a data change, surgically
> updates only the dependent spots — unlike React, where I trigger a re-render by replacing
> state and the VDOM does the diff. I write with the Composition API (`<script setup>`),
> extracting feature logic into composables (like hooks). `computed` — a cached derived
> value, `watch` — a side effect on change. Essentially React and Vue are close
> (components, one-way flow, declarativeness); the difference is the reactivity mechanism:
> explicit re-render + VDOM in React vs auto-tracking in Vue."

---

See [02-vue.tasks.md](./02-vue.tasks.md) — tasks. Solutions in
[02-vue.answers.md](./02-vue.answers.md).
