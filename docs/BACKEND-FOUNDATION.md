# RSS OmniPOS — Backend Foundation (Design Draft)

**Status:** Proposed service design only. No backend code, Worker, database, Hyperdrive binding, R2 bucket or secret has been created. Cloudflare Workers + TypeScript, Hyperdrive and Neon PostgreSQL are candidate components subject to separate approval and validation.

## Responsibilities

The backend is the authoritative boundary for:
- Authentication, session validation and revocation.
- Business membership, branch/warehouse/terminal scope and permission checks.
- Product, pricing, margin and discount policy.
- Purchasing, stock movements, reservations, sales and returns.
- Payment state, idempotency and external-provider reconciliation.
- Licensing, device/user entitlements and feature access.
- Audit events, approvals and administrative operations.
- Data export, reporting and controlled synchronization.

Windows, native Android and web clients must not connect directly to the production database or decide authoritative business outcomes.

## Proposed logical service layout

```text
src/
  app/                 # application composition and startup
  config/              # validated environment configuration
  routes/              # versioned HTTP route registration
  middleware/          # request ID, auth context, rate limits, error mapping
  modules/
    identity/
    businesses/
    access-control/
    branches/
    devices/
    catalogue/
    pricing/
    margin/
    suppliers/
    purchasing/
    inventory/
    customers/
    sales/
    payments/
    cash/
    licensing/
    audit/
    sync/
  shared/
    database/          # transaction and query interfaces
    errors/
    validation/
    money/             # decimal arithmetic, currency and rounding
    idempotency/
    events/            # outbox and event contracts
  tests/
    unit/
    integration/
    security/
```

Keep modules organized by business capability. Route handlers should remain thin: parse and validate input, establish request context, call an application service, and map the result to a versioned response. Business rules belong in application/domain services, not in UI code or scattered route handlers.

## Request lifecycle

1. Assign or validate a correlation/request ID.
2. Apply request size, content-type and rate-limit controls.
3. Authenticate the caller; reject invalid, expired or revoked sessions.
4. Resolve active business membership and requested branch/warehouse/terminal context.
5. Load effective permissions and licence entitlements from trusted server-side state.
6. Validate payload schema and operation-specific invariants.
7. Execute the application service within an explicit transaction where required.
8. Persist audit/outbox/idempotency state in the same transaction as the domain change when applicable.
9. Return a stable versioned response with safe error details and correlation ID.

Never trust client-supplied actor, role, business or branch identifiers without resolving and authorizing them against the authenticated identity.

## Authorization model

- Deny by default.
- Check authorization on every protected operation and every object reference.
- Enforce business isolation in repository/query methods as well as service policy.
- Validate branch, warehouse, terminal and employee scope independently.
- Separate permission to request an operation from permission to approve it.
- Require reason and audit trail for sensitive overrides.
- Do not rely on hidden routes, buttons, client-side role checks or unguessable IDs.
- Revocation of membership/device/session must take effect according to a documented cache/session invalidation policy.

## Data access and transactions

- Use parameterized queries and a reviewed database access layer.
- Pass business scope explicitly to repository methods; avoid unscoped lookups.
- Use transactions for sale + stock + synchronous payment/accounting changes.
- Use row locks or optimistic version checks for concurrent stock, credit, price and cash-session operations.
- Store money in decimal types and use a shared tested money/rounding library.
- Use an outbox for events that must be published after commit.
- Use compensating events/reversals rather than mutating posted ledger history.
- Keep database credentials in runtime secret storage, never in Git or client bundles.

## Idempotency and external operations

For retryable operations, require a stable idempotency key and persist its scope, request hash and outcome. Same key/same payload returns the original result; same key/different payload is rejected. Provider timeouts must be reconciled using provider references and status queries rather than blindly issuing a second charge/refund.

External payment integrations are not assumed until a provider is selected and its API, security, settlement and operational behavior are reviewed.

## Configuration and environments

Configuration must be schema-validated at startup and separated by environment. Candidate variables/bindings include:
- Database connection binding (for example, Hyperdrive if selected).
- Authentication issuer/audience and signing/verification configuration.
- Public/admin base URLs, allowed origins, cookie domain/path and OAuth callback URLs.
- R2 bucket binding if object storage is approved.
- Payment-provider credentials only for explicitly enabled providers.
- Rate-limit, logging and feature-policy configuration.

Never place secrets in source control, public configuration, mobile application packages or logs. Do not hardcode `rssapps.net/accounts/OmniPOS` paths into domain logic; configure the current temporary paths and later domain migration independently.

## Observability and operational controls

- Structured logs with correlation ID, route, duration, result and safe actor/business references.
- Redact authorization headers, tokens, credentials, personal/payment data and decrypted payloads.
- Metrics for request errors/latency, transaction failures, idempotency conflicts, sync backlog and provider reconciliation.
- Alerts for repeated authorization failures, unusual refund/override activity and failed backups.
- Health/readiness checks must not disclose secrets, SQL, internal topology or customer data.
- Define retention and access controls for logs and traces.

## Testing requirements

- Unit tests for domain rules, margin calculations, rounding and state transitions.
- Integration tests against an isolated test database and migrations.
- Tenant-isolation tests for every module and cross-business identifier attack cases.
- Permission matrix tests, including approval separation and revoked access.
- Concurrency tests for stock, credit, payments and idempotency.
- Failure-injection tests for database timeout, provider timeout and interrupted sync.
- Contract tests for Windows, Android and admin clients.
- Dependency, static-analysis and secret-scanning checks.

## Deployment boundary

A local/test implementation may be planned after the schema and API contracts are reviewed. Creating or binding production services, databases, domains, secrets, payment integrations or deployment pipelines requires a separate explicit approval and resource inventory. Production deployment requires a tested rollback path, backup/restore evidence, security review and owner release approval.
