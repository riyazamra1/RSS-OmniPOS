# RSS OmniPOS — Repository and Build Security (Draft)

**Status:** Security requirements for design review. No encryption or protected build pipeline is implemented by this document.

## Protection objectives

- Keep proprietary source and assets out of public plaintext repository history.
- Keep credentials and encryption keys outside Git.
- Restrict who and what can decrypt protected material.
- Prevent untrusted pull requests from accessing decryption secrets.
- Minimize plaintext lifetime and exposure in build environments.
- Prevent sensitive files, logs and intermediate outputs from entering public artifacts.

## Required design decisions before encrypted source is added

1. Select an authenticated encryption format and supported tooling.
2. Define a key-encryption-key and data-encryption-key hierarchy.
3. Define key custody, access approvals, rotation and revocation.
4. Define recovery and disaster-recovery procedures.
5. Define encrypted-file manifest format, integrity checks and versioning.
6. Define trusted branch, tag and environment policies for decryption.
7. Define runner isolation, temporary storage and cleanup.
8. Define artifact scanning and release signing.
9. Threat-model maintainers, compromised credentials, malicious pull requests and compromised runners.

## CI/CD requirements

- Use least-privilege workflow permissions.
- Pin third-party actions to reviewed immutable commit SHAs.
- Do not expose secrets to forked pull requests or untrusted branches.
- Do not print secrets, decrypted source or sensitive environment values.
- Do not upload decrypted source as an artifact or cache.
- Separate validation, protected build and release publication.
- Require explicit approval for production release and deployment.
- Retain build provenance and verification results without retaining plaintext source.

## Important limitation

Encryption at rest does not prevent an authorized or compromised build runner from accessing plaintext during compilation. The runner and its credentials are part of the trusted computing base and must be protected accordingly.

## Current state

No encryption keys, encrypted source bundles, decryption scripts, release signing keys or protected build workflows are configured by this document.
