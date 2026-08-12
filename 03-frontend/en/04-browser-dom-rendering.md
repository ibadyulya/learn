# Browser, DOM and rendering

> 🌐 Russian version: [04-browser-dom-rendering.md](../ru/04-browser-dom-rendering.md)

How the browser turns HTML/CSS into pixels and why it matters for performance. Reads as
a story; it ties the event model to the
[event loop](../../01-js-typescript/en/01-event-loop.md).

The through-line of the topic:

> **The browser turns HTML and CSS into a picture through a pipeline: parse into trees
> (DOM + CSSOM) → build the render tree → compute geometry (layout) → paint → composite
> the layers. Understanding this path explains why some changes are expensive (reflow)
> and others cheap, and how events work.**

---

## The critical rendering path

From bytes to pixels:
1. **HTML → DOM** — the parser builds a **DOM tree** from the tags.
2. **CSS → CSSOM** — a **CSSOM tree** is built from the styles.
3. **Render tree** — DOM + CSSOM merge into a tree of visible nodes with styles (hidden
   `display:none` isn't included).
4. **Layout (reflow)** — **geometry** is computed: the size and position of each element.
5. **Paint** — nodes are **painted** into pixels (colors, text, shadows) by layers.
6. **Composite** — layers are **assembled** on screen (including on the GPU).

JS can interfere at any stage (changing DOM/styles), which triggers a recompute.

---

## Reflow vs repaint — why some changes are costlier

- **Reflow (layout)** — recomputing **geometry**. Triggered by a change affecting
  size/position: width, font, adding/removing a node, reading `offsetHeight`. **Expensive**
  — may recompute a large part of the tree.
- **Repaint** — redrawing **without a geometry change**: color, background, visibility.
  Cheaper than reflow.
- **Composite-only** — `transform`/`opacity` often change **only at the composite stage**
  (on the GPU), without touching layout/paint — **the cheapest** (so you animate
  `transform`, not `top/left`).

Optimization: **batch** DOM changes (not one at a time in a loop), avoid "layout thrashing"
— interleaving style writes and geometry reads (each read forces a reflow), animate
`transform/opacity`.

---

## The DOM — a tree, and it's expensive

The **DOM** — a tree representation of the document, an API to read/change it from JS. DOM
operations are **costlier** than ordinary JS because they can trigger reflow/repaint. So
frameworks (React/Vue) minimize real operations via virtual DOM/reactivity, and by hand you
gather changes and apply them at once (`DocumentFragment`, batching).

---

## The event model

An event (a click) goes through **three phases**: **capturing** (top-down to the target) →
**target** (on the element) → **bubbling** (back up to the root). By default listeners fire
on **bubbling**.

**Event delegation** — put **one** listener on a common parent instead of a listener on
each of thousands of children: the event bubbles up, you check `event.target`. Fewer
listeners → less memory, and it works for dynamically added elements too.

`event.preventDefault()` — cancel the default action; `stopPropagation()` — stop the
bubbling.

---

## The link to the event loop

Rendering and JS share **one thread** (see
[event loop](../../01-js-typescript/en/01-event-loop.md)): while heavy JS runs, the browser
**can't draw** a frame — the page "freezes". So heavy computations are split/moved to web
workers, and UI updates are aligned to the frame via `requestAnimationFrame`.

---

## Interview phrasing

> "The browser builds a DOM tree from HTML, a CSSOM from CSS, merges them into a render
> tree, then layout (geometry) → paint (pixels) → composite (layers). Changes affecting
> geometry cause an expensive reflow; color/background — a cheaper repaint;
> `transform/opacity` — composite-only on the GPU, so I animate those, not `top/left`. DOM
> operations are expensive (they trigger recompute), so I batch changes and avoid layout
> thrashing (interleaving style writes and geometry reads). Events go capture → target →
> bubble; I use delegation — one listener on a parent via bubbling. Render and JS are on
> one thread, heavy JS freezes the frame — I split/move it to a worker, animations via
> requestAnimationFrame."

---

See [04-browser-dom-rendering.tasks.md](./04-browser-dom-rendering.tasks.md) — tasks.
Solutions in [04-browser-dom-rendering.answers.md](./04-browser-dom-rendering.answers.md).
