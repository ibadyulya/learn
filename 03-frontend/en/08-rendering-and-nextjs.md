# Rendering, SSR and Next.js

> 🌐 Russian version: [08-rendering-and-nextjs.md](../ru/08-rendering-and-nextjs.md)

Where and when the HTML is produced and when the page becomes interactive. Reads as a
story; continues [React](./01-react.md), builds on
[web performance](./05-web-performance.md) and
[browser/rendering](./04-browser-dom-rendering.md).

The through-line of the topic:

> **The key question — WHERE the HTML is rendered (in the browser or on the server) and
> WHEN the page becomes interactive. CSR does everything in the browser, SSR generates
> HTML on the server per request, SSG/ISR ahead of time. **Hydration** "brings the
> server HTML to life" on the client. Next.js gives all the strategies plus server
> components (RSC) and streaming.**

---

## The problem with pure CSR (a plain SPA)

A classic React SPA: the server returns a nearly **empty** `<div id="root">` + a large JS
bundle. Then the browser: download JS → execute → build the DOM → (then) fetch data. The
downsides at senior scale:
- **poor SEO and preview sharing** — a crawler/social bot sees an empty page (or must
  execute JS);
- **long LCP/TTI** — the user stares at a white screen until the JS is downloaded and
  executed (see [Core Web Vitals](./05-web-performance.md));
- **request waterfall** — data starts loading only **after** the JS starts.

Hence — move rendering (partly) to the server.

---

## Rendering strategies

- **CSR (Client-Side Rendering)** — rendering in the browser. Simple, fully interactive
  after load, but a poor first paint and SEO. Fine behind a login (dashboards) where SEO
  isn't needed.
- **SSR (Server-Side Rendering)** — HTML is generated on the server **per request** and
  returned ready. Fast first paint (content visible immediately), good SEO, fresh/personal
  data — **at the cost of** server load (TTFB) and the need for **hydration**.
- **SSG (Static Site Generation)** — HTML is generated **at build time**, served as static
  from a CDN. Lightning-fast and cheap, but data "freezes" at build time. Blogs, docs,
  landing pages.
- **ISR (Incremental Static Regeneration)** — SSG + **background regeneration** of pages by
  TTL or on demand: static speed + periodically fresh data. Product catalogs.

| | CSR | SSR | SSG | ISR |
|---|---|---|---|---|
| Where HTML renders | browser | server (per request) | build | build + background |
| Data | at runtime | per request (fresh) | at build | periodically fresh |
| First paint / SEO | bad | good | excellent | excellent |
| Server load | none | per request | none (CDN) | low |

---

## Hydration — the heart of the topic

SSR returned **ready HTML** — the user **sees** it, but it's "dead": no event handlers,
buttons don't click, no state. Then the **same** JS loads on the client, React walks the
**already-existing** server DOM and **attaches** handlers and internal state to it — brings
the markup to life. That's **hydration**.

Three things that get asked:
- **"Uncanny valley" / lost interactivity** — the window when content is **visible but not
  clickable** (JS is still loading/hydrating). Clicks during it are lost → poor **INP**.
- **Hydration mismatch** — if the HTML **from the server** and the first render **on the
  client** diverge (you used `Date.now()`, `window`, `Math.random()`, the browser locale),
  React complains and may re-render/break. Rule: **the server and client first render must
  match**; defer non-deterministic things to `useEffect`.
- **Cost** — SSR speeds up the **paint**, but all the JS is **still** downloaded and
  executed for hydration. Hence the ideas of **partial/selective hydration** and RSC, to
  ship less JS to the client.

---

## React Server Components (RSC)

Components that render **only on the server**: their JS **doesn't ship** to the client
(zero bundle contribution), and they can access the DB/files/secrets directly. **Client
Components** (the `'use client'` directive) — interactive, ship to the client and hydrate.

The split idea:
```
Server Component (default in the App Router): static content, data access, 0 JS to the client
  └─ Client Component ('use client'): interactivity (onClick, useState, useEffect)
```
The benefit: **a smaller bundle** (static isn't shipped to the client), no client waterfall
for data (the server hit the DB during render). RSC isn't a replacement for SSR/hydration
but a different cut: **which components are needed on the client at all**.

---

## Streaming SSR + Suspense

Instead of "generate all HTML → send in one chunk", the server sends HTML **in parts as
they're ready**: fast parts immediately, slow ones (waiting for data) streamed, wrapped in
`<Suspense fallback={...}>`. The user sees the shell and fast content sooner, the page isn't
blocked by the slowest request → better TTFB and LCP.

---

## Next.js — a framework over React

Gives all the strategies out of the box:
- **App Router** (modern) — **RSC by default**, nested layouts, `<Suspense>`/streaming,
  **Server Actions** (mutations straight from the server without a manual API); the strategy
  is chosen at the data-fetch level (cache/revalidate).
- **Pages Router** (old) — `getServerSideProps` (SSR per request), `getStaticProps`
  (SSG/ISR via `revalidate`), `getStaticPaths`.
- plus: file-based routing, image/font optimization, automatic code splitting by route,
  edge/node runtime.

Where things render: **server components** — on the server (0 JS to the client), **client
components** — rendered to HTML on the server and then **hydrated** in the browser.

---

## How to choose a strategy

- public, rarely-changing content (blog, docs, landing) → **SSG** (+ **ISR** if it updates
  periodically);
- need SEO **and** fresh/personal data (feed, product) → **SSR** or **RSC + streaming**;
- behind a login, SEO doesn't matter, lots of interactivity (dashboard, editor) → **CSR** /
  client components;
- in reality you **mix** at the page and even component level — Next allows it.

Rule: choose by **SEO, data freshness, and interactivity**, not "SSR is always better".

---

## Interview phrasing

> "The key question is where the HTML renders and when the page is interactive. CSR — all in
> the browser (empty HTML + JS): simple, but poor LCP/SEO. SSR — HTML on the server per
> request: fast paint and SEO at the cost of TTFB and hydration. SSG — HTML at build,
> served from a CDN: lightning-fast, but data is static; ISR adds background regeneration.
> Hydration — the client loads the same JS and "attaches" handlers to the server DOM; hence
> the window of lost interactivity (INP), hydration mismatch on non-deterministic render, and
> the fact the bundle still ships. RSC render only on the server and don't ship JS to the
> client, client components (`'use client'`) — interactivity; streaming + Suspense sends HTML
> in chunks. Next.js gives all this: the App Router with RSC by default and Server Actions,
> the Pages Router with getServerSideProps/getStaticProps. I choose the strategy by SEO,
> freshness, and interactivity, often mixing."

---

See [08-rendering-and-nextjs.tasks.md](./08-rendering-and-nextjs.tasks.md) — tasks.
Solutions in [08-rendering-and-nextjs.answers.md](./08-rendering-and-nextjs.answers.md).
