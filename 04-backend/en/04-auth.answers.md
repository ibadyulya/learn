# Authentication and authorization — solutions

> 🌐 Russian version: [04-auth.answers.md](../ru/04-auth.answers.md)

---

## Question 1
- **Authentication** — establish **who** you are (login/password, token). Failure →
  **401**.
- **Authorization** — check **what** you're allowed (rights/roles). Failure → **403**.
Authn first (identity), then authz (access).

## Question 2
- **Server sessions:** ➕ can be **revoked instantly** (delete the record); ➖
  **stateful** — needs shared storage and a trip to it per request.
- **JWT:** ➕ **stateless** — the signature is verified without the DB, easy to scale;
  ➖ **can't be revoked instantly**, valid until `exp`, data may go stale.

## Question 3
`header.payload.signature`. The payload is **signed, not encrypted** — anyone can
read it (base64url), but it can't be forged without the secret. In practice: don't put
secret data in a JWT; trust the content only **after verifying the signature and
`exp`**.

## Question 4
One long-lived token = either insecure (stolen → valid long, can't revoke) or
inconvenient (short → frequent re-login). The pair solves the trade-off: **access** is
short-lived on the hot path (stolen → soon expires), **refresh** is long but
**revocable** (stored/checked on the server) and used only to renew access. So it's
mostly stateless yet revocation is possible.

## Question 5
- **`localStorage`** — accessible to JS → vulnerable to **XSS** (an attacker's script
  steals the token).
- **httpOnly cookie** — inaccessible to JS (protection from XSS theft), but a cookie
  is sent automatically → vulnerable to **CSRF** (mitigated by `SameSite`, a CSRF
  token).
The choice is an XSS↔CSRF trade-off; an httpOnly cookie + SameSite is usually safer.

## Question 6
- **OAuth2** — a protocol of **delegated access**: an app gets limited access to a
  user's data at a provider, **without seeing the password** (via access tokens and
  scopes).
- **OIDC** — an **authentication** layer over OAuth2: besides access it issues an
  **id token** about the user's identity.
"Log in with Google": the user logs in at Google, the app gets token(s) and learns who
it is, without storing or seeing the password.

## Question 7
Passwords are **only hashed** (not stored plaintext, not reversibly encrypted). A fast
SHA-256 is bad: it can be brute-forced at **billions of hashes per second** (and
rainbow tables). You need a **slow adaptive** algorithm — **bcrypt/scrypt/argon2** —
where the cost is tunable so brute force is expensive. A **salt** is a unique random
addition per password: identical passwords produce different hashes, rainbow tables
become useless.
