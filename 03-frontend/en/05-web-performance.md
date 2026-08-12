# Web performance

> 🌐 Russian version: [05-web-performance.md](../ru/05-web-performance.md)

How to make a page load fast and stay responsive. Reads as a story; builds on
[browser/rendering](./04-browser-dom-rendering.md) and
[cache/CDN](../../06-system-design/en/07-caching-cdn.md).

The through-line of the topic:

> **Performance — how fast the user **sees** the page and can **use** it. Two fronts:
> loading (fewer bytes and later ones that aren't needed immediately) and runtime (don't
> block the single thread). Measure objectively — by Core Web Vitals, not "by eye".**

---

## What and why we measure: Core Web Vitals

Google formalized three real-experience metrics:
- **LCP (Largest Contentful Paint)** — when the **main content** was painted (loading).
  Target ~< 2.5s.
- **INP (Interaction to Next Paint)** — how fast the page **responds** to actions
  (interactivity; formerly FID). Target < 200ms.
- **CLS (Cumulative Layout Shift)** — how much the layout **jumps** during loading (visual
  stability). Target < 0.1.
The point: show fast, respond fast, don't jerk the layout.

---

## Loading optimization — less and later

- **Bundling + minification + compression** (gzip/brotli) — fewer bytes over the network.
- **Code splitting** — cut the bundle into parts, load by route/on demand, not the whole
  frontend at once.
- **Lazy loading** — load as needed: routes, components, below-the-fold images
  (`loading="lazy"`).
- **Tree-shaking** — drop unused code (works thanks to ESM's staticness, see
  [modules](../../01-js-typescript/en/02-modules-cjs-esm.md)).
- **Critical CSS** — inline styles for the first screen, the rest asynchronously, so as not
  to block rendering.
- **Cache/CDN** — static closer to the user with versioned names (see
  [cache/CDN](../../06-system-design/en/07-caching-cdn.md)).

---

## Asset optimization

Often **images are the heaviest resource**:
- modern formats (**WebP/AVIF**), compression;
- **responsive** images (`srcset` — by screen size), don't ship 4K to a phone;
- **lazy-load** below-the-fold images;
- set sizes (width/height) to avoid a layout jump (**CLS**).
Fonts: `font-display: swap` (don't hide text while loading), subsetting.

---

## Runtime performance

- **Don't block the thread** — long tasks freeze the frame (see
  [browser/rendering](./04-browser-dom-rendering.md) and
  [event loop](../../01-js-typescript/en/01-event-loop.md)); split, move to web workers.
- **List virtualization** — render only the visible rows out of thousands.
- **Debounce/throttle** — limit frequent events (input, scroll, resize).
- Animate `transform/opacity`, minimize reflow.
- Fewer excess re-renders (see [React](./01-react.md)).

---

## How to measure

- **Lab** (synthetic): **Lighthouse**, DevTools — reproducible, for development.
- **RUM** (real user monitoring): metrics from real users — the truth about production.
Rule: **measure → optimize the bottleneck → re-measure**, don't guess.

---

## Interview phrasing

> "I measure performance by Core Web Vitals: LCP (when the main content is visible), INP
> (responsiveness to actions), CLS (layout stability). I optimize loading — bundling,
> minification, compression, code splitting and lazy loading (load on demand),
> tree-shaking, critical CSS, cache/CDN. Assets — modern formats and responsive images with
> set sizes (against CLS), fonts with font-display swap. At runtime I don't block the
> thread (split/worker), virtualize lists, debounce/throttle, animate transform. And I
> always measure (Lighthouse + RUM) rather than optimize blindly."

---

See [05-web-performance.tasks.md](./05-web-performance.tasks.md) — tasks. Solutions in
[05-web-performance.answers.md](./05-web-performance.answers.md).
