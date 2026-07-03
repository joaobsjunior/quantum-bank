## Why

Secure mTLS requires each device to hold a client certificate bound to an
approved onboarding. Quantum Bank needs an onboarding flow that starts from a
one-time key (OTK), drives a CSR-based certificate issuance, and relies on a
real PKI for the certificate lifecycle (issuance, renewal, revocation) rather
than assuming the gateway can do it all.

## What Changes

- Introduce **OTK onboarding**: a one-time key gates device onboarding before any
  runtime client certificate material is provisioned; invalid/expired/reused
  keys are rejected with no certificate issued.
- Introduce **CSR-driven issuance**: the app submits a CSR during onboarding so
  the PKI layer issues a client certificate bound to the approved context;
  malformed or mismatched CSRs are rejected.
- Make the **PKI lifecycle explicit**: when KrakenD cannot satisfy runtime
  issuance/renewal/revocation, an open-source PKI (e.g. OpenXPKI) owns those
  workflows, and revoked certificates are rejected on subsequent mTLS attempts.

## Impact

- **pki**: local CA + certificate lifecycle workflows and trust-anchor material.
- **backend**: OTK validation and onboarding approval endpoints.
- **mobile-app**: OTK submission, CSR generation, and certificate provisioning.
- **Security posture**: certificate provisioning is gated and revocable.
