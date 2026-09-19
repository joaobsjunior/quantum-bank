# Feature Specification: Post-Quantum Application Envelope and Transaction Signatures

**Feature Branch**: `012-pqc-application-envelope`

**Created**: 2026-09-18

**Status**: In implementation

**Input**: User description: "Eu quero o caminho com maior segurança, pois estamos
falando de um sistema bancário. Siga com estas mudanças usando o speckit." The
recommendation being implemented: protect the app edge with a post-quantum
layer that does not depend on the mobile TLS stack, sign every Pix transaction
with the device's post-quantum key, and stop guessing whether the platform can
present ML-DSA over TLS.

## Overview

The app-facing TLS hop (mobile app → Keycloak / KrakenD) is the only hop of the
platform whose key exchange and server authentication can still be classical:
`dart:io` (BoringSSL) negotiates neither `X25519MLKEM768` nor ML-DSA on any
current Flutter release, and a cloud ingress that terminates TLS would replace
the project identity anyway. This feature adds a **second, transport-independent
post-quantum layer** between the app and the backend:

1. every banking request and response body travels inside a **hybrid envelope**
   (ML-KEM-768 + X25519 key agreement, HKDF-SHA-256, AES-256-GCM) that the
   gateway forwards opaquely;
2. every Pix transfer carries an **ML-DSA-65 signature** made with a device
   signing key registered at enrollment, verified and stored by the backend
   (post-quantum non-repudiation per transaction);
3. the envelope key set the backend publishes is **signed with the backend's
   PKI-issued ML-DSA-65 identity** and verified by the app against the bundled
   ML-DSA-87 root in pure Dart, so the trust decision never depends on the
   platform TLS stack;
4. the transport mode of the device stops being auto-detected from certificate
   loading alone: it is an explicit runtime policy (`compatibility` by default,
   `probe` opt-in), because a stack that loads ML-DSA certificates but cannot
   sign the TLS handshake with an ML-DSA key would otherwise lock the device out.

The transport tiers of feature 010/011 stay as they are. The envelope is
required only for clients on the dual (app-edge) tier, identified by their
OAuth2 client; service clients on the strict tier (backend-client, test client)
already have post-quantum TLS end to end and are exempt.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Banking traffic is confidential even if the app-edge TLS is classical (Priority: P1)

As a customer, the content of my statements, profile and Pix requests must stay
confidential against an adversary that records today's traffic and gains a
cryptographically relevant quantum computer later, even though my phone's TLS
stack negotiates classical key exchange with the gateway.

**Why this priority**: "harvest now, decrypt later" is the only quantum threat
that is real today, and the app edge is the only hop where it is not yet
mitigated.

**Independent Test**: Enroll a device, call `GET /statements` through the
gateway, capture the TLS plaintext at the gateway (the terminator sees it) and
confirm the body is an opaque envelope; confirm the backend returns the
statement only inside a response envelope keyed to the request.

**Acceptance Scenarios**:

1. **Given** an enrolled device with a verified envelope key set, **When** the
   app sends a banking request, **Then** the request body (or, for `GET`, the
   `X-Quantum-Envelope` header) carries an ML-KEM-768 ciphertext, an X25519
   public key and an AES-256-GCM ciphertext, and no banking field is visible to
   the gateway.
2. **Given** an enveloped request, **When** the backend answers, **Then** the
   response body is an envelope encrypted under a key derived from the same
   hybrid secret (response direction), and the app decrypts it to the JSON the
   API contract describes.
3. **Given** a request from the mobile OAuth2 client without an envelope,
   **When** it reaches the backend, **Then** the backend rejects it with
   `400 envelope_required` and never processes the plaintext.
4. **Given** a request from a strict-tier service client (backend-client, test
   client), **When** it reaches the backend without an envelope, **Then** the
   backend processes it as before (post-quantum TLS already covers that hop).
5. **Given** an envelope whose `kid` the backend does not know, **When** it is
   received, **Then** the backend rejects it with `400 envelope_key_unknown` and
   the app re-enrolls to obtain the current key set.

---

### User Story 2 - The envelope key set is trusted only through the PKI (Priority: P1)

As the bank, the public keys the app encrypts to must come from the backend and
nobody else, verified with post-quantum signatures the app checks itself.

**Why this priority**: an envelope keyed to an attacker's keys is worse than no
envelope. The verification must not depend on the TLS stack that motivated the
feature.

**Independent Test**: Tamper one byte of the key set or of the signer chain in
a recorded enrollment response and confirm the app refuses it (`untrusted`).

**Acceptance Scenarios**:

1. **Given** an enrollment response, **When** the app receives `envelopeKeys`,
   **Then** it verifies the ML-DSA-65 signature over the canonical key set with
   the leaf certificate of `signerChain`, verifies the chain up to the bundled
   ML-DSA-87 root (signatures, validity window, issuer/subject linkage, CA
   flag), and checks the leaf subject is the backend identity.
