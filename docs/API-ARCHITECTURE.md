# RSS OmniPOS — API Architecture (Draft)

**Status:** Logical API design. No production API or endpoint implementation is represented here.

## Objectives

- Provide one consistent business-rule boundary for Windows, Android, web administration and future customer portals.
- Enforce identity, business membership, branch scope and permissions on every protected operation.
- Keep clients independent of database implementation details.
- Support online and controlled offline workflows.
- Make retryable operations safe through idempotency and explicit transaction states.

## Logical API areas

- Authentication and sessions
- Business profile and settings
- Users, roles, permissions and device registration
- Branches, warehouses and terminals
- Products, categories, barcodes, units and margin policies
- Suppliers, purchasing and goods receiving
- Inventory, transfers, adjustments and stock counts
- Customers, credit and loyalty
- Sales, quotations, orders, returns and refunds
- Payments, cash drawers and reconciliation
- Accounting, tax, expenses and reports
- Employees, attendance, leave, payroll and commission
- Fixed assets, depreciation and budgets
- Licensing, trials, purchases, renewals and entitlements
- Audit, approvals, notifications and synchronization

Exact endpoint names and API versioning policy will be established during implementation.

## Request context and authorization

For an authenticated request, the backend must resolve:
- Authenticated user or service identity
- Active business membership
- Authorized branch/warehouse/terminal context
- Effective permissions and any approval requirements
- Device/session status where relevant
- Licence and feature entitlements where applicable

Client-supplied business, branch, user or role identifiers are untrusted input until checked against the authenticated identity and access policy.

Authorization must be enforced in service logic and data access. Hiding a button or route is not authorization.

## Public website and administration routes

Current temporary web locations:
- Public website: https://rssapps.net/accounts/OmniPOS
- Administrator portal: https://rssapps.net/accounts/OmniPOS/admin

The admin route is an access-controlled interface, not a security boundary by itself. All administrative APIs must independently verify administrator identity, permissions, business scope and operation-specific approval.

The future customer domain is not yet purchased. Public base URL, admin base URL, allowed origins, callback URLs and cookie settings must be environment-configurable. Avoid embedding the current path in backend business logic. Customer and administrator access policies must remain separate even if the interfaces initially share a host.

## Authentication and session requirements

- Use a reviewed authentication protocol and secure password handling if local credentials are supported.
- Prefer secure, short-lived access credentials with a controlled refresh/session mechanism.
- Store credentials using platform-secure storage on native clients.
- Use secure, HttpOnly and appropriate SameSite cookie settings for browser sessions.
- Protect password recovery and account verification against enumeration and abuse.
- Provide session revocation, device/session review and audit events.
- Require stronger verification for high-risk administrative actions where appropriate.
- Never log passwords, access tokens, recovery codes or secret values.

Identity provider choices, MFA policy, session lifetime and recovery flow remain open decisions.

## Request validation and error handling

- Validate types, lengths, ranges, enumerations and business invariants server-side.
- Reject unknown or unauthorized fields rather than mass-assigning client objects.
- Use consistent machine-readable error codes and safe human-readable messages.
- Do not return stack traces, SQL, secrets or internal infrastructure details.
- Include correlation/request IDs for support and audit tracing.
- Apply rate limits to authentication, recovery, payment and other abuse-sensitive operations.

## Transaction and idempotency behavior

For operations that can be retried:
- Require or generate a stable idempotency key.
- Scope the key to the business, authenticated actor and operation.
- Persist the operation outcome so retries return a consistent result.
- Prevent duplicate sales, payment capture, refunds, loyalty entries and offline sync imports.
- Use database transactions for coupled stock, sales, payment and accounting changes.
- Represent asynchronous provider operations with explicit states and reconciliation.

A timeout does not prove that a transaction failed. Clients must query or retry using the same idempotency identity rather than blindly submitting a new operation.

## Offline synchronization API

Synchronization must distinguish:
- Client-created immutable transaction identity
- Business, branch, terminal, user and device identity
- Local event time and server receipt time
- Schema and client version
- Idempotency key and payload integrity
- Validation result and conflict state
- Accepted, rejected, pending-review or duplicate outcome

Server-side rules remain authoritative. Offline submissions must be revalidated for user access, price/margin policy, stock policy, credit policy and licence constraints. The server must not silently rewrite the original client event.

## Administration and audit

Sensitive operations include role changes, margin overrides, refunds, stock adjustments, accounting journals, payroll approvals, licence changes and device revocation. Require operation-specific permissions, reasons and approvals where configured. Record actor, target, business scope, timestamp, outcome and correlation ID.

## API versioning and compatibility

- Version public contracts deliberately.
- Maintain backward compatibility for supported app versions where feasible.
- Include minimum supported client version and upgrade behavior.
- Reject unsupported schema versions with actionable error responses.
- Document deprecation windows before removing contracts.

## Decisions still open

- REST versus RPC-style contract and schema tooling
- Authentication provider and MFA requirements
- Token/session lifetime and refresh/revocation behavior
- API versioning and compatibility policy
- Rate limits and abuse thresholds
- Exact endpoint inventory and payload schemas
- CORS, cookie, callback and allowed-origin configuration for the temporary and future domains
