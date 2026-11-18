# RSS OmniPOS — Database Architecture (Draft)

**Status:** Logical data architecture. This is not a finalized physical schema, migration set or deployed database.

## Core principles

1. The server-side relational database is the authoritative system of record.
2. Every business-owned record must be scoped to its business account.
3. Branch, warehouse, terminal and employee dimensions must be explicit where relevant.
4. Posted financial and inventory history must not be silently overwritten.
5. Correct posted errors through reversal, adjustment or compensating entries.
6. Multi-record business operations must be transactionally consistent.
7. Use stable identifiers and idempotency keys for retryable operations.
8. Preserve historical snapshots for price, cost, tax, exchange rate, margin policy and other values that can change.
9. Enforce tenant and role access in backend queries and service logic; UI filtering alone is insufficient.
10. Store timestamps consistently and render them using the business timezone.

## Logical domains and candidate entities

### Identity, tenancy and access
- businesses
- business_settings
- users
- user_credentials or identity_provider_links
- business_memberships
- roles
- permissions
- role_permissions
- user_role_assignments
- branches
- warehouses
- terminals
- device_registrations
- sessions
- access_audit_logs

A user may have access to multiple businesses only if explicitly supported and authorized. Membership and role scope must identify the relevant business and, where needed, branch.

### Products, pricing and inventory
- products
- product_variants
- product_barcodes
- product_units
- product_categories
- product_prices
- product_margin_policies
- stock_balances (derived or transactionally maintained projection)
- stock_movements
- stock_reservations
- inventory_batches
- serial_numbers
- warehouses
- stock_transfers
- stock_adjustments
- inventory_cost_layers or valuation records

Product margin is product-specific. Store the configured target and minimum margin with effective dates and approval history. Calculate margin from net revenue excluding applicable tax and discounts, against the applicable historical inventory cost. A below-minimum override requires authorization and an audit record.

### Purchasing and suppliers
- suppliers
- supplier_products
- purchase_orders
- purchase_order_lines
- goods_received_notes
- goods_received_lines
- supplier_invoices
- supplier_returns
- supplier_credit_notes
- supplier_payments

A goods receipt, supplier invoice and supplier payment are distinct business events and must be linked without assuming they occur simultaneously.

### Customers, sales and returns
- customers
- customer_addresses
- customer_credit_profiles
- customer_credit_ledger
- sales
- sale_lines
- sale_discounts
- sale_tax_lines
- sale_payments
- returns
- return_lines
- refunds
- quotations
- sales_orders
- held_sales

Posted sales retain product description, SKU, unit, price, discount, tax, cost, margin, branch, cashier and customer snapshots required for historical reporting.

### Payments and cash management
- payment_methods
- payment_transactions
- payment_attempts
- payment_allocations
- cash_drawers
- cash_sessions
- cash_movements
- cash_reconciliations
- bank_accounts
- bank_transactions
- bank_reconciliations

Payment states must distinguish pending, processing, authorized, completed, failed, cancelled, refunded and reversed outcomes. A request to pay is not proof that funds were received.

### Accounting and tax
- chart_of_accounts
- accounting_periods
- journals
- journal_lines
- tax_profiles
- tax_rates
- tax_transactions
- exchange_rates
- expense_categories
- expenses
- income_categories
- income_transactions
- budgets
- budget_lines
- period_closures

Every posted journal must balance: total debits equal total credits. Posted journals are immutable; corrections use reversals or adjustments. Financial statements must reconcile to the journal ledger and source transactions.

### Employees and payroll
- employees
- employee_employment_history
- departments
- job_positions
- attendance_events
- attendance_adjustments
- shifts
- leave_requests
- leave_ledger
- salary_components
- employee_compensation_history
- payroll_periods
- payroll_runs
- payroll_items
- payslips
- commission_plans
- commission_ledger
- employee_advances
- employee_loans
- expense_claims

Sensitive payroll and personal records require restricted access and separate preparation/approval permissions.

### Licensing and audit
- licence_accounts
- licence_entitlements
- licence_events
- trial_periods
- purchases
- renewals
- device_entitlements
- user_entitlements
- audit_events
- approval_requests
- idempotency_records
- integration_events

Licence events and audit events should be append-only or otherwise tamper-evident within the selected architecture.

## Data integrity requirements

- Use primary keys and foreign keys for entity relationships.
- Add uniqueness constraints for identifiers within their intended business scope.
- Use decimal/numeric types for monetary amounts; never binary floating point for financial calculations.
- Store currency codes with monetary values.
- Define rounding rules explicitly and consistently.
- Use effective-dated policy records for tax, pricing, margin and payroll rules.
- Use transaction boundaries for sale + stock + payment + accounting posting.
- Make idempotency scope explicit (business, operation and idempotency key).
- Avoid storing derived balances as independent truth without reconciliation.
- Use optimistic concurrency/version checks for records that can be edited concurrently.

## Tenant and branch isolation

Every business-owned record must be linked to a business identifier either directly or through a rigorously enforced parent relationship. Queries and mutations must validate business membership and branch permissions. Never trust a business or branch ID supplied by a client without authorization checks.

Cross-business support, if required for an operator account, must be explicitly modeled and audited. Do not implement cross-tenant joins in ordinary user workflows.

## Ledger and history rules

- Stock movement history is authoritative; a displayed stock balance is a projection.
- Accounting journal lines are authoritative for posted accounting; reports are derived.
- Customer credit and loyalty balances should be supported by transaction ledgers.
- Payroll and commission adjustments must preserve original calculations and approval history.
- Returns and refunds must reference the original transaction when applicable.
- Reversals must reference the event they correct and explain the reason.

## Privacy, retention and backup

Define data classification, retention, deletion/anonymization, export and backup policies before production. Backups must be access-controlled and encrypted. Restore tests are required; the existence of a backup is not proof that recovery works.

## Decisions still open

- Final relational provider, region and schema migration tool
- Exact primary key strategy and identifier formats
- Partitioning and archival strategy at expected scale
- Tenant isolation implementation (application checks, database policies or both)
- Data retention and deletion obligations
- Backup frequency, RPO/RTO and restore-test schedule
- Reporting projections and analytics storage strategy
