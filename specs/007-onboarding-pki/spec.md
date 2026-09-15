# Feature Specification: Onboarding PKI

**Feature Branch**: `007-onboarding-pki`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `onboarding-pki`

## Overview

Define onboarding and certificate lifecycle behavior for OTK, CSR-driven client
certificate issuance, mTLS readiness, and PKI responsibilities.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - OTK starts device onboarding (Priority: P1)

The app SHALL use a one-time-key onboarding flow before provisioning runtime client certificate material.

**Acceptance Scenarios**:

1. **When** the app submits a valid one-time key during onboarding, **Then** the backend accepts the onboarding attempt, **And** the app can proceed to CSR generation and certificate provisioning
2. **When** the app submits an invalid, expired, or reused one-time key, **Then** onboarding is rejected, **And** no client certificate is issued

---

### User Story 2 - CSR-driven client certificate issuance (Priority: P1)

The app SHALL generate or provide a certificate signing request during onboarding so the PKI layer can issue a client certificate for mTLS.

**Acceptance Scenarios**:

1. **When** onboarding is approved and the app submits a valid CSR, **Then** the PKI layer issues a client certificate bound to the approved onboarding context
2. **When** the CSR is malformed or does not match the onboarding context, **Then** certificate issuance is rejected

---

### User Story 3 - PKI lifecycle is explicit (Priority: P1)

The project SHALL include certificate lifecycle capability for issuance, renewal, and revocation.

**Acceptance Scenarios**:

1. **When** KrakenD cannot satisfy runtime certificate issuance, renewal, and revocation requirements, **Then** an open source PKI such as OpenXPKI is used for lifecycle workflows
2. **When** a client certificate is revoked, **Then** subsequent mTLS attempts with that certificate are rejected after revocation state is enforced

---

### User Story 4 - OTK binding fields and lifetime (Priority: P1)

An OTK SHALL be a short-lived, single-use, opaque backend record bound to
`oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`, with a
`csrFingerprint` stored on consumption, a default local-v1 TTL of 5 minutes
(overridable by configuration), and SHALL NOT embed secret key material.

**Acceptance Scenarios**:

1. **When** the backend issues an OTK for an authenticated subject, **Then** the OTK record is bound to `oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`, **And** the client-facing value is opaque and contains no secret key material
2. **When** the configured TTL elapses before a valid CSR is accepted, **Then** the OTK can no longer be consumed

---

### User Story 5 - OTK state machine (Priority: P1)

Each OTK SHALL hold exactly one state and follow the allowed transitions:
`issued` to `consumed` on successful CSR validation, `issued` to `expired` on
TTL elapse, `issued` to `revoked` on policy invalidation, `issued` to
`rejected_subject_mismatch` or `rejected_device_mismatch` on binding mismatch,
and any terminal state to `replayed` on re-presentation; `consumed`, `expired`,
`replayed`, `revoked`, `rejected_subject_mismatch`, and `rejected_device_mismatch`
are terminal.

**Acceptance Scenarios**:

1. **When** two concurrent CSR submissions present the same `issued` OTK, **Then** at most one submission transitions the OTK to `consumed`, **And** the other is treated as `replayed`
2. **When** an OTK already in a terminal state is presented again, **Then** the submission is rejected as `replayed`

---

### User Story 6 - CSR submission is validated before PKI handoff (Priority: P1)

On `POST /auth/csr` the backend SHALL, before any PKI handoff, resolve the OTK,
confirm it is `issued` and within TTL, confirm the authenticated `oauth2Subject`
(derived from the JWT) and the submitted `appInstanceId`, `deviceId`, and
`certificateProfile` match the binding, parse the CSR with a standards-based
parser, confirm subject/SAN policy, reject any private key material, compute and
persist `csrFingerprint`, and atomically consume the OTK.

**Acceptance Scenarios**:

1. **When** a CSR submission matches the OTK binding, parses correctly, and carries no private key material, **Then** the backend consumes the OTK and hands the CSR to the PKI layer
2. **When** a submission includes private key material or a field that appears to carry a private key, **Then** the backend rejects the request before PKI handoff
3. **When** the submitted subject, app, device, or profile does not match the OTK binding, **Then** the backend rejects the submission without issuing a certificate

---

### User Story 7 - Bootstrap audit events exclude secrets (Priority: P1)

The backend SHALL emit audit events for OTK and CSR state changes (`otk.issued`,
`otk.consumed`, `otk.expired`, `otk.replayed`, `otk.revoked`,
`otk.rejected_subject_mismatch`, `otk.rejected_device_mismatch`,
`csr.rejected_private_key_material`, `csr.accepted_for_pki_handoff`) that record
OTK id, subject, app/device, profile, resulting state, timestamp, and
correlation id, and SHALL NOT include token values, CSR PEM, or private keys.

**Acceptance Scenarios**:

