# HTML and CSS

> 🌐 Russian version: [06-html-css.md](../ru/06-html-css.md)

The basics of markup and styling that even "JS people" get asked. Reads as a story.

The through-line of the topic:

> **HTML describes **meaning** (what the content is), CSS — **presentation** (how it
> looks). Keep them separated: semantic HTML gives accessibility and SEO, while CSS lays
> out and styles via the box model, flex/grid, and the cascade. A separate concern is
> accessibility (a11y).**

---

## Semantic HTML — meaning, not "divs"

Semantic tags (`<header>`, `<nav>`, `<main>`, `<article>`, `<button>`, `<label>`) carry
**meaning**, unlike faceless `<div>/<span>`. Why:
- **accessibility** — screen readers understand the structure (headings, navigation,
  buttons);
- **SEO** — search engines parse content better;
- **predictability** — a `<button>` is already keyboard-clickable, a `<label>` is tied to
  an input.
Rule: a tag **by meaning**, not "a div for everything" + JS plumbing.

---

## The box model

Every element is a rectangle of layers from inside out: **content → padding → border →
margin**. By default `width` counts **only content** (`box-sizing: content-box`), so
padding/border are **added** on top and "break" the width. The practice — globally:
```css
* { box-sizing: border-box; }   /* width includes padding and border — predictable */
```
`margin` collapses between vertical neighbors (collapsing) — a frequent surprise.

---

## Layout: flexbox vs grid

- **Flexbox** — **one-dimensional** layout (a row **or** a column): alignment, distributing
  space, order. Ideal for toolbars, button rows, centering.
  ```css
  .row { display: flex; gap: 8px; justify-content: space-between; align-items: center; }
  ```
- **Grid** — **two-dimensional** layout (rows **and** columns at once): page grids, cards,
  complex layouts.
  ```css
  .grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; }
  ```
Rule: **1D → flex, 2D → grid**; often together (grid for the page, flex inside blocks).

---

## The cascade and specificity

When several rules set one property, the **more specific** one wins. A rough specificity
hierarchy (ascending): tag < class < id < inline style; `!important` overrides everything
(and so is avoided). At equal specificity — the **later declared** one. Plus inheritance
(some properties inherit from the parent, like `color`).

The practice: keep **low specificity** (classes, not ids/`!important`), so styles are
predictable and overridable (methodologies BEM, utility CSS).

---

## Responsive design

One layout for different screens:
- **media queries** — different styles by width: `@media (max-width: 768px) { ... }`;
- **mobile-first** — styles for mobile first, then expand for bigger screens;
- **relative units** — `rem/em/%/vw/fr` instead of hard `px`, so it scales;
- flexible layouts (flex/grid) + `max-width`, responsive images (`srcset`).

---

## Accessibility (a11y)

So everyone can use the site, including screen readers and keyboards:
- **semantic elements** (a button is `<button>`, not `<div onclick>`);
- **`alt`** on images, **`<label>`** on fields;
- **ARIA** attributes where semantics fall short (role, state) — but **semantics first**,
  ARIA as a supplement;
- **keyboard navigation** (focus, tab order), a visible focus;
- sufficient text **contrast**.

---

## Interview phrasing

> "HTML is about meaning, CSS about looks, I keep them separate. Semantic tags give
> accessibility and SEO and work out of the box (button, label). The box model —
> content/padding/border/margin; I set `box-sizing: border-box` so width is predictable.
> Layout: flexbox for one-dimensional, grid for two-dimensional, often together. Style
> conflicts are resolved by the cascade by specificity (tag<class<id<inline, I avoid
> `!important`); I keep specificity low. Responsive — mobile-first, media queries, relative
> units, flexible grids. And accessibility: semantics, alt/label, ARIA as a supplement,
> keyboard, contrast."

---

See [06-html-css.tasks.md](./06-html-css.tasks.md) — tasks. Solutions in
[06-html-css.answers.md](./06-html-css.answers.md).
