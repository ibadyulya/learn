# Validation and error handling — solutions

> 🌐 Russian version: [05-validation-error-handling.answers.md](../ru/05-validation-error-handling.answers.md)

---

## Question 1
Validation at the boundary guarantees that **business logic receives already-checked
data** — it isn't cluttered with checks and doesn't worry about garbage (high
cohesion). It's a single point, not scattered `if`s. In Nest — the **`ValidationPipe`**
at the pipes layer: it checks the DTO by decorators (class-validator) before the
controller; on failure → `400/422` automatically.

## Question 2
- **Expected (operational)** — a normal part of operation, anticipated → a meaningful
  4xx: "email taken" (409), "no rights" (403), "DB timeout" (often 503/retryable) —
  arguably operational (an external dependency).
- **Unexpected (programmer)** — bugs: "Cannot read property of undefined" → 500 + a
  log, fix with code.
The line: foreseeable and meaningful to the client vs a code defect.

## Question 3
Centralized = **one** handler shapes the response in a unified format; handlers don't
duplicate `try/catch`, business code just **throws** typed errors. In **Nest** — an
**exception filter** catches exceptions and maps them to an HTTP response; in
**Express** — **error middleware** `(err, req, res, next)` at the end of the chain.
Result: a unified shape, no duplication, clean logic.

## Question 4
You must not return a **stack trace, SQL queries, file paths, DB exception text, or
internal codes** — that's information leakage and a hint to an attacker (structure,
versions, table names). To the client — only an error code + a human-readable message
(+ fields on validation). Details → to the **log** with context (request id, user id).

## Question 5
**Fail fast** — detect an incorrect state and crash **as early as possible** (check
input/invariants at the start), rather than letting garbage through deep. Otherwise
bad data surfaces somewhere far away as a cryptic error that's hard to diagnose. An
early explicit refusal = simpler debugging and less damage.

## Question 6
`role` from the client's body is **data under the attacker's control**; trusting it =
letting anyone make themselves admin (**privilege escalation**). The role is assigned
by the **server** by its own logic (default `user`, promotion via a separate
authorized process). Privilege/price/owner fields are never taken from client input —
the server sets them.

## Question 7
Status **`400`** or **`422 Unprocessable Entity`**. The body — a unified shape with a
list of problem fields:
```json
{ "error": { "code": "VALIDATION_FAILED",
  "message": "Validation failed",
  "details": [{ "field": "email", "message": "must be an email" },
              { "field": "age", "message": "must be >= 18" }] } }
```
So the client can show errors at specific form fields.
