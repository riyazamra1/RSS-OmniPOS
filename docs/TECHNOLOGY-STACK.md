# RSS OmniPOS — Technology Stack (Proposal)

**Status:** Architecture proposal. Technology choices in this document are not final approvals or evidence of deployed services.

## Selection principles

- Prefer reliable, maintainable and cost-conscious technologies.
- Keep Windows and Android clients native.
- Centralize authoritative business rules and permissions in a shared backend.
- Use standards-based APIs and portable data formats where practical.
- Avoid unnecessary paid infrastructure before the product and operating costs are validated.
- Do not create new external services, accounts, databases or paid resources without explicit approval.
- Record supported versions and upgrade policy when implementation begins.

## Proposed client platforms

### Windows desktop

**Candidate:** C# with modern .NET and a native Windows UI framework.

Expected responsibilities:
- POS checkout and peripheral integration
- Barcode scanners, receipt printers and supported cash drawers
- Local encrypted operational cache for approved offline functions
- Secure credential/session storage using platform facilities
- Updates, diagnostics and controlled synchronization

Framework selection, Windows minimum version, deployment packaging and peripheral compatibility remain open decisions.

### Android

**Candidate:** Kotlin with Jetpack Compose.

Expected responsibilities:
- Native Android POS workflows
- Camera/barcode support where required
- Supported Bluetooth/USB peripherals
- Encrypted local storage and controlled offline queue
- Android secure credential storage
- Background synchronization within Android platform limits

This project must not use a WebView wrapper as a replacement for a native Android application. Minimum SDK, target SDK, supported device classes and printer compatibility remain open.

## Proposed backend and data services

| Layer | Candidate | Status |
|---|---|---|
| API runtime | Cloudflare Workers with TypeScript | Proposed |
| Relational database | Neon PostgreSQL | Proposed |
| Database connectivity | Cloudflare Hyperdrive | Proposed |
| Object and document storage | Cloudflare R2 | Proposed |
| Authentication | Application-managed identity and secure session design | To be specified |
| Background processing | Queue/worker pattern where justified | To be evaluated |
| Observability | Structured logs, metrics and alerts | To be selected |

These are architectural candidates only. No RSS OmniPOS production resources or bindings are created by this document. Existing RSS infrastructure must be inspected before any resource is requested.

## API and integration approach

- Version APIs and define explicit request/response contracts.
- Keep client applications independent of internal database schemas.
- Use server-side authorization for every protected operation.
- Use idempotency for retryable commands and payment/sync submissions.
- Isolate external payment, email, tax and peripheral integrations behind adapters.
- Store secrets in an approved secret-management facility, never in source control.
- Avoid exposing internal API hostnames or credentials in public UI.

## Data and local storage

- The server-side relational database is the authoritative system of record.
- Clients may keep only the minimum local data required for approved offline workflows.
- Sensitive local data must be encrypted and scoped to the signed-in business, user and device.
- Local records must carry stable identifiers, schema versions and synchronization state.
- Backups and exports must use explicit retention, encryption and access controls.

## Development and quality tooling

Proposed practices:
- Version-controlled schema migrations
- Automated formatting, linting and unit tests
- API contract and integration tests
- Dependency and static security analysis
- Reproducible, reviewed build processes
- Separate development, test, staging and production configuration
- Release provenance and signed artifacts where technically supported

Exact tools and supported versions remain open.

## Decisions required before implementation

1. Confirm Windows UI framework and supported Windows versions.
2. Confirm Android minimum SDK, target SDK policy and supported hardware.
3. Confirm backend API framework and runtime constraints.
4. Confirm database provider, region, connection and migration strategy after infrastructure inspection.
5. Confirm authentication/session architecture and account recovery.
6. Confirm peripheral and payment-provider compatibility requirements.
7. Confirm build, signing, update and release distribution strategy.
8. Review total operating cost and vendor lock-in before committing to paid services.
