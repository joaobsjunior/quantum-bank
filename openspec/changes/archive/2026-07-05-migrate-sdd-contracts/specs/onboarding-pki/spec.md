## ADDED Requirements

### Requirement: OTK binding fields and lifetime
An OTK SHALL be a short-lived, single-use, opaque backend record bound to
`oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`, with a
`csrFingerprint` stored on consumption, a default local-v1 TTL of 5 minutes
(overridable by configuration), and SHALL NOT embed secret key material.

#### Scenario: OTK is issued after authentication
- **WHEN** the backend issues an OTK for an authenticated subject
- **THEN** the OTK record is bound to `oauth2Subject`, `appInstanceId`,
  `deviceId`, and `certificateProfile`
- **AND** the client-facing value is opaque and contains no secret key material

#### Scenario: OTK exceeds its TTL
- **WHEN** the configured TTL elapses before a valid CSR is accepted
- **THEN** the OTK can no longer be consumed

### Requirement: OTK state machine
Each OTK SHALL hold exactly one state and follow the allowed transitions:
`issued` to `consumed` on successful CSR validation, `issued` to `expired` on
TTL elapse, `issued` to `revoked` on policy invalidation, `issued` to
`rejected_subject_mismatch` or `rejected_device_mismatch` on binding mismatch,
and any terminal state to `replayed` on re-presentation; `consumed`, `expired`,
`replayed`, `revoked`, `rejected_subject_mismatch`, and `rejected_device_mismatch`
are terminal.

#### Scenario: One-use consumption is atomic
- **WHEN** two concurrent CSR submissions present the same `issued` OTK
- **THEN** at most one submission transitions the OTK to `consumed`
- **AND** the other is treated as `replayed`

#### Scenario: Terminal OTK is re-presented
- **WHEN** an OTK already in a terminal state is presented again
- **THEN** the submission is rejected as `replayed`

### Requirement: CSR submission is validated before PKI handoff
On `POST /auth/csr` the backend SHALL, before any PKI handoff, resolve the OTK,
confirm it is `issued` and within TTL, confirm the authenticated `oauth2Subject`
(derived from the JWT) and the submitted `appInstanceId`, `deviceId`, and
`certificateProfile` match the binding, parse the CSR with a standards-based
parser, confirm subject/SAN policy, reject any private key material, compute and
persist `csrFingerprint`, and atomically consume the OTK.

#### Scenario: Valid CSR is consumed
- **WHEN** a CSR submission matches the OTK binding, parses correctly, and
  carries no private key material
- **THEN** the backend consumes the OTK and hands the CSR to the PKI layer

#### Scenario: Private key material is submitted
- **WHEN** a submission includes private key material or a field that appears to
  carry a private key
- **THEN** the backend rejects the request before PKI handoff

#### Scenario: Binding mismatch
- **WHEN** the submitted subject, app, device, or profile does not match the OTK
  binding
- **THEN** the backend rejects the submission without issuing a certificate

### Requirement: Bootstrap audit events exclude secrets
The backend SHALL emit audit events for OTK and CSR state changes (`otk.issued`,
`otk.consumed`, `otk.expired`, `otk.replayed`, `otk.revoked`,
`otk.rejected_subject_mismatch`, `otk.rejected_device_mismatch`,
`csr.rejected_private_key_material`, `csr.accepted_for_pki_handoff`) that record
OTK id, subject, app/device, profile, resulting state, timestamp, and
correlation id, and SHALL NOT include token values, CSR PEM, or private keys.

#### Scenario: OTK is consumed
- **WHEN** an OTK transitions to `consumed`
- **THEN** an `otk.consumed` audit event is emitted with the binding metadata and
  correlation id
- **AND** the event contains no token value, CSR PEM, or private key material

### Requirement: Stable OTK and CSR error codes
Backend OTK/CSR failures SHALL use `application/problem+json` with stable
`errorCode` values and HTTP statuses: `otk_not_found` (404), `otk_expired` (409),
`otk_replayed` (409), `otk_revoked` (409), `subject_mismatch` (400),
`device_mismatch` (400), `certificate_profile_mismatch` (400),
`unsupported_environment` (400), `csr_invalid` (400), `private_key_rejected`
(400), and `pki_handoff_failed` (502), each safe for mobile display.

