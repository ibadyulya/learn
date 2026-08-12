# Authentication and authorization

> 🌐 Russian version: [04-auth.md](../ru/04-auth.md)

Who you are and what you're allowed to do. Reads as a story; behind the terms is a
simple split.

The through-line of the topic:

> **Authentication (authn) is "who you are", authorization (authz) is "what you're
> allowed". In a stateless API, session state lives either on the server (a session
> id + storage) or in the token itself (JWT). A JWT is self-contained and doesn't
> need a DB trip per request, but it can't be revoked instantly — hence an
> access+refresh pair and a short lifetime.**

---

## Authn vs authz — don't confuse them

- **Authentication** — establish identity (login+password, OAuth, biometrics). The
  answer to "who are you". Failure → **401**.
- **Authorization** — check the rights of an already-known user. The answer to "what
  you're allowed". Failure → **403**.
Authn first, then authz.

---

## Two ways to keep a session

**1. Server sessions (stateful).** On login the server creates a session record in
storage (Redis/DB) and gives the client a **session id** in a cookie. On each request
the server goes to storage by the id.
- ➕ can be **revoked instantly** (delete the record — the session is dead);
- ➖ **state on the server** → needs shared storage across multiple instances (see
  [scaling](../../06-system-design/en/05-scaling.md)), a storage trip per request.

**2. JWT (stateless).** On login the server issues a **signed token** that carries
data about the user. The client sends it on each request; the server **verifies the
signature** — no DB trip needed.
- ➕ **stateless**, easy to scale, no storage trip;
- ➖ **can't be revoked instantly** (the token is valid until expiry); its data may
  go stale.

---

## JWT on the inside

Three parts dot-separated: **`header.payload.signature`**, each base64url.
- **header** — the signing algorithm;
- **payload** — claims (user id, roles, `exp` — expiry);
- **signature** — a signature of header+payload with the server's secret.

Key: **the payload isn't encrypted but signed** — readable by anyone, but not
forgeable without the secret. So: don't put secrets in a JWT; trust the content only
after verifying the signature and `exp`.

---

## Access + refresh tokens

A compromise between "stateless" and "can't revoke":
- **access token** — short-lived (minutes), sent on each request; if stolen, it soon
  expires;
- **refresh token** — long-lived, stored securely, used **only** to get a new access.
  It **can be revoked** (stored/checked on the server).
This keeps stateless on the hot path plus the ability to revoke via refresh.

**Where to store on the client:** access — better in memory or an **httpOnly cookie**
(inaccessible to JS → protection from XSS theft), but a cookie needs **CSRF**
protection (SameSite, a CSRF token). `localStorage` is vulnerable to XSS. It's always
a trade-off between XSS and CSRF.

---

## OAuth2 / OIDC — logging in via others

**OAuth2** is a protocol of **delegated access**: an app gets limited access to a
user's resources at a provider (Google) without seeing the password. **OIDC** is an
authentication layer over OAuth2 ("log in with Google", issues an id token about the
identity). The user logs in at the provider, the app gets a token — it stores no
password.

---

## Authorization: RBAC vs ABAC

- **RBAC (role-based)** — rights are tied to roles (`admin`, `editor`), the user is
  granted roles. Simple and covers most cases.
- **ABAC (attribute-based)** — the decision uses attributes (owner of the resource?
  business hours? region?). More flexible, more complex.
In Nest authz usually lives in a **guard** (see [nestjs](./02-nestjs.md)).

---

## Password storage

Passwords are **never stored in plaintext or reversibly encrypted** — only **hashed**
with a slow adaptive algorithm (**bcrypt/scrypt/argon2**) with a **salt** (unique per
password, against rainbow tables). The slowness is deliberate protection against
brute force.

---

## Interview phrasing

> "Authn — who you are (failure 401), authz — what you're allowed (failure 403). A
> session is kept either server-side (a session id in a cookie + storage — revocable
> but stateful) or via a JWT (a signed self-contained token — stateless but not
> instantly revocable). A JWT is header.payload.signature, the payload is signed not
> encrypted, so I verify the signature and exp and put no secrets in it. I use a short
> access + a revocable refresh; access in an httpOnly cookie (XSS) with CSRF
> protection, or in memory. External login via OAuth2/OIDC. Authorization — RBAC
> (roles) or ABAC (attributes), in Nest — in a guard. Passwords — only a bcrypt/argon2
> hash with a salt."

---

See [04-auth.tasks.md](./04-auth.tasks.md) — tasks. Solutions in
[04-auth.answers.md](./04-auth.answers.md).
