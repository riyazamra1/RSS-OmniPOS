# RSS OmniPOS — Database Schema V1 (Logical Draft)

**Status:** Initial logical schema proposal for review. This is not executable SQL, an approved physical schema, a migration or a provisioned database. Provider, region, migration tool and tenant-isolation mechanism remain open.

## Design conventions

- Use PostgreSQL-compatible relational modeling if PostgreSQL is selected.
- Use UUID primary keys unless a reviewed decision selects another stable identifier strategy.
- Use `timestamptz` for event timestamps; display in the business timezone.
- Use `numeric`/decimal for monetary values and quantities requiring precision. Never use binary floating point for financial calculations.
- Store ISO currency codes with money values; LKR is the base currency and USD is additionally supported.
- Include `business_id` on every business-owned table, even where a parent relationship also carries scope, unless a documented exception is justified.
- Use foreign keys, scoped unique constraints, check constraints and indexes for integrity.
- Prefer effective-dated policy rows and immutable posted-event/ledger records.
- Use explicit status columns with documented state transitions.
- This schema describes a starting foundation; later accounting, payroll, fixed-asset and analytics tables must extend it without weakening these rules.

## Identity and tenancy

| Table | Core fields | Integrity and intent |
|---|---|---|
| `businesses` | id, business_code, legal_name, display_name, base_currency, timezone, status, created_at | Unique business_code; base currency initially LKR |
| `business_settings` | id, business_id, setting_key, setting_value, updated_at | Unique (business_id, setting_key); validate sensitive settings by schema |
| `users` | id, email_normalized, display_name, status, created_at | Unique normalized email where account model requires it |
| `business_memberships` | id, business_id, user_id, status, created_at | Unique (business_id, user_id) |
| `roles` | id, business_id nullable, role_key, name, is_system | Unique role key within scope; system roles controlled |
| `permissions` | id, permission_key, description | Unique permission_key |
| `role_permissions` | role_id, permission_id | Composite primary key |
| `membership_roles` | membership_id, role_id, branch_id nullable | Scope role assignment; validate branch belongs to same business |
| `branches` | id, business_id, branch_code, name, status | Unique (business_id, branch_code) |
| `warehouses` | id, business_id, branch_id, warehouse_code, name, status | Unique warehouse code within business |
| `terminals` | id, business_id, branch_id, terminal_code, status | Unique terminal code within business |
| `device_registrations` | id, business_id, device_key, platform, status, registered_at, revoked_at | Unique device key within business; never store raw private keys |
| `user_sessions` | id, user_id, business_id, device_id nullable, issued_at, expires_at, revoked_at | Session revocation and expiry; protect token material |

Identity provider links, password credentials and recovery tokens require a separate reviewed authentication design. Do not store plaintext passwords or reusable secrets.

## Product catalogue and pricing

| Table | Core fields | Integrity and intent |
|---|---|---|
| `product_categories` | id, business_id, parent_id nullable, code, name, status | Scoped category hierarchy |
| `products` | id, business_id, sku, name, category_id, brand, status, track_stock, tax_profile_id nullable | Unique (business_id, sku) |
| `product_variants` | id, business_id, product_id, variant_code, attributes | Scoped unique variant code |
| `product_barcodes` | id, business_id, product_id, variant_id nullable, barcode, unit_id nullable | Unique barcode within business unless an approved exception |
| `units_of_measure` | id, business_id, unit_code, name, precision | Unique (business_id, unit_code) |
| `product_prices` | id, business_id, product_id, price_group_id, currency_code, unit_price, valid_from, valid_to | Effective-dated price; nonnegative price |
| `price_groups` | id, business_id, code, name, status | Retail/wholesale/VIP or customer-specific groups |
| `product_margin_policies` | id, business_id, product_id, target_margin_pct, minimum_margin_pct, valid_from, valid_to, approval_policy | Target and minimum are product-specific; enforce 0 <= minimum <= target < 100 |
| `product_cost_snapshots` | id, business_id, product_id, source_movement_id, unit_cost, currency_code, effective_at | Preserve cost basis used by posted sales |

Margin is gross profit divided by net selling revenue, excluding applicable tax. Net revenue is after discounts. Suggested selling price for a target margin is `cost / (1 - target_margin)`. Validate final sale margin server-side using the effective policy and historical inventory cost. A below-minimum override must record actor, reason, approver (if required), policy version and calculated result.

## Suppliers and purchasing

- `suppliers`: id, business_id, supplier_code, name, contact fields, payment_terms, credit_limit, status.
- `supplier_products`: supplier/product mapping and supplier-specific item code.
- `purchase_orders` and `purchase_order_lines`: order state, quantities, quoted cost, approvals.
- `goods_received_notes` and `goods_received_lines`: received/accepted/rejected/damaged quantities, batch, expiry, serial and landed-cost details.
- `supplier_invoices`: invoice reference, currency, totals, due date and matching state.
- `supplier_returns` and `supplier_credit_notes`: returned quantities and supplier credits.
- `supplier_payments`: payment state and allocation references.

