# Validation and error handling

> 🌐 Russian version: [05-validation-error-handling.md](../ru/05-validation-error-handling.md)

How to keep bad data out of the system and react to errors uniformly. Reads as a
story.

The through-line of the topic:

> **Rule number one: don't trust input — validate at the boundary, before business
> logic. And handle errors centrally and uniformly, separating expected ones (bad
> input, a violated business rule → 4xx) from unexpected ones (bugs, crashes →
> 5xx), and never leak internals outward.**

---

## Validation at the boundary

The client may send anything (accidentally or maliciously). Check **at the entry**,
in one place, not scattering `if`s through the code:
```ts
// DTO + declarative rules (class-validator in Nest)
class CreateUserDto {
  @IsEmail()               email!: string;
  @MinLength(8)            password!: string;
  @IsInt() @Min(18)        age!: number;
}
```
In Nest the **`ValidationPipe`** does this at the pipes layer (see
[nestjs](./02-nestjs.md)): if it fails — automatically `400/422`, the controller is
never reached. Outside Nest — Zod / Joi. The point is the same: **business logic
receives already-checked data** and doesn't worry about garbage.

Validate **type, bounds, and invariants** (email — format, age ≥ 18), and don't
trust "hidden" fields (role, price shouldn't come from the client's body).

---

## Two kinds of errors

A key distinction for the strategy:
- **Expected (operational)** — part of normal operation: failed validation, no
  resource, a conflict, no rights. We **anticipate** them and turn them into a
  meaningful **4xx** with a clear message.
- **Unexpected (programmer errors)** — bugs, an unavailable dependency, an unhandled
  exception. Outward — an anonymized **5xx** "something went wrong", inward — a
  detailed log; fix with code.

Don't mix them: "email taken" is expected (409), "undefined is not a function" is
unexpected (500).

---

## Centralized handling

Don't wrap every handler in a `try/catch` with a hand-built response — set up **one**
handler:
- in Nest — an **exception filter** catches thrown exceptions and shapes the HTTP
  response; business code just **throws** typed errors (`NotFoundException`,
  `ConflictException`), the filter maps them to a status + a unified shape;
- in Express — an **error middleware** `(err, req, res, next)` at the end of the
  chain.

```ts
// business code: just throw
if (!user) throw new NotFoundException('User not found');
// filter/middleware: a unified JSON { error: { code, message } } + the right status
```
The benefit: a **unified shape** for errors (see [REST](./03-rest-api-design.md)), no
duplicated handling, clean logic.

---

## Don't leak internals

Outward — only the safe: an error code, a human-readable message, for validation — a
list of fields. You **must not** give the client a stack trace, SQL, paths, or DB
exception contents — that's a leak and a hint to an attacker. Details → to the log
(with context: request id, user id), outward → an anonymized response.

---

## Fail fast and logging

- **Fail fast** — crash on a bad state early (check input/invariants at the start),
  rather than dragging garbage deep where it surfaces as a cryptic error.
- **Log with context** — structured (JSON), with a correlation id to tie a record to
  a request; levels (error/warn/info). (Link to
  [observability](../../06-system-design/en/11-observability.md).)

---

## Interview phrasing

> "I don't trust input — I validate at the boundary declaratively (DTO +
> class-validator/Zod), in Nest that's the ValidationPipe, bad data never reaches the
> controller. I split errors into expected (validation, not found, conflict, rights →
> a meaningful 4xx) and unexpected (bugs → an anonymized 5xx + a detailed log). I
> handle them centrally — an exception filter in Nest or error middleware in Express:
> business code throws typed errors, one layer maps them to a unified response shape.
> I don't leak stack traces/SQL outward — details only to the log with a request id.
> Plus fail fast and structured logging."

---

See [05-validation-error-handling.tasks.md](./05-validation-error-handling.tasks.md)
— tasks. Solutions in [05-validation-error-handling.answers.md](./05-validation-error-handling.answers.md).
