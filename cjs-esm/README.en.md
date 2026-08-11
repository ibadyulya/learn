# CommonJS vs ES Modules

Two module systems in JavaScript: what they are, where they came from, how they
differ and why it comes up in interviews (including in connection with the event
loop). Reads as a story, not a cheat sheet — the goal is to **understand**, not
memorize.

> 🌐 Russian version: [README.md](./README.md)

In short, the whole topic boils down to one difference, from which everything
else grows: **CommonJS figures out dependencies at runtime and synchronously,
while ES Modules do it ahead of time and statically.** Keep that sentence in
mind — everything below keeps coming back to it.

---

## Where modules came from in the first place

Picture early JavaScript. It lived only in the browser, and all the code on a
page dumped into one shared namespace. Include two `<script>`s — and if both
have a `user` variable, they silently overwrite each other. No isolation. While
scripts were small, people tolerated it.

Then **Node.js** appeared — JavaScript on the server. And on the server you
can't live like that: a server app is hundreds of files, and if everything
piles into one heap, it's hell. You need a way to say: "this file is a separate
box. Inside it, variables don't leak out, and only what I allow sticks out."

That's what a **module** is: a file as an isolated box with an "output window"
(what we hand out) and an "input window" (what we take from others).

The key historical fact from which all the confusion grows: **Node needed
modules before the language itself came up with a standard.** Node didn't wait
and made its own format — **CommonJS (CJS)**. A few years later the language
itself got an official standard — **ES Modules (ESM)**. Now the two live side by
side. Hence all the duality we deal with.

---

## CommonJS — "execute as you go"

Node made modules the simplest possible way — **through ordinary functions and
objects**, without any new language syntax.

To hand something out — put it in a special `module.exports` object:

```js
// math.js
function add(a, b) { return a + b; }
module.exports = { add };
```

To take someone else's — call the `require` function, it returns that object:

```js
// app.js
const math = require('./math.js');
math.add(2, 3);
```

Now grasp what actually happens here, because it's the root of everything else.
`require('./math.js')` is **literally a function call right on this line**. When
the engine reaches it, it stops, goes to disk, reads `math.js`, executes it fully
top to bottom, grabs whatever is left in `module.exports`, and returns it to you.
And only then does execution continue.

That is, Node learns about your dependency **at the moment it reaches
`require`** — not a second earlier. This is called "dynamic loading":
dependencies are resolved on the fly, during execution.

And hence a practical consequence. Since `require` is just code that runs when
you reach it, you can put it **anywhere**:

```js
if (process.env.DEBUG) {
  const logger = require('./logger');   // load it only if needed
}
```

Perfectly legal. It's an ordinary function call — what difference does it make
where it stands.

Keep this picture in mind: **you walk through the code, hit a `require` — "ah, I
need to load this", loaded it, moved on.** You learn about dependencies *along
the way*.

---

## ES Modules — "sort it out first, then execute"

When modules were added to the language itself, they did it differently — with
new keywords `import` / `export`:

```js
// math.js
export function add(a, b) { return a + b; }

// app.js
import { add } from './math.js';
```

Looks almost the same, but the mechanics are **the opposite**, and that's the
most important thing in the whole topic.

`import` is **not a function call**. It's a declaration the engine reads **before
it executes a single line of code**. First it scans the file, finds all the
`import`s and `export`s, follows them into other files, finds their imports
there too — and so builds a **complete dependency map of the whole application
before running it**. And only when the map is ready does it start executing.

Formally that's three phases, but the essence is in the first:
1. **construct / parse** — read the files, find all import/export, walk the whole
   dependency graph;
