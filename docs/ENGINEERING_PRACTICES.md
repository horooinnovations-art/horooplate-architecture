# Engineering Practices

## Testing

**606 automated tests, 2,229 assertions**, run against a real MySQL database, not
an in-memory substitute, because constraint enforcement, locking and SQL behaviour are exactly
the things worth testing. The suite builds the schema **by running every migration from an
empty database**, so a migration that only works on an already-evolved database fails.

### What the tests cover

| Area | Examples of what is asserted |
|---|---|
| **Service** | Opening, splitting, merging and transferring tickets · firing lines to the right station · voids need a reason and, once fired, a supervisor · bar-tab limits · counter pickup numbers · QR orders wait for staff |
| **Stock & recipes** | Selling a dish consumes its ingredients in their own units · unit conversions chain correctly · production yield · stock counts and variance · wastage approval · made-to-order dishes never show as "out of stock" |
| **Money** | Every non-cash payment posts to a ledger · shifts reconcile expected and counted cash · service charge, discounts and comps · clients can't set prices or the tax rate · credit-sale revenue counts only when collected · finalized records refuse edits |
| **Security & access** | Middleware order · segregation of duties · TOTP replay · session invalidation · reset enumeration · stored XSS · CSP · API guards · each role's menu and permissions |
| **Concurrency & idempotency** | A double-submitted approval takes effect once · row locks are proven to execute · document numbers are generated under lock · a replayed checkout records one sale |

### Regression discipline

Defects are fixed together with a test that reproduces them, and the test's docblock records
what happened and why. The suite is also a history of the failure modes the system has already
survived.

## Data integrity by construction

- **Foreign keys are enforced by the database** — 278 of them, each with a deliberate
  `ON DELETE` rule: `CASCADE` for owned children, `RESTRICT` where deletion would destroy
  financial history, `SET NULL` for optional links.
- **Money is `DECIMAL`**, never floating point.
- **Soft deletes** on master data and financial documents, so history stays reconstructible.
- **Document numbers** are generated under lock and unique per organization.

## Concurrency control

Read-modify-write paths on shared state take row locks inside a transaction: account balances,
stock quantities, tickets, approvals and document-number sequences. Each guarded action has a
test that invokes it twice against the same record, asserts the side effect happens once, and
captures the executed SQL to prove the `FOR UPDATE` lock really ran.

## Continuous integration

Every push runs the full suite against MySQL 8 from a clean environment:

```
checkout → install dependencies → generate app key → migrate from empty → run every test
```

## Release integrity

Each approved release is identified by a SHA-256 checksum over the application files, produced
in `sha256sum` format so an inspector can reproduce it.

## Deployment & operations

- **Assets are built before deployment** and shipped as artifacts, never compiled on the
  production host.
- **Scheduled operations** — backups, off-site verification, alerts — run from the framework
  scheduler, and a failure alerts administrators.
- **Configuration** lives in environment variables. Secrets are kept out of version control, and
  selected sensitive columns are encrypted with a key held outside the database.
