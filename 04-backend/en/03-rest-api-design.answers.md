# REST API design — solutions

> 🌐 Russian version: [03-rest-api-design.answers.md](../ru/03-rest-api-design.answers.md)

---

## Question 1
`/getUser` puts a **verb in the URL**, duplicating what the HTTP method already
expresses. In REST a URL is a noun-resource, the action is the method: **`GET
/users/5`**. Predictable and uniform: the same resource `/users/5` for reading (GET),
updating (PATCH), deleting (DELETE).

## Question 2
- **`401 Unauthorized`** — "not authenticated": no/expired token, the system doesn't
  know who you are. Scenario: a request with no `Authorization` header.
- **`403 Forbidden`** — "authenticated but not allowed": who you are is known, you
  lack the rights. Scenario: a regular user reaching into `/admin`.

## Question 3
- **Safe** — doesn't change state. **Idempotent** — a repeat gives the same result.

| Method | Safe | Idempotent |
|---|:--:|:--:|
| GET | ✅ | ✅ |
| POST | ❌ | ❌ |
| PUT | ❌ | ✅ |
| PATCH | ❌ | ⚠️ usually not (depends) |
| DELETE | ❌ | ✅ |

## Question 4
`POST /orders` creates a **new** resource every time → a repeat (network retry, double
click) creates two orders/two charges. The API-level solution — an **Idempotency-Key**:
the client generates a unique key and sends it in a header; the server remembers the
result by the key and, on a repeat with the same key, returns the **same** response
without performing the operation twice. (The same as idempotency in
reliability-patterns.)

## Question 5
Stateless = the server **stores no session state between requests** in memory; each
request is self-contained (carries the token and all params). Then **any** instance
behind the balancer can handle **any** request — you can add instances horizontally
without "sticky" sessions. State is moved to the DB/cache/token.

## Question 6
- (a) resource created → **`201 Created`** (+ `Location`);
- (b) failed validation → **`422 Unprocessable Entity`** (or `400`);
- (c) deleted, no body → **`204 No Content`**;
- (d) duplicate email → **`409 Conflict`**.

## Question 7
- **Offset** (`?page=2&limit=20` / `OFFSET/LIMIT`) — simple, but on **live data** it
  glitches: inserts/deletes shift pages (skips/duplicates), and large offsets are slow
  (the DB pages through everything up to the target).
- **Cursor** — "give 20 records after this id/time": stable under changes and fast (an
  index lookup from the cursor), but you can't jump to an arbitrary page.
Cursor is better for feeds/infinite scroll and large sets.