Purchase order, goods receipt, supplier invoice and payment are separate events. Enforce business scope on all relationships. Partial receipts and partial invoices must be representable.

## Inventory and costing

- `inventory_batches`: product/variant, warehouse, batch code, manufacture/expiry dates and status.
- `stock_movements`: immutable movement identity, business, product, warehouse, quantity delta, movement type, source reference, unit cost, currency, actor and occurred/recorded timestamps.
- `stock_balances`: rebuildable or reconciled projection keyed by business/product/warehouse/batch.
- `stock_reservations`: reserved quantity, source transaction, expiry and release state.
- `stock_transfers` and `stock_transfer_lines`: source/destination, dispatch, in-transit and receiving states.
- `stock_adjustments`: reason, approval and linked movement(s).
- `inventory_cost_layers` or equivalent valuation records: FIFO, weighted-average or batch valuation history.
- `serial_numbers`: serialized unit identity and lifecycle.

Never silently edit posted stock movements. Correct by reversal/compensating movement. Destination stock is unavailable until receipt is confirmed. Prevent negative stock by default; any configured override must be authorized and audited. Preserve traceability from sale/return to batch or serial where tracking is enabled.

## Customers, sales and payments

- `customers`, `customer_addresses`, `customer_credit_profiles`, `customer_credit_ledger`.
- `sales`, `sale_lines`, `sale_discounts`, `sale_tax_lines`.
- `sale_returns`, `sale_return_lines`, `refunds`.
- `payment_methods`, `payment_transactions`, `payment_attempts`, `payment_allocations`.
- `cash_drawers`, `cash_sessions`, `cash_movements`, `cash_reconciliations`.
- `held_sales`, `quotations`, `sales_orders`.

Posted sale lines preserve SKU, description, unit, quantity, original price, discount, tax, net revenue, historical cost, gross profit/margin and applicable policy snapshot. Walk-in sales must not require customer registration. Returns reference original sale lines and cannot exceed sold quantity less prior accepted returns. A refund request is not a completed refund.

Payment attempts and payment transactions must have explicit states and provider references where applicable. Use idempotency records for sale creation, payment capture, refund and offline imports. Cash drawer movements are not automatically revenue.

## Audit, approvals and idempotency

- `audit_events`: business_id, actor_id, action, entity_type, entity_id, reason, before/after metadata (redacted), correlation_id, occurred_at.
- `approval_requests`: business_id, request_type, target reference, requester, approver, state, reason and timestamps.
- `idempotency_records`: business_id, actor_id, operation, idempotency_key, request_hash, response reference/state and expiry.
- `integration_events`: provider, external reference, event type, processing state and deduplication key.

Unique idempotency scope should include business, actor/operation as appropriate, and key. Reusing a key with a different request hash must be rejected. Audit records must not contain passwords, tokens, payment credentials or unnecessary personal data.

## Required indexes and constraints

At minimum, review and implement:
- Unique (business_id, branch_code), (business_id, warehouse_code), (business_id, sku), and other scoped business identifiers.
- Foreign keys that prevent cross-business relationships; where a simple FK cannot enforce same-business scope, use composite keys or validated service/database constraints.
- Indexes for business_id plus common filters (status, created_at, branch_id, product_id, customer_id).
- Indexes on stock movement source references, sale invoice numbers, payment provider references and idempotency keys.
- Check constraints for nonnegative quantities/prices where valid, currency codes, valid date ranges and policy percentages.
- Concurrency controls for stock reservation, price/policy changes, credit limits and cash session closure.
- Monetary precision, rounding mode, tax allocation and unit conversion rules documented before migration creation.

## Transaction boundaries

A completed sale must not leave stock, payment, sale and accounting records in contradictory states. Use a database transaction for synchronous coupled writes; use an outbox/reconciliation workflow for external asynchronous providers. Never assume a network timeout means the transaction rolled back. Retries use the same idempotency identity.

## Deferred domains

Accounting journals/periods, tax, AR/AP, employee/payroll, attendance, fixed assets, depreciation, budgets, loyalty, advanced reporting and analytics will receive detailed schema extensions in their respective milestones. This V1 foundation must reserve stable business, branch, user, currency, audit and source-reference conventions for them.

## Open decisions before physical schema

- Final PostgreSQL provider, region, connection model and migration tooling.
- UUID versus alternative ID format and public business code format.
- Row-level security versus application-enforced isolation or both.
- Exact inventory valuation implementation and negative-stock policy.
- Authentication provider, credential tables and MFA/session design.
- Retention, deletion/anonymization, backup, RPO/RTO and restore policy.
- Data volumes, partitioning, indexing and reporting projections.
