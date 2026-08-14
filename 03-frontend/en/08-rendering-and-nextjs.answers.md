# Rendering, SSR and Next.js — solutions

> 🌐 Russian version: [08-rendering-and-nextjs.answers.md](../ru/08-rendering-and-nextjs.answers.md)

---

## Question 1
A pure SPA returns **empty HTML + a large bundle**; the browser must download and execute JS
before showing anything. Hence: **poor SEO/preview** (a crawler/social bot sees an empty page
or must run JS), **long LCP/TTI** (a white screen while JS loads/executes), and a **request
waterfall** (data loads only after JS starts). Bad for the first paint and SEO.

## Question 2
- **CSR:** HTML in the **browser**, data at runtime; first paint/SEO — bad.
- **SSR:** HTML on the **server per request**, data fresh; paint/SEO — good, but TTFB and
  hydration.
- **SSG:** HTML at **build**, served from a CDN; data as of build time; paint/SEO —
  excellent, but not fresh.
- **ISR:** SSG + **background regeneration** by TTL/on demand: static speed + periodically
  fresh data.

## Question 3
1. The server generated and sent **ready HTML** — the user **sees** it, but it's "dead" (no
   handlers, no state).
2. The browser loads the **same** client JS bundle.
3. React **doesn't rebuild the DOM** but walks the **existing** server DOM and **attaches**
   event handlers and components' internal state.
4. After that the markup becomes **interactive**. That's hydration — "bringing the server
   HTML to life".

## Question 4
A **hydration mismatch** — a divergence between the HTML generated **on the server** and the
first render **on the client**. It arises from a **non-deterministic** render: `Date.now()`,
`Math.random()`, `window`/`localStorage`, the browser locale/timezone, conditional rendering
by client data. React expects a match; on a divergence it complains and may re-render/break
the markup. Avoid it: keep the **first render deterministic and identical**, and defer
anything client-dependent to `useEffect` (it runs after hydration).

## Question 5
SSR speeds up the **paint** (the user sees the server HTML immediately), but to make the page
**interactive**, the full JS bundle is **still** downloaded and executed on the client for
hydration — the JS work doesn't disappear, it only shifts. Addressed by: **RSC** (server
components don't ship their JS to the client at all) and **partial/selective hydration**
(hydrate only interactive islands, not the whole page) — so less JS ships to the client.

## Question 6
**RSC** render **only on the server**: their code **doesn't enter** the client bundle (0 JS),
and they can access the DB/files/secrets directly. A **client component** (`'use client'`) —
interactive (useState/useEffect/onClick), ships to the client and hydrates. The benefit: **a
smaller bundle** (static isn't shipped to the browser) and **no client waterfall** for data —
the server already hit the DB during render. Split: static/data → server, interactivity →
client.

## Question 7
Plain SSR waits until **all** the HTML (including slow data-dependent parts) is generated, and
only then sends it — the slowest request delays the whole page. **Streaming + Suspense** sends
HTML **in parts as they're ready**: the fast shell and content immediately, slow blocks (in
`<Suspense>`) streamed with a fallback. Result: earlier TTFB/LCP, the page isn't blocked by
the slowest chunk.

## Question 8
- **(a) Docs blog** — public content, rarely changing, SEO matters → **SSG** (served from a
  CDN, lightning-fast; ISR if edited periodically).
- **(b) Personalized recommendations feed with SEO** — need both SEO and **fresh/personal**
  data per request → **SSR** (or **RSC + streaming**: the server assembles personal HTML,
  streams it as ready).
- **(c) Dashboard editor behind a login** — no SEO needed, lots of interactivity and private
  data → **CSR**/client components (possibly with a small server shell), no spending on SSR
  for SEO that isn't there.
