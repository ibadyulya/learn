# HTML and CSS — solutions

> 🌐 Russian version: [06-html-css.answers.md](../ru/06-html-css.answers.md)

---

## Question 1
Semantic HTML carries **meaning**: screen readers and search engines understand the
structure, and the browser gives behavior out of the box. `<div onclick>` instead of
`<button>` loses: **keyboard accessibility** (a div isn't focusable and isn't pressed with
Enter/Space), the **role** for a screen reader ("button"), the focus state, semantics for
SEO. You'd have to recreate all that manually (tabindex, role, key handlers) — and it's
usually done poorly.

## Question 2
Layers from inside out: **content → padding → border → margin**. `box-sizing: border-box`
includes **padding and border in `width`/`height`**: set width to 200px and it stays 200px
including padding and border. It's set globally because the default `content-box` **adds**
padding/border on top, breaking width calculations — with border-box the layout is
predictable.

## Question 3
- **Flexbox** — **one-dimensional**: layout along **one** axis (a row **or** a column),
  alignment and space distribution along it. For toolbars, rows, centering.
- **Grid** — **two-dimensional**: rows **and** columns **at once**, grids/layouts.
The choice: laying out in one line → flex; need a real grid (rows and columns) → grid. Often
combined (grid for the page, flex inside cells).

## Question 4
The **more specific** rule wins. Ascending: **tag < class < id < inline**; `!important`
overrides everything. At equal specificity — the **later declared** one. `!important` is
avoided because it **breaks the cascade**: it can only be overridden by another `!important`,
starting an "importance race", and styles become unpredictable and unmaintainable. Keep low
specificity (classes).

## Question 5
**Mobile-first** — write base styles for the **mobile** screen, and add for bigger ones via
`@media (min-width: ...)`. It's simpler (the mobile layout is usually simpler) and more
robust for the most common case. **Relative units** (`rem/em/%/vw`) **scale** from a base
size/viewport/parent, adapting to the screen and user settings (enlarged font); hard `px`
are fixed and don't adapt, breaking accessibility and responsiveness.

## Question 6
a11y practices: **semantic elements**; **`alt`** on images; **`<label>`** on form fields;
**ARIA** (roles/states) where semantics fall short; **keyboard navigation** and a visible
focus; sufficient **contrast**. "Semantics first" — because native elements are **already**
accessible (focus, role, behavior), while ARIA **adds nothing to behavior**, it only
announces a role/state to assistive tech; incorrect ARIA is easy to make **worse** than
correct semantics. ARIA is a patch, not a replacement.

## Question 7
```css
.parent {
  display: flex;
  justify-content: center;  /* along the main axis (horizontal) */
  align-items: center;      /* along the cross axis (vertical) */
  min-height: 100vh;        /* so there's room to center vertically */
}
```
Three lines — and the block is centered in both axes.
