# REST API design

> 🌐 Russian version: [03-rest-api-design.md](../ru/03-rest-api-design.md)

How to build predictable HTTP APIs. Reads as a story; it all grows from the idea
"resources + standard verbs".

The through-line of the topic:

> **REST is a style of designing APIs around resources: a URL is a noun (what), the
> HTTP method is a verb (the action on it), the status code is the result. The
> server is stateless: every request is self-contained. A good API is predictable —
> from the URL and method it's clear what will happen.**

---

## Resources and verbs

**A URL names a resource (a noun), the method sets the action.** No verbs in the
address:
```
GET    /orders          — list
GET    /orders/42       — one
POST   /orders          — create
PUT    /orders/42       — replace whole
PATCH  /orders/42       — partial update
DELETE /orders/42       — delete
```
Bad: `/getOrders`, `/createOrder` — the action is already in the method. Nesting for
relations: `GET /orders/42/items`.

---

## Status codes — the result

The code class tells you what happened:
- **2xx success:** `200 OK`, `201 Created` (a resource created, + a `Location`
  header), `204 No Content` (success with no body, e.g. DELETE).
- **3xx** — redirects.
- **4xx client error:** `400` (bad request), `401` (not authenticated), `403`
  (authenticated but not allowed), `404` (no resource), `409` (conflict, e.g. a
  duplicate), `422` (validation), `429` (rate limit).
- **5xx server error:** `500`, `503` (unavailable).

Distinguish `401` (who are you?) from `403` (you're not allowed) — a frequent
question.

---

## Method safety and idempotency

Two properties (interviewers love them):
- **Safe** — doesn't change state: `GET`, `HEAD`. Cacheable and freely repeatable.
- **Idempotent** — a repeat gives the same result: `GET`, `PUT`, `DELETE`. `PUT
  /orders/42` twice — one state. **`POST` isn't idempotent** — twice creates two
  resources.

Hence the practice: for unsafe **POST** with a duplicate risk (payment) introduce an
**Idempotency-Key** — the client sends a unique key, the server dedups by it (a
direct link to idempotency from
[reliability-patterns](../../06-system-design/en/03-reliability-patterns.md)).

---

## Statelessness — the server doesn't remember the client

A REST server **stores no session state between requests**: each request carries all
it needs (token, params). Why: any instance behind the balancer handles any request →
horizontal scaling (see [scaling](../../06-system-design/en/05-scaling.md)). State
lives in the DB/cache/token, not in the process's memory.

---

## Versioning, pagination, filters

- **Version** — to not break clients on changes: in the path (`/v1/orders`) or a
  header. The path is simpler and more visible.
- **Pagination** — don't return 100k records at once: offset/limit
  (`?page=2&limit=20`) or cursor-based (more stable on live data).
- **Filters/sorting** — via query: `GET /orders?status=paid&sort=-createdAt`.

---

## A unified error shape

Errors with a predictable structure, not ad hoc:
```json
{ "error": { "code": "ORDER_NOT_FOUND", "message": "Order 42 not found", "details": [] } }
```
The client parses it the same for all endpoints. The status code is in HTTP, the
machine `code` is in the body.

---

## Interview phrasing

> "REST is designing APIs around resources: a URL is a noun, the HTTP method is the
> action, the status code is the result (2xx success, 4xx client, 5xx server; 401
> 'who are you' vs 403 'not allowed'). GET is safe and idempotent, PUT/DELETE are
> idempotent, POST isn't, so for critical POSTs I add an Idempotency-Key. The server
> is stateless — state in the token/DB, which enables horizontal scaling. Plus
> versioning to not break clients, pagination (offset or cursor), filters via query,
> and a unified error shape with a machine code."

---

See [03-rest-api-design.tasks.md](./03-rest-api-design.tasks.md) — tasks. Solutions
in [03-rest-api-design.answers.md](./03-rest-api-design.answers.md).
