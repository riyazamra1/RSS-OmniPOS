# RSS OmniPOS — Deployment Architecture (Draft)

**Status:** Deployment plan only. No production website, API, database, storage or release environment is configured by this document.

## Deployment objectives

- Complete and validate RSS OmniPOS using the existing RSS website path first.
- Keep public, administrator and future customer portal access policies separate.
- Make hostnames, base paths, API endpoints and callback URLs environment-configurable.
- Avoid hardcoding temporary domain details into business logic.
- Use separate development, test, staging and production configuration.
- Require explicit approval before creating infrastructure, purchasing domains or deploying.
- Verify both technical health and relevant user workflows after each deployment.

## Current website plan

| Purpose | Current address | State |
|---|---|---|
| Temporary public website | https://rssapps.net/accounts/OmniPOS | User-designated current website |
| Administrator portal | https://rssapps.net/accounts/OmniPOS/admin | User-designated private administration route |
| Future customer domain | Not yet purchased | Deferred until system completion |

The public website and admin portal may initially share the same host and path prefix, but must have distinct route authorization, session policies and backend permissions. A route name is not a security boundary.

## Future domain migration

A separate customer domain will be purchased and configured later. Do not purchase or register it during the current completion phase.

Prepare for migration by:
- Centralizing public base URL and API base URL in environment configuration.
- Keeping canonical business IDs independent of hostnames and URL paths.
- Configuring allowed origins, CORS, cookie domains, OAuth callbacks, email links and redirect URIs per environment.
- Avoiding absolute temporary-domain URLs in persisted business records.
- Supporting controlled redirects only after the future domain is approved and configured.
- Testing login, password recovery, deep links, customer portals and admin access after migration.
- Maintaining the existing URL until the new domain is verified and migration is explicitly approved.

## Proposed environment separation

### Development
Local development with synthetic/test data and no production credentials.

### Test
Automated tests, integration tests and disposable test data. No customer production data.

### Staging
Production-like configuration with restricted access and test accounts. Use only approved non-production credentials and services.

### Production
Customer-facing service with approved domains, monitored infrastructure, backups, access controls, recovery procedures and release approval.

Exact hosting, environment names, regions and infrastructure products remain subject to inspection and approval.

## Build and release pipeline

- Validate documentation and repository hygiene on pull requests and pushes.
- Add language-specific lint, test and security checks when application source is introduced.
- Build from reviewed source and pinned dependencies/actions.
- Do not expose decryption secrets to untrusted pull requests.
- Keep signing and production credentials in protected secret storage.
- Separate build artifacts from deployment actions.
- Require approval for production deployment.
- Record commit SHA, artifact identity, workflow run, deployment target and result.
- Never claim deployment success from a successful build alone.

## Configuration and secrets

- Keep environment-specific configuration outside source code.
- Do not commit .env files, credentials, private keys or production data.
- Use least-privilege service credentials and rotate them under an approved process.
- Keep public configuration separate from secrets.
- Restrict production secrets to trusted workflows and environments.
- Avoid printing sensitive values in logs or build output.

## Database migration and backup

- Version all schema migrations.
- Test migrations against representative non-production data.
- Define backup frequency, retention, encryption and access control.
- Define recovery point objective (RPO) and recovery time objective (RTO).
- Perform restore tests and record evidence.
- Plan rollback/forward-fix behavior for schema and application releases.
- Never use a successful backup job as proof of recoverability without restore verification.

## Monitoring and operations

Plan for:
- Application/API availability and latency
- Authentication failures and suspicious access
- Payment and webhook processing outcomes
- Queue age and offline synchronization conflicts
- Database health and connection saturation
- Storage capacity and backup status
- Error rates, release regressions and audit events

Logs must avoid passwords, tokens, payment secrets and unnecessary personal/customer data.

## Deployment verification checklist

Before declaring a release successful:
1. Confirm the intended commit and artifact.
2. Confirm the deployment target and environment.
3. Check service health and critical API endpoints.
4. Test public website routing.
5. Test admin authentication and backend authorization.
6. Test business-critical workflows relevant to the release.
7. Verify database migration and backup state.
8. Check logs and monitoring for new errors.
9. Record evidence and rollback/recovery option.

## Decisions still open

- Hosting products, regions and exact service topology
- Domain/DNS and TLS configuration
- Environment and release approval model
- RPO/RTO and backup retention
- Monitoring provider and alert thresholds
- Rollback strategy and maintenance windows
- Future customer domain name and migration schedule
