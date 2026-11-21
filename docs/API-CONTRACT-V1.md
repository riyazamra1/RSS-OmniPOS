# RSS OmniPOS — API Contract V1 (Initial Draft)

**Status:** Initial contract proposal for review. Endpoint names, payloads and status codes are not frozen. No API is implemented or deployed by this document.

## Contract conventions

- Base path: `/api/v1`; the externally configured origin/base path may change.
- JSON request and response bodies unless an endpoint explicitly documents another media type.
- UTF-8; timestamps in ISO 8601 UTC; business-local display timezone is returned in business settings.
- Monetary values are decimal strings plus ISO currency code to avoid binary floating-point ambiguity.
- IDs are opaque strings; clients must not infer tenant ownership from an ID.
- Authenticated routes require a valid session and server-resolved business membership.
- All protected object reads and writes enforce business and branch scope.
- Use pagination for collections; stable cursor pagination is preferred for high-volume ledgers.

## Standard response shapes

Success:

```json
{
  "data": {},
  "meta": {
    "request_id": "opaque-correlation-id"
  }
}
```

Error:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request could not be processed.",
    "details": [],
    "request_id": "opaque-correlation-id"
  }
}
```

Do not expose stack traces, SQL, secret values, internal hostnames or sensitive authorization details. Error messages must not reveal whether an unrelated user's account or resource exists.

## Authentication and context

Proposed initial routes:
- `POST /auth/login`
- `POST /auth/logout`
- `POST /auth/refresh`
- `GET /auth/session`
- `POST /auth/password/forgot`
- `POST /auth/password/reset`
- `POST /auth/email/verify`

Exact identity provider, MFA, recovery controls, token format and session lifetime remain open. Browser cookies must be Secure, HttpOnly and configured with an appropriate SameSite policy. Native clients must use platform-secure credential storage.

Business context:
- `GET /businesses` — list businesses the authenticated user is authorized to access.
- `GET /businesses/{businessId}`
- `PATCH /businesses/{businessId}`
- `GET /businesses/{businessId}/settings`
- `PATCH /businesses/{businessId}/settings`

The server must verify membership for the selected business on every request. A client-provided business ID is a context selector, not proof of access.

## Administration and access control

- `GET /businesses/{businessId}/members`
- `POST /businesses/{businessId}/invitations`
- `PATCH /businesses/{businessId}/members/{membershipId}`
- `GET /businesses/{businessId}/roles`
- `GET /businesses/{businessId}/permissions`
- `GET /businesses/{businessId}/branches`
- `POST /businesses/{businessId}/branches`
- `GET /businesses/{businessId}/devices`
- `POST /businesses/{businessId}/devices`
- `POST /businesses/{businessId}/devices/{deviceId}/revoke`

Administrative APIs must independently enforce administrator permission, business scope and operation-specific approval. The temporary admin URL path is not a security control.

## Product and margin policy

- `GET /businesses/{businessId}/products`
- `POST /businesses/{businessId}/products`
- `GET /businesses/{businessId}/products/{productId}`
- `PATCH /businesses/{businessId}/products/{productId}`
- `GET /businesses/{businessId}/products/{productId}/margin-policy`
- `PUT /businesses/{businessId}/products/{productId}/margin-policy`
- `POST /businesses/{businessId}/pricing/price-guidance`

Illustrative price-guidance request:

```json
{
  "product_id": "opaque-product-id",
  "unit_cost": "600.00",
  "currency": "LKR",
  "target_margin_percent": "25.00"
}
```

Illustrative response:

```json
{
  "data": {
    "suggested_unit_price": "800.00",
    "currency": "LKR",
    "target_margin_percent": "25.00"
  },
  "meta": { "request_id": "opaque-correlation-id" }
}
```

The guidance example uses `cost / (1 - margin)`. Actual checkout margin must be calculated on the server after discounts and excluding applicable tax, against the effective historical inventory cost and policy. The client cannot authorize its own below-minimum override.

## Inventory and purchasing

Proposed routes:
- `/suppliers`, `/purchase-orders`, `/goods-received-notes`, `/supplier-invoices`, `/supplier-returns`
- `/inventory/balances`, `/inventory/movements`, `/inventory/adjustments`, `/inventory/transfers`, `/inventory/reservations`

Create/approve/receive/dispatch/complete actions should use explicit action endpoints or documented state transitions. Do not allow clients to directly set protected status fields through generic PATCH operations. Posted movements are corrected by reversal/adjustment, not deletion.

## Sales, returns, payments and cash

Proposed routes:
- `POST /sales`, `GET /sales/{saleId}`, `GET /sales`
- `POST /sales/{saleId}/returns`
- `POST /returns/{returnId}/refunds`
- `GET /payments/{paymentId}`, `POST /payments/{paymentId}/capture` where provider flow supports it
- `GET /cash-sessions`, `POST /cash-sessions`, `POST /cash-sessions/{sessionId}/close`
- `POST /sync/batches`, `GET /sync/batches/{batchId}`

Sale, payment, refund and sync creation must accept an idempotency key, for example via `Idempotency-Key` header. The server stores the request hash and original outcome. Same key with different payload is rejected. A timeout is an unknown outcome until the same idempotency identity is queried/retried.

Payment states must distinguish pending, processing, authorized, completed, failed, cancelled, refunded, partially refunded and reversed. A refund request is not a completed refund. Offline transactions must carry stable device and transaction identity and be revalidated by the server.

## Pagination, filtering and sorting

Collection endpoints should support:
- `limit` with a documented maximum.
- Opaque `cursor` for stable continuation.
- Allowlisted filters and sort keys only.
- Explicit branch/date/status filters where relevant.
- Consistent `next_cursor` metadata.

Never interpolate arbitrary client sort or filter strings into SQL.

## Error code baseline

| Code | Meaning |
|---|---|
| `VALIDATION_ERROR` | Payload or field validation failed |
| `UNAUTHENTICATED` | Missing, invalid, expired or revoked authentication |
| `FORBIDDEN` | Caller lacks required permission or scope |
| `NOT_FOUND` | Resource not available in the caller's authorized scope |
| `CONFLICT` | State/version conflict or duplicate incompatible operation |
| `IDEMPOTENCY_CONFLICT` | Key reused with a different request payload |
| `POLICY_BLOCKED` | Business policy (such as minimum margin) blocks the operation |
| `APPROVAL_REQUIRED` | An authorized approval is required |
| `LICENCE_RESTRICTED` | Requested operation is outside current entitlement |
| `RATE_LIMITED` | Request rate exceeds policy |
| `INTERNAL_ERROR` | Unexpected server error; response contains no internal details |

HTTP status mapping and retry guidance must be finalized with the implementation and documented for each endpoint.

## Compatibility and contract governance

- Version breaking changes deliberately; do not silently change monetary, tax or state semantics.
- Document minimum supported Windows/Android client versions.
- Keep generated API schemas and contract tests in sync with implementation.
- Mark draft examples as illustrative until approved.
- Record deprecation windows and migration guidance before removing a contract.
