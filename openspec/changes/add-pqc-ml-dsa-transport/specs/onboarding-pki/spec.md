## ADDED Requirements

### Requirement: PKI algorithm policy is post-quantum only
The local CA hierarchy SHALL use ML-DSA-87 keys and signatures, every
runtime and mobile certificate SHALL carry an ML-DSA-65 (or ML-DSA-87) subject
key signed with ML-DSA-87, and the PKI scripts SHALL require OpenSSL >= 3.5
(host binary or pinned container) and fail closed without it.

#### Scenario: Trust anchors are verified
- **WHEN** `verify-trust-anchors.sh` inspects the root and issuing certificates
- **THEN** both carry ML-DSA-87 public keys and ML-DSA-87 signatures, chain to
  each other, and match the local private keys when present

#### Scenario: Classical key material is found
- **WHEN** a bootstrap script finds a pre-existing RSA/EC key at a CA or
  runtime key path
- **THEN** it moves the key aside and generates a fresh ML-DSA key instead of
  reusing it

### Requirement: Runtime material is built without a JDK
PKCS#12 key stores and trust stores for the JVM services SHALL be built with
OpenSSL (`-jdktrust anyExtendedKeyUsage` for trust anchors), and HAProxy
terminators SHALL receive PEM bundles (certificate chain + key).

#### Scenario: Runtime certificates are generated
- **WHEN** `bootstrap-runtime-certs.sh` runs
- **THEN** every runtime leaf is ML-DSA-65 with `digitalSignature` key usage,
  PEM bundles and PKCS#12 stores exist, and `verify-runtime-certs.sh` passes

## MODIFIED Requirements

### Requirement: CSR submission is validated before PKI handoff
On `POST /auth/csr` the backend SHALL, before any PKI handoff, resolve the OTK,
confirm it is `issued` and within TTL, confirm the authenticated `oauth2Subject`
(derived from the JWT) and the submitted `appInstanceId`, `deviceId`, and
`certificateProfile` match the binding, parse the CSR with a standards-based
parser, verify the proof-of-possession signature, accept only ML-DSA-65 or
ML-DSA-87 subject public keys (rejecting RSA, EC, EdDSA and ML-DSA-44 with
`csr_key_rejected`), confirm subject/SAN policy, reject any private key
material, compute and persist `csrFingerprint`, and atomically consume the OTK;
the PKI sign script SHALL enforce the same key policy.

#### Scenario: Valid CSR is consumed
- **WHEN** a CSR submission matches the OTK binding, parses correctly, carries
  an ML-DSA-65 key with a valid ML-DSA proof of possession, and carries no
  private key material
- **THEN** the backend consumes the OTK and hands the CSR to the PKI layer

#### Scenario: Private key material is submitted
- **WHEN** a submission includes private key material or a field that appears to
  carry a private key
- **THEN** the backend rejects the request before PKI handoff

#### Scenario: Binding mismatch
- **WHEN** the submitted subject, app, device, or profile does not match the OTK
  binding
- **THEN** the backend rejects the submission without issuing a certificate

#### Scenario: Classical CSR is rejected
- **WHEN** a CSR carries an RSA, EC, EdDSA or ML-DSA-44 key
- **THEN** the backend returns `400 csr_key_rejected` without consuming the OTK
- **AND** the PKI sign script would refuse the same CSR

### Requirement: Mobile generates key material and models certificate-ready state
The mobile app SHALL generate an ML-DSA-65 client keypair at runtime, keep the
private key on-device as PKCS#8 (seed and expanded key; never sent to backend,
gateway, or PKI, never committed, bundled, logged, or exported), generate a
PKCS#10 CSR whose subject key and proof-of-possession signature are ML-DSA and
that carries the same identity inputs, use a fail-closed TLS client that never
accepts all certificates, and model bootstrap state as one of `missing`,
`ready`, `expired`, `untrusted`, `csrRejected`, `otkExpired`, or `otkReplayed`;
the CSR SHALL verify with OpenSSL >= 3.5 and BouncyCastle.

#### Scenario: Runtime keypair and CSR
- **WHEN** the app begins certificate bootstrap after OAuth2 authentication
- **THEN** it generates the ML-DSA-65 keypair at runtime and a CSR bound to
  `oauth2Subject`, `appInstanceId`, `deviceId`, and `certificateProfile` whose
  public key and signature algorithms are `ML-DSA-65`
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

#### Scenario: Interoperability fixture
- **WHEN** `tool/emit_ml_dsa_csr.dart` produces a CSR
- **THEN** `scripts/verify-pqc-csr-interop.sh` verifies it with OpenSSL and
  the backend's `CsrValidator` accepts the checked-in copy

