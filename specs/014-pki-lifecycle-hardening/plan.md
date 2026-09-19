# Implementation Plan: PKI Lifecycle and Cloud Passthrough Hardening

**Branch**: `014-pki-lifecycle-hardening` | **Date**: 2026-09-19 | **Spec**: [spec.md](spec.md)

## Summary

Add CRL-based revocation for both PKI chains (published with the runtime
material, checked by the banking terminator and the backend), scripted renewal
with terminator reload, automatic device renewal in the app, an operator
device-revocation endpoint, a Terraform gate against TLS-terminating ingress,
and bounded refresh tokens.

## Technical Context

**Language/Version**: Bash + OpenSSL ≥ 3.5 (`pki`), HAProxy 3.2 config
(`api-gateway`), Kotlin/Spring Boot (`backend`), Dart (`mobile-app`),
Terraform + Bash (`infrastructure`), Keycloak realm JSON

**Primary Dependencies**: OpenSSL `ca -gencrl` with ML-DSA-87 / ECDSA P-384
CA keys; HAProxy `crl-file`; BCJSSE revocation (`jdk.security.certpath`
properties or an explicit `PKIXRevocationChecker`); Keycloak client
settings

**Storage**: `pki/local-ca/runtime/*.crl`; backend `device_signing_keys`
delete; H2 unchanged

**Testing**: `pki/scripts/ci-validate.sh` (CRL generation and signature
algorithm), `negative-mtls-tests` (revoked identities refused),
`verify-terraform-config.sh` negative fixtures, backend and mobile unit tests
at 100%

**Constraints**: CRLs signed with ML-DSA-87 are large; hourly cadence; fail
closed on the strict tier when the CRL is stale

## Constitution Check

| Principle | Status | Notes |
| --- | --- | --- |
| I. Gateway única rota | PASS | Revocation endpoint is backend-only, operator scope, not app-facing |
| II. TLS sem atalhos | PASS | Adds revocation checks; no permissive path |
| III. Segurança antes da tela | PASS | |
| IV. Pix simulação | PASS | |
| V. Nuvem explícita | PASS | Per-cloud passthrough checks, no generic module |

## Project Structure

```text
pki/scripts/
├── revoke-local-cert.sh        # regenerate CRLs for the chain of the revoked cert
├── gen-crls.sh                 # CRL per issuing CA, union bundle for the banking bind
└── renew-runtime-certs.sh      # re-issue near-expiry identities, keep old until reload

api-gateway/tls/
└── haproxy-banking.cfg         # bind ... crl-file /etc/quantum-bank/tls/ca-chain-all.crl

backend/src/main/kotlin/.../
├── security/PostQuantumTls.kt  # revocation checking for the truststore
└── bootstrap/DeviceRevocation* # DELETE /auth/devices/{deviceId} (operator scope)

mobile-app/lib/features/bootstrap/
└── renewal_policy.dart         # re-enroll under 20% remaining validity

infrastructure/
├── scripts/verify-terraform-config.sh   # passthrough gate per cloud
└── keycloak/quantum-bank-local-realm.json  # mobile client refresh token limits
```

**Structure Decision**: revocation and renewal stay in `pki` scripts (the
lifecycle owner), consumers only add checks; OpenXPKI remains the escalation
if the scripts cannot meet the cadence in production.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| CRL distribution to terminators | Cloud terminators need the CRL without shared disk | OCSP would add an online responder to operate; CRLs reuse the existing material sync |
