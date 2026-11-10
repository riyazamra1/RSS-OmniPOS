# Security policy

## Reporting a vulnerability

Do not publish exploitable vulnerability details, credentials, customer information or private security reports in public issues or discussions.

Use GitHub's private vulnerability reporting feature for this repository when available. If it is unavailable, contact Razeen Secure Solution through an independently verified official security contact.

Include:
- A concise description and affected component
- Reproduction steps and prerequisites
- Potential impact
- Any suggested mitigation

Do not access, modify, delete or exfiltrate data belonging to other users or businesses while investigating.

## Repository handling

Never commit:
- API keys, passwords, tokens or private keys
- Production database dumps or customer data
- Plaintext proprietary source or protected assets
- Signing certificates or release credentials
- Decrypted build outputs or sensitive logs

Report accidental exposure immediately so credentials can be revoked or rotated. Removing a secret from the latest commit does not invalidate a credential that may already have been exposed.

## Security status

This file describes reporting expectations. It is not a claim that the application has completed a security audit or that production security controls are deployed.
