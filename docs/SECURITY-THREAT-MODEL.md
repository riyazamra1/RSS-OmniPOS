# RSS OmniPOS — Security Threat Model (Initial Draft)

**Status:** Initial design threat model. This is not a completed security assessment, penetration test or implemented control set.

## System assets

- Business, customer, employee and supplier records.
- Credentials, sessions, device identities and recovery flows.
- Product prices, cost data, product-specific margin policies and stock.
- Sales, payment references, refunds, cash drawer and accounting records.
- Licensing entitlements, source code, signing material and release artifacts.
- Audit trails, backups, uploaded documents and integration credentials.

## Trust boundaries

1. Windows/Android/web clients to public API.
2. One business tenant to another tenant.
3. Branch, warehouse, terminal and employee scopes within a business.
4. Public website to authenticated administration interface.
5. API runtime to database, object storage and external providers.
6. Trusted build/release workflow to source decryption and signing keys.
7. Offline device storage to server synchronization.
8. Human operators and support personnel to sensitive administrative functions.

The current public website path is `https://rssapps.net/accounts/OmniPOS`; the admin path is `https://rssapps.net/accounts/OmniPOS/admin`. These paths are not security boundaries. Future domain purchase is deferred; route/origin migration must not weaken authentication or authorization.

## Threats and required mitigations

| Threat | Example impact | Required design controls |
|---|---|---|
| Cross-tenant IDOR | User reads or changes another business's records | Deny-by-default authorization; business-scoped queries; composite constraints; cross-tenant negative tests |
| Privilege escalation | Cashier changes roles, refunds or margin rules | Server-side permission checks; approval separation; least privilege; audit |
| Session theft/replay | Attacker uses a stolen browser/native session | Secure cookie policy; platform secure storage; expiry/revocation; reauthentication for sensitive actions |
| Credential stuffing/account enumeration | Account takeover or identity disclosure | Rate limits; generic recovery responses; abuse monitoring; MFA policy review |
| Price or margin tampering | Client changes price/discount to bypass minimum margin | Server-side recomputation; effective policy and cost snapshots; override permission/reason/approval |
| Stock manipulation | Duplicate or unauthorized adjustment/transfer | Immutable movement ledger; scoped permissions; approvals; concurrency controls and reconciliation |
| Duplicate sale/payment/refund | Double charge, duplicate invoice or refund | Idempotency keys, request hashes, unique provider references and reconciliation |
| Offline replay/conflict | Stale or revoked device submits old transactions | Device revocation checks; stable event IDs; server revalidation; conflict queue; audit |
| SQL injection/mass assignment | Data disclosure or unauthorized field updates | Parameterized queries; strict schema validation; allowlisted fields; least-privilege DB role |
| Malicious upload | Malware or stored script delivered to users | Type/size validation; safe object naming; private storage; scanning where supported; no execution |
| Secret/source exposure | Public repository or logs disclose proprietary material | Secret scanning; no plaintext proprietary source; protected secrets; log redaction; artifact review |
| Compromised CI runner | Decrypted source or signing keys stolen | Isolated trusted runners; protected environments; short-lived credentials; restricted workflow triggers; cleanup; signing-key separation |
| Dependency/supply-chain compromise | Malicious package/action enters build | Lockfiles; dependency review; pinned actions; updates reviewed; provenance and artifact verification |
| Backup compromise or unusable restore | Data theft or prolonged outage | Encryption, access separation, retention policy, restore drills and recovery objectives |
| Insider misuse | Unauthorized export, refund, payroll or licence change | Least privilege; dual control for high-risk actions; tamper-evident audit; periodic access review |
| Domain/callback misconfiguration | Session leakage or OAuth redirect abuse | Environment-configured allowlists; exact callback validation; secure cookies; controlled migration checklist |
| Denial of service | API unavailable or excessive provider cost | Rate limits, quotas, request limits, backpressure, abuse alerts and recovery plan |

## Mandatory security invariants

- Every protected request is authenticated and authorized on the server.
- Every business-owned data access is tenant-scoped; branch/warehouse/terminal scope is checked where applicable.
- The client cannot set trusted actor, role, approval, payment-completed, stock-received or ledger-posted fields.
- Posted financial and inventory history is corrected through reversal/adjustment, not silent mutation.
- Sensitive operations record actor, target, business scope, reason, outcome and correlation ID.
- Secrets, passwords, tokens, recovery codes and payment credentials never appear in source control, client packages or logs.
- Public/admin route obscurity is never considered authorization.
- Offline events are untrusted until server validation succeeds.
- Encryption at rest does not protect plaintext while an authorized process or compromised runner uses it.

## Security testing plan

Before production:
1. Build an authorization matrix for roles, branches and business scopes.
2. Test cross-tenant and cross-branch read/write attempts for every resource family.
3. Test revoked membership, device and session behavior.
4. Test margin, discount, refund, stock and payroll approval boundaries.
5. Test idempotency under concurrent duplicate requests and network failures.
6. Test SQL injection, mass assignment, request validation and upload handling.
7. Review authentication, password recovery, MFA and session lifecycle.
8. Scan dependencies, repository history, artifacts and logs for exposed secrets.
9. Review CI workflow permissions, untrusted pull-request behavior and signing-key access.
10. Conduct independent penetration testing and remediate findings before production.

## Residual risks and open decisions

- Authentication provider, MFA, session duration and recovery process.
- Database tenant isolation: application checks, database row-level security or both.
- Encryption/key management, key custody, rotation and recovery.
- Data classification, retention, deletion/anonymization and privacy obligations.
- Payment provider threat model and reconciliation behavior.
- Offline risk policy and acceptable exposure for cash/credit/stock.
- Backup RPO/RTO, recovery ownership and incident response.
- Build runner isolation and signing architecture.

No security control should be marked implemented until code/configuration exists and its test evidence is recorded.
