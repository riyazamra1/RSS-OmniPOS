# RSS OmniPOS

RSS OmniPOS is a point-of-sale and business-management platform being developed by Razeen Secure Solution (RSS).

> **Project status:** Specification and architecture planning. This repository scaffold does not indicate that application code, production services, or deployments are complete.

## Current website locations

The system will be completed and tested on the existing RSS website before a separate customer domain is purchased.

- **Temporary public website:** https://rssapps.net/accounts/OmniPOS
- **Private administration portal:** https://rssapps.net/accounts/OmniPOS/admin
- **Future customer domain:** To be purchased and configured after the system is completed.

The public website and administration portal are distinct access surfaces. The admin path must require authenticated administrator access and backend-enforced authorization. Do not treat a hidden route or client-side navigation guard as a security boundary.

Domain names and base paths must be configurable by environment. Do not hardcode the temporary website URL into business logic, API authorization or persisted customer records. Future domain migration must not require rewriting core application logic.

## Project goals

The planned platform covers:

- Point-of-sale checkout for supported desktop and mobile clients
- Product, pricing, barcode, purchasing, supplier and inventory management
- Multi-branch and multi-warehouse operations
- Customer accounts, returns, loyalty and credit sales
- Payments, cash drawers and reconciliation
- Double-entry accounting, tax configuration and financial reporting
- Employee, attendance, payroll and sales commission management
- Fixed assets, budgets and management analytics
- Offline transaction capture and controlled synchronization
- Licensing, device entitlements and business administration

## Architecture principles

- Native Android application; do not use a WebView wrapper as a substitute for a native client.
- Enforce authorization and business rules on the backend, not only in client interfaces.
- Preserve audit history for financial, inventory, permission and administrative changes.
- Use transactional and idempotent processing for sales, payments, stock movements and synchronization.
- Keep business, branch, warehouse, terminal and user scopes explicit.
- Treat offline operations as a synchronization problem with conflict and duplicate protection.
- Keep secrets, customer data and proprietary source out of public repository content.
- Keep the public website, administrator interface and future customer portal access policies separate.

## Commercial decisions currently established

- Primary currency: LKR; USD is additionally supported.
- Evaluation period: 14-day full-feature trial.
- After the trial: one-time system purchase plus mandatory annual system-only renewal.
- One device is included with a system purchase; additional devices are charged separately.
- No lifetime licence and no permanent free plan.

Pricing amounts and other provisional commercial proposals are not approved by this statement.

## Repository security model

This repository is intended to contain public-safe documentation and, once the protection design is finalized, encrypted proprietary source and assets.

**Do not commit** plaintext proprietary source, credentials, encryption keys, customer records, production data, private certificates or unreviewed build artifacts.

Encryption format, key custody, key rotation, recovery, protected build execution and release publication must be designed and reviewed before encrypted source or decryption workflows are introduced. Encryption at rest does not protect plaintext exposed to an authorized or compromised build runner.

## Documentation

- [Project overview](docs/PROJECT-OVERVIEW.md)
- [System architecture](docs/SYSTEM-ARCHITECTURE.md)
- [Technology stack](docs/TECHNOLOGY-STACK.md)
- [Database architecture](docs/DATABASE-ARCHITECTURE.md)
- [API architecture](docs/API-ARCHITECTURE.md)
- [Offline synchronization](docs/OFFLINE-SYNC-ARCHITECTURE.md)
- [Licensing architecture](docs/LICENSING-ARCHITECTURE.md)
- [Deployment architecture](docs/DEPLOYMENT-ARCHITECTURE.md)
- [Security architecture](docs/SECURITY-ARCHITECTURE.md)
- [Repository contribution rules](CONTRIBUTING.md)
- [Security reporting](SECURITY.md)

## Current limitations

This repository scaffold is not an application release. It does not provide an executable POS, a production API, a database deployment, a licensing service or a configured CI/CD release pipeline.
