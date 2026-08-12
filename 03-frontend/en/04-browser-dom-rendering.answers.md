# Browser, DOM and rendering — solutions

> 🌐 Russian version: [04-browser-dom-rendering.answers.md](../ru/04-browser-dom-rendering.answers.md)

---

## Question 1
1. **HTML → DOM** (a tree of elements);
2. **CSS → CSSOM** (a tree of styles);
3. **Render tree** — merging DOM+CSSOM, only visible nodes with styles;
4. **Layout (reflow)** — computing geometry (sizes/positions);
5. **Paint** — painting into pixels by layers;
6. **Composite** — assembling the layers on screen (often on the GPU).

## Question 2
- **Reflow (layout)** — recomputing **geometry**; triggered by size/position changes
  (width, font, adding a node) and **reading** geometry (`offsetHeight`). Expensive.
- **Repaint** — redrawing **without geometry** (color, background, visibility). Cheaper.
- **Composite** — assembling already-painted layers; `transform`/`opacity` often change
  **only here** (on the GPU). The cheapest.

## Question 3
`top/left/width` change **geometry** → a full **reflow** (layout recompute) + repaint +
composite on every animation frame — expensive and janky. `transform`/`opacity` are usually
handled **only at the composite stage** (on the GPU), **without touching** layout and paint
— smooth and cheap. So shifts/scale use `transform: translate/scale`, and transparency uses
`opacity`.

## Question 4
**Layout thrashing** — repeatedly **interleaving style writes and geometry reads** in a
loop: each read (`offsetHeight`, `getBoundingClientRect`) **forces a reflow** to return an
up-to-date value — dozens of times, killing performance. Avoid it: **group** — all **reads**
first, then all **writes** (batch read/write); don't read geometry right after a style write
in a loop.

## Question 5
1. **Capturing** — the event goes **top-down** from the root to the target element;
2. **Target** — reaches the element itself;
3. **Bubbling** — **bubbles** back up to the root.
By default listeners (`addEventListener` without `capture:true`) fire on **bubbling**.

## Question 6
**Delegation** — put **one** listener on a common parent instead of one on each child; the
event bubbles, and in the handler you check `event.target` to see where the click was.
Advantages: **fewer listeners** (memory/performance), **works for dynamically added**
elements (no need to re-attach), simpler to manage.

## Question 7
Browser rendering and your JS share **one thread**. While heavy **synchronous** JS runs, the
event loop is busy and the browser **can't draw** the next frame — visually the page freezes
(unresponsive, no animation). The event-loop link: a long task blocks the loop. Solutions:
**split** the work (in chunks, yielding control), move it to a **web worker** (another
thread), align updates to the frame via `requestAnimationFrame`.
