# RSS OmniPOS — Project Overview

## Purpose

RSS OmniPOS is planned as a multi-branch point-of-sale and business-management system for retail and related business operations.

## Planned functional domains

1. Product catalogue, variants, units, barcode and product-specific margin controls
2. Purchasing, suppliers, purchase orders and goods received notes
3. Inventory costing, stock movements, warehouses and branch transfers
4. POS checkout, sales, customers, returns, loyalty and credit
5. Payments, cash drawers, refunds and reconciliation
6. Double-entry accounting, tax configuration and financial reporting
7. Employee records, attendance, leave, payroll and sales commission
8. Fixed assets, depreciation, budgets and management analytics
9. Offline operation, synchronization and conflict handling
10. Licensing, device/user entitlements and business administration

## Established commercial constraints

- LKR is the primary currency; USD is additionally supported.
- Trial: 14 days with full-feature evaluation.
- After trial: one-time system purchase plus mandatory annual system-only renewal.
- One device is included with each system purchase; additional devices are charged separately.
- No lifetime licence and no permanent free plan.
- Proposed edition prices and user bands remain provisional unless separately approved.

## Architecture requirements

- Backend-enforced authorization and business rules.
- Business, branch, warehouse, terminal and employee/user scope separation.
- Auditable, reversible financial and inventory operations.
- Product-by-product target and minimum margin controls.
- Idempotent payment and offline synchronization workflows.
- Native Android client rather than a WebView wrapper.
- Public repository content must not disclose proprietary source or secrets.

## Specification status

Functional areas are being specified incrementally. Draft specifications are not automatically approved requirements, implemented features or tested capabilities. Each milestone must identify confirmed decisions, proposals and unresolved questions.