2. **Given** a key set whose signature, chain or subject does not verify,
   **When** the app processes the enrollment, **Then** the certificate state is
   `untrusted` and no banking call is possible.
3. **Given** a key set past `notAfter`, **When** the app is about to use it,
   **Then** it re-enrolls instead of encrypting to an expired key.

---

### User Story 3 - Every Pix transfer is signed with the device's post-quantum key (Priority: P1)

As the bank, I need proof that a Pix order came from the enrolled device and
was not altered, independent of the transport and of the bearer token.

**Why this priority**: non-repudiation of payment orders is the banking control
that matters most, and RS256 tokens plus classical TLS do not provide a
post-quantum one.

**Independent Test**: Submit a Pix order with a valid signature and see it
accepted and stored with the signature; alter the amount after signing, or
replay the same signed order, and see the backend refuse.

**Acceptance Scenarios**:

1. **Given** a device that registered an ML-DSA-65 signing key at enrollment,
   **When** it submits `POST /pix/transfers`, **Then** the request carries a
   signature over the canonical transaction (subject, device, amount,
   recipient, description, scenario, nonce, issuedAt) with the FIPS 204
   context `quantum-bank-pix-v1`, and the backend verifies it before simulating
   the transfer.
2. **Given** a signature that does not verify, a nonce already used, an
   `issuedAt` outside the allowed skew, or an unknown device key, **When** the
   order is received, **Then** the backend rejects it with a
   `400 transaction_signature_invalid` / `409 transaction_signature_replayed`
   problem and records nothing as completed.
3. **Given** an accepted order, **When** the backend stores the attempt,
   **Then** the signature, nonce and device are stored with it for audit.
4. **Given** a strict-tier service client, **When** it submits a Pix order
   without a signature, **Then** the order is processed as before (the policy
   is keyed by OAuth2 client, like the envelope).

---

### User Story 4 - The device registers its signing key at enrollment (Priority: P2)

As the bank, the signing key of a device must be bound to the same OAuth2
subject, device and one-time key that authorize its certificate.

**Independent Test**: Submit a CSR with a signing-key registration whose proof
of possession does not verify and see `400 signing_key_invalid`; submit a valid
one and see the key stored for (subject, deviceId).

**Acceptance Scenarios**:

1. **Given** an enrollment, **When** the app submits the CSR, **Then** the
   request also carries `signingKey { alg: ML-DSA-65, publicKey, proof }` where
   `proof` is an ML-DSA-65 signature over the DER CSR with context
   `quantum-bank-signing-key-v1`.
2. **Given** a registration whose proof does not verify or whose algorithm is
   not ML-DSA-65, **When** it is received, **Then** the backend rejects the
   enrollment with `400 signing_key_invalid`.
3. **Given** the mobile OAuth2 client, **When** it submits a CSR without a
   registration, **Then** the backend rejects it with `400 signing_key_required`.
4. **Given** a re-enrollment of the same (subject, deviceId), **When** it is
   accepted, **Then** the new signing key replaces the previous one.

---

### User Story 5 - The transport mode is an explicit policy (Priority: P2)

As an operator, I decide when the device may present an ML-DSA identity over
TLS; the app must not infer it from a certificate-loading probe.

**Independent Test**: Build the app with the default policy on a stack that
loads ML-DSA certificates but cannot sign the handshake (Dart 3.13) and confirm
it enrolls an ECDSA P-256 transport identity and still passes envelope and Pix
signature checks.

**Acceptance Scenarios**:

1. **Given** no `PQC_TRANSPORT_POLICY` build setting, **When** the app starts,
   **Then** it runs the compatibility transport (ECDSA P-256 identity, ECDSA
   root trusted) and the post-quantum protections of stories 1–3 apply
   unchanged.
2. **Given** `PQC_TRANSPORT_POLICY=probe`, **When** the app starts, **Then** the
   existing certificate-loading probe selects the mode as in feature 011.
3. **Given** either policy, **When** the gate screen renders, **Then** it shows
   the transport mode and that the application-layer post-quantum envelope is
   active.

---

### Edge Cases

- A response that is a problem document produced before the envelope filter
  (401 from the resource server, 403 from the gateway) is plaintext: the app
  parses it as a problem and never treats it as decrypted data.
- The envelope key set rotates while a device still holds the previous `kid`:
  the backend keeps the previous set for a grace period and answers
  `envelope_key_unknown` afterwards; the app re-enrolls on that error.
- A `GET` request has no body: the encapsulation travels in the
  `X-Quantum-Envelope` header, which KrakenD must forward.
