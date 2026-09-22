# HOROOPLATE — Architecture & Database Design

**HOROOPLATE** is a café, restaurant and bar management system built by
[HOROO Innovations](https://horooinnovations.com) for venues in Ethiopia. Table service, counter
service, bar tabs and QR ordering; the kitchen and bar displays; stock that follows the recipe;
shifts and cash-up; finance, payroll and reporting — for one outlet or a group of them, with
every record scoped to its organization.

This repository publishes the system's **design**: its database schema, domain architecture,
financial-integrity model, security architecture and engineering practices. The application
source code is proprietary and is not included, and no client data appears anywhere in it.

**Documentation:** [docs on `main`](https://github.com/horooinnovations-art/horooplate-architecture/tree/main/docs)
· **Website:** [horooinnovations.com](https://horooinnovations.com)
· **Maintainer:** [Haji Omer](https://github.com/Hadji4)

---

## What's here

| Document | Contents |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | Domains, service styles, the order lifecycle, recipe-driven stock, tenancy, integrations |
| [Financial integrity](docs/FINANCIAL_INTEGRITY.md) | The account ledger, server-side pricing, immutability, and the controls a venue needs: shifts, voids, comps, wastage |
| [Security architecture](docs/SECURITY_ARCHITECTURE.md) | Authentication, authorization, guests and shared screens, data protection, hardening |
| [Engineering practices](docs/ENGINEERING_PRACTICES.md) | Test strategy, concurrency control, data integrity, release integrity |
| [Database: data dictionary](database/SCHEMA.md) | Every table and column, with types, keys, defaults, foreign keys and indexes |
| [Database: ER diagrams](database/ERD.md) | One entity-relationship diagram per domain, plus a system-wide dependency map |
| [Database: DDL](database/schema.sql) | The complete MySQL schema, structure only, no data |

## The system at a glance

| | |
|---|---|
| **Service styles** | Table service · counter service with a pickup screen · bar tabs · QR ordering from the table — switched on per outlet |
| **Domains** | 20 bounded modules, from Restaurant, Kitchen, Menu and Loyalty to Financial, Inventory and HR |
| **Database** | 101 tables · 1,273 columns · 278 enforced foreign keys · 467 indexes |
| **Automated tests** | 606 tests · 2,229 assertions, run against a database migrated from empty |
| **Stack** | Laravel 13 · PHP 8.4 · MySQL 8 (InnoDB) · Blade + Alpine.js (CSP build) · Tailwind CSS · Vite |
| **Languages** | English, Amharic (አማርኛ), Afaan Oromoo |
| **Deployment** | Progressive Web App, phone-first for waiters and bar staff; REST API (`/api/v1`) with token authentication |

## Capabilities

- **Service.** Tables and areas, tickets that can be split, merged and moved, seats and courses,
  pre-bills with service charge, split payments, tips, and named bar tabs with limits.
- **Kitchen and bar.** Each line goes to its station's display (and printer), is bumped when
  ready, and can be recalled. Time-boxed menus, price overrides and modifiers.
- **Stock that follows the recipe.** Bottles are counted; dishes consume their ingredients in
  grams and millilitres, costed at the moment of sale. Production with yield, stock counts with
  variance, wastage with approval, and a central store that issues to outlets.
- **Money.** Shifts with float, expected and counted cash and a Z-report; voids, discounts and
  comps with reasons; credit (on-account) sales; banks and mobile-wallet accounts (e.g. Telebirr)
  on one ledger; expenses, payroll and loans.
- **Manager's day.** A dashboard of today's trading day: sales by hour and by service, what is
  open on the floor and in the kitchen, the shift, voids and comps, and what is running low.
- **Roles.** Waiter, Cashier, Barista, Bartender, Kitchen Staff, Head Chef, Supervisor, Stock
  Keeper, Accountant, Branch Manager and Admin, each seeing only what it may open.

## Design principles

1. **The database enforces integrity**, not just the application. Relationships are real foreign
   keys with a deliberate `ON DELETE` rule each.
2. **Every money movement reaches a ledger, or it fails loudly.** See
   [Financial integrity](docs/FINANCIAL_INTEGRITY.md).
3. **Retries are safe.** Checkouts and payments carry idempotency keys, so a double-tap or a
   replay after lost connectivity cannot double-post.
4. **Finalized records are immutable.** Corrections are new, auditable records: voids, credit
   notes, adjustments.
5. **Nobody approves their own work.** Segregation of duties is enforced in code.

---

© 2026 HOROO Innovations. All rights reserved. This repository is published for reference and
evaluation; see [LICENSE](LICENSE).
