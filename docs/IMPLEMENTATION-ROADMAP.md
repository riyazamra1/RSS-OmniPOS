# RSS OmniPOS — Implementation Roadmap

**Status:** Planning baseline. This roadmap is not evidence that any feature is implemented, deployed or tested. Milestone completion requires reviewed code, automated tests, security review and recorded acceptance evidence.

## Delivery principles

- Build one shared, server-authoritative business platform for Windows, native Android and web administration.
- Preserve the current temporary public route `https://rssapps.net/accounts/OmniPOS` and admin route `https://rssapps.net/accounts/OmniPOS/admin`. Keep origins, base paths, callbacks, cookie settings and allowed origins configurable. The future customer domain is deferred until the system is complete.
- Do not create infrastructure, domains, paid services, credentials or other external resources without separate explicit approval.
- Prefer Cloudflare-first and low-cost deployment where technically suitable. Cloudflare Workers, Hyperdrive, Neon PostgreSQL and R2 remain proposed components, not provisioned resources or final provider approvals.
- Keep proprietary source and secrets out of public plaintext Git history. Do not treat repository encryption as protection against a compromised trusted build runner.
- Distinguish confirmed requirements, proposals and unresolved decisions in every milestone.
- No production release until backup restoration, security, licensing, accounting and operational acceptance gates pass.

## Milestones

### M0 — Product and technical baseline
- Confirm functional scope, user roles, branch model, offline policy, licensing, commercial terms and deployment constraints.
- Resolve open decisions in the existing architecture documents.
- Produce traceable requirements and acceptance criteria.
- Gate: owner-reviewed scope baseline; unresolved items explicitly listed.

### M1 — Identity, tenancy and business administration
- Business registration/profile, settings, users, memberships, roles, permissions, sessions and device registration.
- Branches, warehouses and terminals with explicit scope.
- Enforce business and branch authorization on every protected API and database access path.
- Gate: cross-tenant and cross-branch negative tests; session revocation and audit tests.

### M2 — Product catalogue, pricing and margin policy
- Products, categories, variants, barcodes, units, price lists and product-specific margin policies.
- Target/minimum margin calculations and price guidance.
- Validate margin after discounts and excluding applicable tax, using the applicable historical inventory cost.
- Below-minimum sale requires configured block or authorized override with reason and audit record.
- Gate: calculation, rounding, tax, discount, override and concurrency tests.

### M3 — Suppliers, purchasing and inventory
- Suppliers, purchase orders, approvals, goods received notes, supplier invoices and returns.
- Stock movement ledger, costing, batches/serials, adjustments, reservations and branch transfers.
- Distinguish ordered, received, invoiced and paid quantities/events.
- Gate: inventory reconciliation, partial receipt, transfer-in-transit, costing and reversal tests.

### M4 — POS sales, customers and payments
- Customer profiles and walk-in checkout, retail/wholesale pricing, quotations, orders, sales, returns and refunds.
- Payment attempts, allocations, cash drawers, shifts and reconciliation.
- Atomic sale/stock/payment/accounting boundaries and idempotent retries.
- Gate: duplicate request, partial payment, refund, return, cash variance and failure-recovery tests.

### M5 — Offline operation and synchronization
- Native local persistence and controlled offline checkout workflow.
- Stable transaction/device identity, queueing, sync outcomes and conflict review.
- Revalidate access, price, margin, stock, credit, licence and payment policy on the server.
- Gate: duplicate sync, out-of-order events, revoked device, stale price, insufficient stock and recovery tests.

### M6 — Accounting, tax and financial reporting
- Double-entry journal engine, chart of accounts, periods, AR/AP, tax, expenses, cash/bank and reconciliations.
- LKR base currency with USD support; preserve transaction currency and historical exchange rates.
- Verify Sri Lankan statutory tax and reporting rules against current official sources before release; do not hardcode unverified rates.
- Gate: balanced journals, source-to-ledger reconciliation, period lock and statutory review.

### M7 — Employees, attendance, leave and payroll
- Employee records, access separation, attendance, shifts, leave, payroll, commission, advances and claims.
- Effective-dated rules, approval separation and restricted payslip access.
- Verify current Sri Lankan EPF, ETF, APIT and employment requirements before release.
- Gate: payroll calculation, privacy, duplicate prevention, approval, posting and payment tests.

### M8 — Fixed assets, depreciation, budgets and analytics
- Asset register, acquisition, capitalization, depreciation, disposal and impairment workflows.
- Budgets, approvals, actual-vs-budget and management analytics.
- Gate: depreciation schedule, disposal, budget authorization and report reconciliation tests.

### M9 — Native clients, administration and integrations
- Windows client and native Kotlin/Jetpack Compose Android client; no WebView wrapper for Android.
- Web administration and customer portal, with separate authorization policies.
- Receipt printing, barcode devices, supported payment providers and approved integrations.
- Gate: supported-device matrix, accessibility/usability review and end-to-end tests.

### M10 — Licensing, release hardening and operations
- 14-day full-feature trial; after trial, one-time system purchase plus mandatory annual system-only renewal; no lifetime licence or permanent free plan.
- One included device per system purchase; additional device entitlements charged separately. Proposed edition prices/user bands remain unapproved.
- Protected builds, signing, backup/restore, monitoring, incident response, release rollback and support procedures.
- Gate: licensing abuse tests, restore drill, security review, release provenance and explicit production approval.

## Cross-cutting completion gates

Each milestone must provide:
1. Requirements-to-test traceability and a decision log.
2. Reviewed code and reproducible build evidence.
3. Unit, integration, authorization and regression test results.
4. Migration/rollback and data-integrity plan where applicable.
5. Threat-model updates and dependency review.
6. Operational documentation and support diagnostics.
7. Owner acceptance before marking the milestone complete.

## Current implementation status

This document and the architecture documents are planning artifacts only. No application runtime, database, API, cloud infrastructure, secrets, customer domain or production deployment is created by this roadmap.