- Nonce storage grows with every signed order: the backend keeps nonces for
  the accepted skew window plus retention and rejects duplicates inside it.
- The backend runs without a PKI-issued identity (unit tests, `local` without
  TLS): it must fail closed outside the `local` environment and use an
  ephemeral self-signed ML-DSA-65 signer only when explicitly configured.
- Both envelope and signature use hybrid or post-quantum primitives only; no
  RSA, EdDSA or classical-only construction is accepted anywhere.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The mobile app MUST encrypt every banking request body with a
  fresh hybrid secret (ML-KEM-768 encapsulation to the backend's key, X25519
  ephemeral agreement, HKDF-SHA-256, AES-256-GCM) and MUST decrypt the
  response with the response key derived from the same secret.
- **FR-002**: The backend MUST publish an envelope key set (ML-KEM-768 and
  X25519 public keys, `kid`, `notAfter`) signed with its PKI-issued ML-DSA-65
  identity, and MUST return it in the CSR response.
- **FR-003**: The mobile app MUST verify the key set signature and the signer
  chain up to the bundled ML-DSA-87 root in pure Dart before using the keys,
  and MUST fail closed (`untrusted`) otherwise.
- **FR-004**: The backend MUST require the envelope for requests whose OAuth2
  client is in `quantum-bank.envelope.required-for-clients` (default:
  `quantum-bank-mobile`) and MUST accept plaintext from other clients.
- **FR-005**: The backend MUST reject envelopes it cannot open with a problem
  document (`envelope_invalid`, `envelope_key_unknown`, `envelope_required`)
  and MUST never process the ciphertext as a request.
- **FR-006**: The mobile app MUST generate an ML-DSA-65 signing key at
  enrollment (independent of the transport identity), register it in the CSR
  submission with a proof of possession, and sign every Pix order with it.
- **FR-007**: The backend MUST verify Pix signatures (algorithm, device key,
  nonce uniqueness, `issuedAt` skew) for required clients and MUST store the
  signature with the attempt.
- **FR-008**: KrakenD MUST forward the `X-Quantum-Envelope` request header and
  the envelope content type unchanged; the envelope MUST be opaque to the
  gateway.
- **FR-009**: The transport mode MUST be selected by the `PQC_TRANSPORT_POLICY`
  build setting (`compatibility` default, `probe`), and the gate screen MUST
  show the mode and the envelope status.
- **FR-010**: All new cryptography MUST be covered by cross-implementation
  evidence: a Dart-generated fixture opened by the backend (BouncyCastle) and
  OpenSSL 3.5 checks of the ML-KEM and X25519 primitives.
- **FR-011**: Line coverage MUST stay at 100% in `mobile-app` and `backend`.

### Key Entities

- **EnvelopeKeySet**: `kid`, `alg` (`X25519MLKEM768-HKDF-SHA256-AES256GCM`),
  `mlkemPublicKey` (1184 bytes), `x25519PublicKey` (32 bytes), `notAfter`.
- **SignedEnvelopeKeySet**: an EnvelopeKeySet plus `signature` (ML-DSA-65 over
  the canonical encoding, context `quantum-bank-envelope-keys-v1`) and
  `signerChain` (PEM certificates, leaf first).
- **EnvelopeMessage**: `v`, `kid`, `mlkemCiphertext`, `x25519PublicKey`,
  `nonce`, `ciphertext` (AES-256-GCM with tag), `aad` (method + path).
- **DeviceSigningKey**: (`subject`, `deviceId`) → ML-DSA-65 public key,
  `registeredAt`.
- **TransactionSignature**: `alg`, `deviceId`, `nonce`, `issuedAt`, `value`;
  stored with the Pix attempt.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: With a device enrolled through the local stack, no banking field
  appears in the gateway-side plaintext of any banking request or response.
- **SC-002**: A tampered or replayed Pix order is rejected in 100% of the
  negative tests; an untampered order is accepted and stored with its
  signature.
- **SC-003**: The Dart-generated envelope fixture is opened by the backend test
  suite and the ML-KEM and X25519 shared secrets match OpenSSL 3.5.
- **SC-004**: `flutter test --coverage` and `gradle check` keep 100% line
  coverage; `verify-pqc-gateway.sh` and `ci-validate.sh` stay green.
- **SC-005**: The compatibility transport plus envelope path works on Flutter
  3.41 without any change to server TLS configuration.

## Assumptions

- Keycloak and KrakenD remain unchanged except for header forwarding; JWTs
  stay RS256 and only travel inside TLS.
- The backend keeps envelope keys in memory per instance (v1); a shared key
  store is a deployment concern for multi-instance runs and is out of scope.
- Hardware-backed key storage on the device is feature 013; here the signing
  key and the transport key live in process memory like the existing identity.
- PKI revocation and renewal automation are feature 014.