2. **instantiate / link** — allocate memory cells for exports and link imports to
   them (by reference — remember this, it'll matter below);
3. **evaluate** — run the modules' code.

That's why you can't stuff `import` into an `if`:

```js
if (x) {
  import foo from './foo.js';   // ← syntax error
}
```

The engine needs to build the dependency map **before running**, i.e. before it
even knows what `x` equals. It physically cannot decide whether this dependency
exists. So `import` must stand statically, at the top level of the file.

Picture: **first you fully draw the map ("who depends on whom"), and only then
drive off to execute.** You learn about dependencies *ahead of time*.

> If you need exactly conditional loading — there's a separate **dynamic
> `import()`** for that: it looks like a function, works asynchronously and
> returns a promise. It can go inside an `if`. But that's a deliberate
> exception, not the ordinary `import`.

This difference — **"along the way" (CJS) versus "ahead of time" (ESM)** — is
what everything else grows from: why `import` is only at the top, why ESM can do
tree-shaking (the bundler sees the whole map in advance and drops what nobody
imports), why loading is asynchronous, and everything below.

---

## A snapshot versus a live wire

A subtlety interviewers love, and it follows directly from "along the way vs
ahead of time".

In CJS `require` returns the `module.exports` object. When you write
`const { count } = require('./counter.js')`, you grab **what's in the box right
now**. It's like photographing a shelf in a store: you have a snapshot at the
moment of the shot. The clerk later rearranges the goods — your photo still shows
the old state.

```js
// counter.js (CJS)
let count = 0;
setInterval(() => count++, 1000);   // the module changes count within itself
module.exports = { count };

// app.js
const { count } = require('./counter.js');
// no matter how long you wait — here it's always 0.
// you took a snapshot of the value 0, not a link to the variable.
```

In ESM — the opposite. An import is **not a snapshot of a value, but a live wire**
to that very variable in the other file:

```js
// counter.mjs
export let count = 0;
setInterval(() => count++, 1000);

// app.mjs
import { count } from './counter.mjs';
// wait a couple of seconds, read count — you'll see 3, 4, 5...
```

Why? Again from the same root. CJS runs the module as an ordinary function and
hands you the result object — an ordinary value copy. ESM, back at the map-building
stage (the link phase), **linked your `import` directly to the memory cell of the
other module's `export`**. It didn't copy the value — it ran a reference. Hence
"live".

One caveat: the wire is **one-way, read-only**. You see the other side's changes,
but you can't assign to the imported variable yourself (`count = 100`) — error.
Only the file that exported it changes it.

---

## Who can import whom

Since there are two formats, the question arises: can they call each other. The
answer is asymmetric — and, if you understood the previous part, now obvious.

**ESM can pull in CJS** — no problem:

```js
import fs from 'fs';   // fs is an old CJS module, and it all works
```

Why is it allowed? ESM loads asynchronously, it has plenty of time; waiting for
a synchronous CJS module to run is no problem for it. It takes whatever ended up
in `module.exports` and substitutes it as `default`.

**CJS cannot synchronously pull in ESM** — this is historically forbidden:

```js
const stuff = require('./modern.mjs');   // ← nope
```

Why? `require` by its nature is **synchronous** — "stand still, wait, return
right now". And ESM loads **asynchronously** — it needs time to build the map,
load something. Synchronous `require` can't wait for the asynchronous. A clash of
temperaments.

The workaround is that same dynamic `import()`, it's asynchronous and returns a
promise:

```js
// inside an async function in a CJS file:
const stuff = await import('./modern.mjs');   // this works
```

Rule of thumb: **upward — allowed, downward synchronously — no.** The new (ESM)
calmly pulls in the old (CJS). The old (CJS) can't reach the new synchronously —
only through the asynchronous `import()`.

---

## How Node figures out the format

Node decides by signals, in priority order:
1. Extension: `.mjs` — always ESM, `.cjs` — always CJS.
2. For a plain `.js` — the `"type"` field in the nearest `package.json`:
   - `"type": "module"` → `.js` is ESM;
   - `"type": "commonjs"` or no field → `.js` is CJS.

```json
// package.json
{ "type": "module" }   // now all .js in the project are ESM
```

## Little things that differ

CJS has "magic" variables ESM doesn't, and vice versa:

- `__dirname` and `__filename` (path to the file and folder) — **CJS only**. In
  ESM you assemble them by hand from `import.meta.url`:

```js
import { fileURLToPath } from 'node:url';
import { dirname } from 'node:path';
const __dirname = dirname(fileURLToPath(import.meta.url));
```

- **top-level await** — the ability to write `await` right at the top level of a
  module, without wrapping it in an `async function` — exists **only in ESM**. In
  CJS you can't. It's not just convenience: it's precisely why timer timing
  shifts (next section).

---

## And now — the link to the event loop (the reason this is in the feedback)

Recall the main race from the event loop topic: at the top level `setTimeout(fn, 0)`
versus `setImmediate` — who's first is unpredictable, because `setTimeout(0)`
secretly turns into `setTimeout(1ms)`, and it all comes down to whether that
millisecond has passed.

Here's the thing. **The event loop itself is exactly the same for CJS and ESM** —
same phases, same microtasks, same clamp to 1ms. What changes is only **the
moment when the timers even manage to register**.

- In **CJS** a module is one solid synchronous run top to bottom. We read the
  whole file as a single chunk, registering `setTimeout` and `setImmediate` along
  the way, and only then does the loop spin up. Fast, in one go.
- In **ESM** the start is stretched out: asynchronous loading, map building, and
  if there's `top-level await` — execution is even torn into pieces (the code
  after `await` moves to a microtask).

Bottom line: since in ESM the start is slower and choppy, by the time the loop
reaches the timers phase that 1ms has more often **managed** to pass → the timer
is already "ripe" → `setTimeout` wins more often. In CJS the start is instant,
the millisecond more often **hasn't** managed → `setImmediate` wins more often.

But it's still **not a guarantee, just a probability shift.** A race stays a
race. The only place where the order is rock-solid deterministic is inside an
I/O callback (there `setImmediate` is always first, because the `check` phase
follows `poll` immediately), and that rule doesn't depend on the module format.

---

## Interview phrasing

> "CommonJS is the old Node format: `require`/`module.exports`. Loading is
> synchronous and dynamic — `require` is an ordinary function call, runs where
> it stands, so you can put it even in an `if`; what's handed out is a snapshot
> of the value. ES Modules is the language standard: `import`/`export`. Loading
> is asynchronous and static — the engine builds the dependency map before
> execution, so `import` is top-level only, but tree-shaking is possible; an
> import is a live read-only reference to another module's export. ESM can pull
> in CJS, the reverse synchronously can't — only through dynamic `import()`. The
> event loop is common to both, but top-level await and ESM's asynchronous start
> shift when timers register, so the `setTimeout(0)` vs `setImmediate` race in
> ESM falls toward the timer more often — though there are still no guarantees."

---

See [tasks.en.md](./tasks.en.md) — comprehension questions. Solutions — in [answers.en.md](./answers.en.md).
