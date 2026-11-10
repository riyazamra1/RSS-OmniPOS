# Contribution and change-control rules

RSS OmniPOS is a proprietary product. Public visibility of this repository does not grant permission to copy, redistribute, reverse engineer or commercially use proprietary materials. A formal licence will be published separately after legal review.

## Before changing the project

- Keep changes scoped and explain their purpose.
- Do not add proprietary plaintext source, private assets, customer data, credentials or secrets.
- Do not create infrastructure, domains, databases, accounts or paid services as part of a code change without explicit approval.
- Do not introduce a new dependency without documenting its purpose, licence and maintenance status.
- Do not change approved commercial or product rules without recording the decision.

## Pull requests

- Describe the problem, proposed change, risks and verification performed.
- Include tests or explain why tests are not applicable.
- Never include credentials, production data or unreviewed generated artifacts.
- Do not merge changes that expose protected source or weaken authorization, audit, payment or licensing controls.

## Release discipline

A successful CI run is not proof of a production deployment. Releases and deployments require their own approval, deployment evidence and functional verification.
