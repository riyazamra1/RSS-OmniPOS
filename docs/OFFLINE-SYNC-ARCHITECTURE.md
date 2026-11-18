# RSS OmniPOS — Offline Operation and Synchronization (Draft)

**Status:** Behavioral design proposal. Offline functionality is not implemented or tested by this document.

## Goals

- Permit explicitly approved POS tasks during network interruption.
- Preserve each local transaction exactly once from the business perspective.
- Prevent duplicate sales, payments, returns, stock movements and loyalty entries.
- Revalidate business rules when the server receives queued work.
- Make conflicts visible and resolvable rather than silently overwriting data.

## Offline capability policy

Offline support must be defined per operation. A client must not imply that a server-dependent operation is complete when it has only been saved locally.

Potentially supportable offline tasks, subject to final policy:
- Read a limited, recently synchronized product catalogue and price snapshot.
- Create a local cash sale under an explicitly approved offline policy.
- Print a receipt marked with its offline/pending status.
- Queue stock and transaction events for later synchronization.
- View limited customer information already cached and authorized.

Operations that generally require online confirmation or an explicit offline exception:
- Current centralized stock availability across branches
- Credit-limit and overdue-account validation
- Online card, bank or wallet authorization
- Real-time licence/device entitlement checks
- High-risk refunds, payroll approvals and accounting period closure
- Changes to users, roles, tax configuration and margin policies

The final offline policy must state which payment methods, sale types and override rules are permitted.

## Local transaction envelope

Each queued operation should include:
- Globally unique transaction/event ID generated before first submission
- Business, branch, warehouse, terminal and device identifiers
- Authenticated user identity and locally known authorization context
- Client application and schema version
- Local creation timestamp and timezone context
- Server receipt timestamp added by the server
- Operation type and immutable payload
- Idempotency key
- Payload integrity metadata
- Synchronization state and attempt metadata

Sensitive local data must be minimized, encrypted at rest and protected by OS-level access controls. The client must not store server credentials in plaintext.

## Synchronization states

Suggested lifecycle:
- Local Draft
- Queued
- Uploading
- Received
- Validating
- Accepted
- Rejected
- Conflict Review
- Completed
- Duplicate Acknowledged

The UI must distinguish locally saved, transmitted, server-accepted and fully completed states. A network response timeout must not cause the client to create a new transaction identity.

## Idempotency and duplicate prevention

- Generate the transaction ID once and persist it before upload.
- Reuse the same ID and idempotency key for all retries.
- Enforce uniqueness on the server within the defined business and operation scope.
- Store the authoritative result of the first accepted submission.
- Return that result for duplicate retries rather than reapplying side effects.
- Apply the same principle to payment attempts, refunds, loyalty points and stock movements.
- Detect cloned or restored local databases that could replay old transaction IDs.

## Conflict handling

Potential conflicts include:
- Stock was sold or transferred elsewhere before offline sale sync.
- Product price, tax or margin policy changed while the terminal was offline.
- User or device access was revoked.
- Customer credit limit was reached.
- Another terminal submitted the same external reference.
- A product was deactivated or removed.
- An offline return references a sale that is not yet synchronized.

Do not silently merge financial or stock conflicts. Classify the conflict, preserve the submitted event, record server validation evidence and route it to an authorized resolution workflow.

## Stock and margin policy

Offline stock is a local snapshot, not a guarantee of current central availability. Define per-business policy for offline stock allowance, maximum offline duration/quantity/value and manager approval.

Margin must be computed from the locally approved price/cost/policy snapshot for display, then revalidated on the server at synchronization. A below-minimum margin sale must follow the configured block or authorized override policy. Any offline exception must be explicit, bounded and auditable.

## Payment handling

Cash received locally may be recorded as a locally collected payment subject to reconciliation. Card, QR, bank and other provider-backed payments must not be marked completed unless the provider or an approved offline payment process confirms the result.

Keep payment attempt identity separate from sale identity. Reconcile provider outcomes after connectivity returns. Do not convert an unknown payment outcome into a second charge.

## Recovery and operations

- Resume interrupted uploads safely.
- Retain queued events until server acknowledgment and local retention rules permit cleanup.
- Provide support diagnostics without exposing credentials or unnecessary customer data.
- Detect device clock drift and retain both local and server timestamps.
- Handle app upgrades with explicit local schema migration.
- Support controlled device revocation and secure local data deletion.
- Monitor queue age, rejected events and unresolved conflicts.

## Acceptance scenarios to test

1. Same offline sale uploaded repeatedly creates one sale.
2. Timeout after server commit does not create a duplicate on retry.
3. Offline sale conflicts with exhausted stock and follows configured resolution.
4. Price/margin policy changed while offline is revalidated.
5. Revoked user/device submission is rejected or reviewed according to policy.
6. Payment provider timeout does not cause duplicate capture.
7. Offline return cannot exceed the original sold quantity after prior returns.
8. App crash and restart preserve transaction identity and queue contents.
9. Old client schema is migrated or safely rejected.
10. Local pending state is never presented as server-completed.

## Decisions still open

- Exact offline sale and payment methods
- Maximum offline duration and value/quantity thresholds
- Negative stock and reserved stock behavior
- Conflict resolution roles and approval workflow
- Local cache retention and encryption approach
- Multi-device queue recovery and device replacement process
- Supported offline printer and cash-drawer behavior
