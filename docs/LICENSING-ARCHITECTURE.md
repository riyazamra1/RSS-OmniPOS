# RSS OmniPOS — Licensing Architecture (Draft)

**Status:** Commercial and technical behavior specification. No licence server, payment processing or entitlement enforcement is implemented by this document.

## Established commercial rules

- Full-feature evaluation trial: 14 days.
- After the trial, the customer must purchase the system.
- Purchase is a one-time system purchase, followed by mandatory annual system-only renewal.
- No lifetime licence.
- No permanent free plan.
- One device is included with each system purchase.
- Additional devices require a separate charge.
- LKR is the primary currency; USD is additionally supported.
- Proposed edition prices and user bands are provisional and are not approved by this document.

## Licence concepts

The licence model should separate:
- Business/customer account
- Trial eligibility and trial start/end timestamps
- System purchase and purchase status
- Annual renewal term and due date
- Device entitlements and registered devices
- User entitlements and purchased user band
- Enabled product modules/features
- Grace period and restricted states
- Administrative adjustments and audit events

A licence is not a user account. A business may have multiple authorized users and devices within its purchased entitlements.

## Suggested lifecycle states

- Not Registered
- Trial Active
- Trial Expired
- Purchase Pending
- Active
- Renewal Due
- Grace Period
- Suspended
- Cancelled or Terminated

Exact transition rules, payment confirmation requirements and customer notification timing must be approved before implementation.

## Trial behavior

- Trial duration is 14 days.
- Trial provides full-feature evaluation access.
- Trial start must be recorded once for the eligible business/account under a defined policy.
- Reinstalling an application or changing a device must not silently reset trial eligibility.
- Trial timestamps must be determined by the authoritative server, not a device clock.
- Define how a trial converts to a paid licence and how an abandoned trial is retained or deleted.

## Purchase and annual renewal

- A purchase becomes active only after the configured payment/administrative confirmation succeeds.
- The initial purchase and annual system-only renewal must be represented as separate commercial events.
- Renewal extends the entitlement term; it does not create a lifetime entitlement.
- Failed, pending or disputed payment must not be treated as successful renewal.
- Define invoices, receipts, refunds, cancellations, grace period and suspension rules.
- Do not hardcode provisional prices or tax rates into application code.

## Device entitlements

- One device is included with each system purchase.
- Additional devices require an approved entitlement/purchase.
- Device registration must identify business, device, platform, app version and registration state.
- Support device revocation, replacement and audit history.
- Prevent simple reinstall or identifier changes from silently bypassing device limits.
- Device fingerprinting must respect platform privacy and security constraints; do not rely on invasive or unstable hardware identifiers.
- Offline device behavior and entitlement cache duration remain open decisions.

## User entitlements

User limits and bands are not finalized. The system should support configurable entitlements without assuming proposed bands or prices are approved. Define whether inactive, invited, service or administrator accounts count toward the purchased limit.

## Enforcement

- Validate licence state and feature entitlement on the backend for protected operations.
- The client may cache a signed, time-bounded entitlement snapshot for approved offline behavior.
- Never rely only on client-side licence checks.
- Record entitlement decisions and administrative overrides.
- Use server time for trial and renewal calculations.
- Ensure retries and payment webhooks are idempotent.
- Protect licence APIs against replay, tampering and cross-business access.

## Domain independence

The current temporary public website is https://rssapps.net/accounts/OmniPOS and the administrator portal is https://rssapps.net/accounts/OmniPOS/admin. A separate customer domain will be purchased after system completion.

Licence records must identify the business and entitlement, not the current website hostname. Domain changes must not reset trials, purchases, renewals or device registrations. Allowed origins, callbacks and public URLs are deployment configuration, not licence identity.

## Administrative controls

Administrative changes to trial dates, entitlements, device limits, renewal status or licence state require explicit permissions, reason capture and immutable audit history. Separate ordinary support access from authority to grant commercial entitlements.

## Decisions still open

- Exact trial eligibility and one-trial-per-business policy
- Purchase payment confirmation and refund behavior
- Renewal due date, grace period and post-grace restrictions
- Device replacement, deactivation and offline limits
- User count rules and final user bands
- Final product editions, add-ons and approved prices
- Tax/invoice handling and payment providers
- Licence transfer, business ownership change and termination policy
