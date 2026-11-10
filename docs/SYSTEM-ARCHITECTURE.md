# RSS OmniPOS — System Architecture (Draft)

**Status:** Proposed architecture for review. Not an implementation description.

## Logical components

- **POS clients:** Windows and native Android applications for checkout and operational workflows.
- **Backend API:** Authentication, authorization, business rules, transaction orchestration and integration endpoints.
- **Data layer:** Relational business records, transactional ledgers, audit history and reporting data.
- **Synchronization service:** Offline transaction upload, idempotency, conflict detection and reconciliation.
- **Licensing service:** Trial state, purchase/renewal status, device entitlements and user limits.
- **Administration:** Business, branch, warehouse, tax, pricing, user and policy configuration.
- **Integration adapters:** Printers, barcode scanners, cash drawers, payment providers and supported external systems.
- **Reporting:** Operational and financial reports with permission-aware data scopes.

## Core design rules

1. Validate permissions and business constraints on the server.
2. Use database transactions for operations that change related financial and inventory records.
3. Record immutable movement/ledger events and correct errors through reversal or adjustment.
4. Use idempotency keys for payments, sales synchronization and other retryable operations.
5. Preserve historical price, cost, tax, exchange-rate and policy snapshots on posted transactions.
6. Do not treat a client-side success screen as proof of backend completion.
7. Define offline limitations explicitly; current stock, credit and payment authorization may be unavailable offline.
8. Keep reporting projections reconcilable to authoritative transaction records.

## Proposed deployment direction

Cloudflare-first and low-cost infrastructure should be evaluated where suitable. Specific databases, compute products, bindings, regions, providers and paid services are not selected by this document. Inspect existing RSS infrastructure and obtain approval before creating new external resources.

## Open architecture decisions

- Final client frameworks and supported OS versions
- Backend runtime and API contract
- Database and migration strategy
- Offline data model and conflict resolution policy
- Payment-provider integrations
- Printer and peripheral compatibility matrix
- Backup, disaster recovery and retention targets
- Observability, alerting and operational support model