1. **When** an OTK transitions to `consumed`, **Then** an `otk.consumed` audit event is emitted with the binding metadata and correlation id, **And** the event contains no token value, CSR PEM, or private key material

---

### User Story 8 - Stable OTK and CSR error codes (Priority: P1)

Backend OTK/CSR failures SHALL use `application/problem+json` with stable
`errorCode` values and HTTP statuses: `otk_not_found` (404), `otk_expired` (409),
`otk_replayed` (409), `otk_revoked` (409), `subject_mismatch` (400),
`device_mismatch` (400), `certificate_profile_mismatch` (400),
`unsupported_environment` (400), `csr_invalid` (400), `private_key_rejected`
(400), and `pki_handoff_failed` (502), each safe for mobile display.

**Acceptance Scenarios**:

1. **When** a CSR is submitted with an OTK past its TTL, **Then** the response is `409` with `errorCode` `otk_expired`, **And** it leaks no token value, CSR internals, or CA implementation detail

---

### User Story 9 - PKI owns certificate profiles and issuance (Priority: P1)

The PKI layer SHALL own the certificate lifecycle for the local-v1 mobile client
profile `quantum-bank-mobile-client-v1`, accept CSR intake only after backend
OTK validation (with CSR content, `oauth2Subject`, `appInstanceId`, `deviceId`,
`environment`, profile, `csrFingerprint`, and correlation id), enforce
environment separation, issue a certificate carrying that identity and lifecycle
metadata, and SHALL NOT receive, store, or reconstruct the mobile private key.
KrakenD and the backend SHALL NOT act as certificate authorities.

**Acceptance Scenarios**:

1. **When** the PKI layer receives a validated CSR intake for `quantum-bank-mobile-client-v1`, **Then** it issues a client certificate bound to the validated subject, app, device, and environment with issuance and expiration metadata, **And** it never receives the mobile private key
2. **When** a certificate issued for one `environment` is presented in another, **Then** it is not trusted as a client certificate in that other environment

---

### User Story 10 - Local CA adapter with OpenXPKI swap-in boundary (Priority: P1)

Local v1 SHALL provide a PKI-owned local CA adapter (OpenSSL scripts under
`pki/scripts/` and profile material under `pki/local-ca/`) invoked only after OTK
and CSR validation, exposing a stable adapter request of CSR, `oauth2Subject`,
`appInstanceId`, `deviceId`, `environment`, `certificateProfile`,
`csrFingerprint`, and backend correlation id, so OpenXPKI (or another approved
open-source PKI) can replace the implementation without changing mobile, gateway,
or backend contracts.

**Acceptance Scenarios**:

1. **When** the backend hands off a validated CSR in local v1, **Then** it invokes the PKI-owned adapter with the stable request fields, **And** neither the gateway nor the backend signs the certificate itself

---

### User Story 11 - Revocation is represented and enforceable (Priority: P1)

The PKI layer SHALL represent revocation as a local serial denylist
(`pki/local-ca/revoked-serials.txt`, one uppercase hex serial per non-comment
line) writable via `scripts/revoke-local-cert.sh`, and SHALL support
invalidating a certificate by certificate identifier, `oauth2Subject`,
`appInstanceId`, or `deviceId`.

**Acceptance Scenarios**:

1. **When** `scripts/revoke-local-cert.sh` is run for a certificate serial, **Then** the serial is appended to the PKI-owned revocation denylist

---

### User Story 12 - Trust anchors are PKI-owned public outputs (Priority: P1)

The PKI layer SHALL publish public-only trust anchors (`root-ca.crt`,
`issuing-ca.crt`) bound to an environment for gateway consumption, and SHALL NOT
source-control private CA keys, issued private keys, CSRs, or serial state.

**Acceptance Scenarios**:

1. **When** the gateway is configured to enforce mobile client certificates, **Then** it consumes the PKI-published trust anchors, **And** the trust anchors contain certificates only, never private keys

---

### User Story 13 - Mobile generates key material and models certificate-ready state (Priority: P1)

The mobile app SHALL generate its client keypair at runtime, keep the private
key on-device (never sent to backend, gateway, or PKI, never committed, bundled,
logged, or exported), generate a CSR carrying the same identity inputs, use a
fail-closed TLS client that never accepts all certificates, and model bootstrap
state as one of `missing`, `ready`, `expired`, `untrusted`, `csrRejected`,
`otkExpired`, or `otkReplayed`.

**Acceptance Scenarios**:

1. **When** the app begins certificate bootstrap after OAuth2 authentication, **Then** it generates the keypair at runtime and a CSR bound to `oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`, **And** the private key never leaves the device and is not logged or exported
2. **When** the gateway presents an untrusted certificate, **Then** the mobile TLS client rejects the connection without a permissive certificate callback, **And** the app models the state as `untrusted`
3. **When** onboarding fails due to OTK expiry, OTK replay, or CSR rejection, **Then** the app models the corresponding `otkExpired`, `otkReplayed`, or `csrRejected` state without exposing token values or CSR internals

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The app SHALL use a one-time-key onboarding flow before provisioning runtime client certificate material.
- **FR-002**: The app SHALL generate or provide a certificate signing request during onboarding so the PKI layer can issue a client certificate for mTLS.
- **FR-003**: The project SHALL include certificate lifecycle capability for issuance, renewal, and revocation.
- **FR-004**: An OTK SHALL be a short-lived, single-use, opaque backend record bound to `oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`, with a `csrFingerprint` stored on consumption, a default local-v1 TTL of 5 minutes (overridable by configuration), and SHALL NOT embed secret key material.
- **FR-005**: Each OTK SHALL hold exactly one state and follow the allowed transitions: `issued` to `consumed` on successful CSR validation, `issued` to `expired` on TTL elapse, `issued` to `revoked` on policy invalidation, `issued` to `rejected_subject_mismatch` or `rejected_device_mismatch` on binding mismatch, and any terminal state to `replayed` on re-presentation; `consumed`, `expired`, `replayed`, `revoked`, `rejected_subject_mismatch`, and `rejected_device_mismatch` are terminal.
- **FR-006**: On `POST /auth/csr` the backend SHALL, before any PKI handoff, resolve the OTK, confirm it is `issued` and within TTL, confirm the authenticated `oauth2Subject` (derived from the JWT) and the submitted `appInstanceId`, `deviceId`, and `certificateProfile` match the binding, parse the CSR with a standards-based parser, confirm subject/SAN policy, reject any private key material, compute and persist `csrFingerprint`, and atomically consume the OTK.
- **FR-007**: The backend SHALL emit audit events for OTK and CSR state changes (`otk.issued`, `otk.consumed`, `otk.expired`, `otk.replayed`, `otk.revoked`, `otk.rejected_subject_mismatch`, `otk.rejected_device_mismatch`, `csr.rejected_private_key_material`, `csr.accepted_for_pki_handoff`) that record OTK id, subject, app/device, profile, resulting state, timestamp, and correlation id, and SHALL NOT include token values, CSR PEM, or private keys.
- **FR-008**: Backend OTK/CSR failures SHALL use `application/problem+json` with stable `errorCode` values and HTTP statuses: `otk_not_found` (404), `otk_expired` (409), `otk_replayed` (409), `otk_revoked` (409), `subject_mismatch` (400), `device_mismatch` (400), `certificate_profile_mismatch` (400), `unsupported_environment` (400), `csr_invalid` (400), `private_key_rejected` (400), and `pki_handoff_failed` (502), each safe for mobile display.
- **FR-009**: The PKI layer SHALL own the certificate lifecycle for the local-v1 mobile client profile `quantum-bank-mobile-client-v1`, accept CSR intake only after backend OTK validation (with CSR content, `oauth2Subject`, `appInstanceId`, `deviceId`, `environment`, profile, `csrFingerprint`, and correlation id), enforce environment separation, issue a certificate carrying that identity and lifecycle metadata, and SHALL NOT receive, store, or reconstruct the mobile private key. KrakenD and the backend SHALL NOT act as certificate authorities.
- **FR-010**: Local v1 SHALL provide a PKI-owned local CA adapter (OpenSSL scripts under `pki/scripts/` and profile material under `pki/local-ca/`) invoked only after OTK and CSR validation, exposing a stable adapter request of CSR, `oauth2Subject`, `appInstanceId`, `deviceId`, `environment`, `certificateProfile`, `csrFingerprint`, and backend correlation id, so OpenXPKI (or another approved open-source PKI) can replace the implementation without changing mobile, gateway, or backend contracts.
- **FR-011**: The PKI layer SHALL represent revocation as a local serial denylist (`pki/local-ca/revoked-serials.txt`, one uppercase hex serial per non-comment line) writable via `scripts/revoke-local-cert.sh`, and SHALL support invalidating a certificate by certificate identifier, `oauth2Subject`, `appInstanceId`, or `deviceId`.
- **FR-012**: The PKI layer SHALL publish public-only trust anchors (`root-ca.crt`, `issuing-ca.crt`) bound to an environment for gateway consumption, and SHALL NOT source-control private CA keys, issued private keys, CSRs, or serial state.
- **FR-013**: The mobile app SHALL generate its client keypair at runtime, keep the private key on-device (never sent to backend, gateway, or PKI, never committed, bundled, logged, or exported), generate a CSR carrying the same identity inputs, use a fail-closed TLS client that never accepts all certificates, and model bootstrap state as one of `missing`, `ready`, `expired`, `untrusted`, `csrRejected`, `otkExpired`, or `otkReplayed`.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
