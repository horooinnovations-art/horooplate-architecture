# Security Architecture

HOROOPLATE holds a venue's takings, bank account numbers, salaries and guest contact details.
Its security model is defence in depth: each layer assumes the one in front of it might fail.
It inherits HOROOMART's security core and adds the controls a restaurant needs — staff on
shared phones and guests with no login.

## Authentication

| Control | Behaviour |
|---|---|
| **Password policy** | Configurable **per organization**: minimum length plus required character classes. Applied to password set, change and reset alike. |
| **Two-factor authentication** | TOTP (RFC 6238, any standard authenticator app). **Mandatory** for the Super Admin; available to every other user. Secrets are encrypted at rest. |
| **TOTP replay protection** | A one-time code is accepted **once**. Re-submitting a code already used within its validity window is rejected. |
| **Trusted devices** | A user may mark a device as trusted to skip the second factor on it. Administrators can list and **block** trusted devices, revoking that trust immediately. |
| **Brute-force throttling** | Login, password-reset and verification endpoints are rate-limited per client. |
| **Forced password change** | When an administrator resets a password, the user must set their own before reaching anything else. |
| **Session invalidation** | Changing a password terminates the user's **other** sessions. |
| **No account enumeration** | Password-reset responses are identical whether or not the email exists. |
| **Sessions** | Session data is encrypted by default, and each organization sets its own idle-session timeout. |

## Authorization

- **Role-based access control** with fine-grained permissions. Each role sees only what it may
  open — a waiter sees the floor and their own tickets, kitchen staff see the kitchen display —
  and every action is checked on the server, not only hidden in the menu.
- **Sensitive actions carry their own permission**: voiding an item the kitchen already has,
  voiding a whole ticket, comps, raising a tab's limit, approving wastage and stock counts. The
  standard **Supervisor** role holds these for the floor; a cashier or waiter does not.
- **Tenant isolation** runs as middleware on every authenticated request and as query scopes
  on operational data. See [ARCHITECTURE.md](ARCHITECTURE.md#multi-tenancy).
- **Segregation of duties** is enforced in code: nobody approves a record they created.
- **API access** uses revocable per-user tokens, with the same permission checks as the web
  interface and a regression test guarding against an endpoint exposed without them.

## Guests and shared screens

| Surface | Protection |
|---|---|
| **QR table ordering** | Each table's link carries a random token, not an ID. A guest can read the menu and place an order, nothing else; the order waits for staff to confirm before it reaches the kitchen. Reading is limited to 120 requests a minute, ordering to 10. |
| **Pickup screen** | A random-token link that shows order numbers and first names only — no amounts or payment details. If the link leaks, a manager regenerates it and the old one stops working. |
| **Staff phones** | An organization can require that phones record sales only within a set radius of their branch (geofencing), so a waiter's phone cannot ring up sales from outside the venue. |

## Data protection

**Field-level encryption** (AES-256 via the framework's encrypter) covers:

| Record | Encrypted fields |
|---|---|
| Bank accounts | account number, SWIFT code |
| Employees and users | salary |
| Customers | phone number |
| Users | two-factor secret |

For these fields a copy of the database alone is not enough; the application key, held outside
the database, is also required. Other data is protected by authentication, permissions, tenant
isolation and restricted database access.

**Audit logs redact sensitive fields**, so the audit trail never becomes a second copy of the
secrets it exists to protect.

**Backups** run daily with 14-day retention, are replicated off-site over SFTP, and are verified
daily; a failed check alerts administrators. Restores are a permission-gated, audited operation.

## Application hardening

| Threat | Control |
|---|---|
| **Cross-site scripting** | Output escaping by default; a **strict Content Security Policy** with no `unsafe-eval` (Alpine.js CSP build) and per-request script nonces; regression tests that store script payloads in editable fields and assert they render inert |
| **Host header attacks** | Requests are accepted only for the configured application hosts |
| **Clickjacking** | `X-Frame-Options` and CSP frame restrictions |
| **Protocol downgrade** | HTTP Strict Transport Security |
| **MIME sniffing** | `X-Content-Type-Options: nosniff` |
| **Browser features** | `Permissions-Policy` disables camera and microphone; geolocation is allowed only for the application itself (for geofencing) |
| **CSRF** | Framework CSRF tokens on every state-changing web request |
| **SQL injection** | Parameterized queries throughout (query builder / ORM) |
| **Price and tax tampering** | Prices, modifiers and the tax rate are resolved on the server; values sent by the client are ignored |
| **Mass assignment** | Explicit fillable whitelists |
| **Resource exhaustion via reports** | Report date ranges are capped, and heavy exports run on a background queue |

## Verification

Security properties are covered by automated tests that run with every change, among them:
middleware ordering, two-factor setup and replay rejection, session invalidation on password
change, reset enumeration resistance, CSP header content, stored-XSS inertness, API guard
coverage, segregation of duties and server-side pricing.

## Reporting a vulnerability

If you believe you have found a security issue in HOROOPLATE, please report it privately as
described in [SECURITY.md](../SECURITY.md). Never open a public issue for it.