#### Scenario: Expired OTK is submitted
- **WHEN** a CSR is submitted with an OTK past its TTL
- **THEN** the response is `409` with `errorCode` `otk_expired`
- **AND** it leaks no token value, CSR internals, or CA implementation detail

### Requirement: PKI owns certificate profiles and issuance
The PKI layer SHALL own the certificate lifecycle for the local-v1 mobile client
profile `quantum-bank-mobile-client-v1`, accept CSR intake only after backend
OTK validation (with CSR content, `oauth2Subject`, `appInstanceId`, `deviceId`,
`environment`, profile, `csrFingerprint`, and correlation id), enforce
environment separation, issue a certificate carrying that identity and lifecycle
metadata, and SHALL NOT receive, store, or reconstruct the mobile private key.
KrakenD and the backend SHALL NOT act as certificate authorities.

#### Scenario: Certificate is issued for a validated CSR
- **WHEN** the PKI layer receives a validated CSR intake for
  `quantum-bank-mobile-client-v1`
- **THEN** it issues a client certificate bound to the validated subject, app,
  device, and environment with issuance and expiration metadata
- **AND** it never receives the mobile private key

#### Scenario: Certificate is used across environments
- **WHEN** a certificate issued for one `environment` is presented in another
- **THEN** it is not trusted as a client certificate in that other environment

### Requirement: Local CA adapter with OpenXPKI swap-in boundary
Local v1 SHALL provide a PKI-owned local CA adapter (OpenSSL scripts under
`pki/scripts/` and profile material under `pki/local-ca/`) invoked only after OTK
and CSR validation, exposing a stable adapter request of CSR, `oauth2Subject`,
`appInstanceId`, `deviceId`, `environment`, `certificateProfile`,
`csrFingerprint`, and backend correlation id, so OpenXPKI (or another approved
open-source PKI) can replace the implementation without changing mobile, gateway,
or backend contracts.

#### Scenario: Backend invokes the local adapter
- **WHEN** the backend hands off a validated CSR in local v1
- **THEN** it invokes the PKI-owned adapter with the stable request fields
- **AND** neither the gateway nor the backend signs the certificate itself

### Requirement: Revocation is represented and enforceable
The PKI layer SHALL represent revocation as a local serial denylist
(`pki/local-ca/revoked-serials.txt`, one uppercase hex serial per non-comment
line) writable via `scripts/revoke-local-cert.sh`, and SHALL support
invalidating a certificate by certificate identifier, `oauth2Subject`,
`appInstanceId`, or `deviceId`.

#### Scenario: A certificate serial is revoked
- **WHEN** `scripts/revoke-local-cert.sh` is run for a certificate serial
- **THEN** the serial is appended to the PKI-owned revocation denylist

### Requirement: Trust anchors are PKI-owned public outputs
The PKI layer SHALL publish public-only trust anchors (`root-ca.crt`,
`issuing-ca.crt`) bound to an environment for gateway consumption, and SHALL NOT
source-control private CA keys, issued private keys, CSRs, or serial state.

#### Scenario: Gateway consumes trust material
- **WHEN** the gateway is configured to enforce mobile client certificates
- **THEN** it consumes the PKI-published trust anchors
- **AND** the trust anchors contain certificates only, never private keys

### Requirement: Mobile generates key material and models certificate-ready state
The mobile app SHALL generate its client keypair at runtime, keep the private
key on-device (never sent to backend, gateway, or PKI, never committed, bundled,
logged, or exported), generate a CSR carrying the same identity inputs, use a
fail-closed TLS client that never accepts all certificates, and model bootstrap
state as one of `missing`, `ready`, `expired`, `untrusted`, `csrRejected`,
`otkExpired`, or `otkReplayed`.

#### Scenario: Runtime keypair and CSR
- **WHEN** the app begins certificate bootstrap after OAuth2 authentication
- **THEN** it generates the keypair at runtime and a CSR bound to
  `oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile`
- **AND** the private key never leaves the device and is not logged or exported

#### Scenario: Untrusted gateway certificate
- **WHEN** the gateway presents an untrusted certificate
- **THEN** the mobile TLS client rejects the connection without a permissive
  certificate callback
- **AND** the app models the state as `untrusted`

#### Scenario: Bootstrap failure states
- **WHEN** onboarding fails due to OTK expiry, OTK replay, or CSR rejection
- **THEN** the app models the corresponding `otkExpired`, `otkReplayed`, or
  `csrRejected` state without exposing token values or CSR internals
