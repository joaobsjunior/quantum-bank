---

description: "Task list for PKI lifecycle and cloud passthrough hardening"
---

# Tasks: PKI Lifecycle and Cloud Passthrough Hardening

**Input**: Design documents from `/specs/014-pki-lifecycle-hardening/`

**Status**: not started.

## Phase 1: Revocation (US1)

- [ ] T001 `pki/scripts/gen-crls.sh`: CRL per issuing CA (ML-DSA-87 / ECDSA P-384), `ca-chain-all.crl` union; wired into `bootstrap-runtime-certs.sh` and `revoke-local-cert.sh`
- [ ] T002 `pki/scripts/ci-validate.sh`: CRL present, signature algorithm per chain, `nextUpdate` within cadence
- [ ] T003 `api-gateway/tls/haproxy-banking.cfg`: `crl-file` on the banking bind; `verify-pqc-gateway.sh` enforces it
- [ ] T004 `backend/.../security/PostQuantumTls.kt`: revocation checking for gateway identities; test with a revoked identity in `PostQuantumTlsTest`
- [ ] T005 `backend`: `DELETE /auth/devices/{deviceId}` (operator scope), revokes the certificate through the PKI script and deletes the signing key; audit event
- [ ] T006 `infrastructure/scripts/negative-mtls-tests.sh`: revoked device and revoked service identity refused

## Phase 2: Renewal (US2)

- [ ] T007 `pki/scripts/renew-runtime-certs.sh`: re-issue identities under 20% remaining validity; keep previous material until reload
- [ ] T008 `infrastructure/compose.yaml`: terminator reload hook (`haproxy -sf`) after renewal; smoke before/after
- [ ] T009 `mobile-app/lib/features/bootstrap/renewal_policy.dart`: automatic re-enrollment under 20% remaining validity; tests to 100%

## Phase 3: Cloud passthrough gate (US3)

- [ ] T010 `infrastructure/scripts/verify-terraform-config.sh`: fail on HTTPS listeners / managed certificates in front of the sidecar (AWS NLB TCP only, Azure TCP ingress, GCP TCP proxy); negative fixtures per cloud
- [ ] T011 `infrastructure/terraform/*/README.md`: passthrough requirement stated per cloud

## Phase 4: Tokens

- [ ] T012 `infrastructure/keycloak/quantum-bank-local-realm.json`: mobile client refresh token 30 min idle, client-bound; `verify-keycloak-config.sh` check
- [ ] T013 Docs: `docs/pqc-ml-dsa-transport.md` lifecycle section
