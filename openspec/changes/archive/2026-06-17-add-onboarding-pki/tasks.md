## 1. OTK onboarding

- [x] 1.1 Implement one-time-key issuance and validation in the backend
- [x] 1.2 Reject invalid, expired, or reused keys without issuing a certificate
- [x] 1.3 Gate CSR/provisioning behind an approved OTK onboarding

## 2. CSR-driven issuance

- [x] 2.1 Generate a CSR in the mobile app during onboarding
- [x] 2.2 Issue a client certificate bound to the approved onboarding context
- [x] 2.3 Reject malformed or context-mismatched CSRs

## 3. PKI lifecycle

- [x] 3.1 Stand up an open-source PKI for issuance/renewal/revocation workflows
- [x] 3.2 Enforce revocation so revoked certs fail subsequent mTLS attempts
- [x] 3.3 Provide trust-anchor material and verify-trust-anchors tooling
