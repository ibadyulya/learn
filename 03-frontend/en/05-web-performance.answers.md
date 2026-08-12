# Web performance — solutions

> 🌐 Russian version: [05-web-performance.answers.md](../ru/05-web-performance.answers.md)

---

## Question 1
- **LCP (Largest Contentful Paint)** — when the **main content** was painted (loading speed
  of what the user sees).
- **INP (Interaction to Next Paint)** — how fast the page **responds** to actions
  (interactivity/responsiveness).
- **CLS (Cumulative Layout Shift)** — how much the **layout jumps** during loading (visual
  stability).

## Question 2
- **Code splitting** — cut one big bundle into **parts** (by route/feature).
- **Lazy loading** — load those parts **on demand**, not all at once.
They solve the problem of a **heavy initial bundle**: the user needs the current screen's
code, not the whole app's. Faster first paint and interactivity (better LCP/INP), less
traffic.

## Question 3
**Tree-shaking** — removing **unused** code from the build. It works because **ESM is
static**: `import`/`export` are parsed **before execution**, the bundler sees the dependency
graph in advance and knows exactly what isn't imported → drops it. **CommonJS** is dynamic
(`require` is a runtime call, can be conditional/computed), so you can't reliably determine
the unused before running.

## Question 4
Formats **WebP/AVIF** + compression (less weight); **responsive** images via `srcset` (by
screen size/density — don't ship 4K to a phone); **lazy-load** for below-the-fold images;
font **subsetting**. Set **width/height** (or `aspect-ratio`) matters for **CLS**: the
browser reserves space in advance and the content **doesn't jump** when the image loads;
without sizes the layout shifts down — a bad CLS.

## Question 5
JS and rendering share **one thread**: heavy synchronous JS blocks painting and input
handling → bad INP, "freezes". Techniques: **split** long tasks (yield), move to a **web
worker**, **virtualize** long lists (render only the visible), **debounce/throttle** frequent
events, minimize re-renders and reflow, animate `transform/opacity`.

## Question 6
- **Debounce** — run the action **once after a pause**: wait until events stop for N ms
  (search-as-you-type — request after typing stops).
- **Throttle** — run **at most once per N ms** during a continuous stream (scroll, resize —
  update position at most every 100ms).
Debounce — "wait for silence", throttle — "cap the frequency".

## Question 7
- **Lab (Lighthouse/DevTools)** — synthetic under controlled conditions: **reproducible**,
  handy in development/CI, but doesn't reflect real users' devices/networks.
- **RUM (Real User Monitoring)** — metrics collected from **real users** in production: the
  truth about the actual experience (different devices, networks, geos).
You need both: lab — to fix and not regress during development, RUM — to know the real
picture.
