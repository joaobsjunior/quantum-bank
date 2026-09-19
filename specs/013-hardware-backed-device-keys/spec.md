# Feature Specification: Hardware-Backed Device Keys

**Feature Branch**: `013-hardware-backed-device-keys`

**Created**: 2026-09-19

**Status**: Planned (specification and plan only; implementation requires
platform work on Android and iOS devices)

**Input**: Recommendation item 3 for a banking-grade posture: keep the device
transport identity in platform hardware (Android StrongBox / iOS Secure
Enclave) and seal the ML-DSA-65 signing seed with a hardware key, because key
extraction from process memory is a concrete risk today while breaking ECDSA
P-256 is not.

## Overview

Feature 012 gave the device two long-lived secrets: the TLS transport private
key (ECDSA P-256 in compatibility mode, ML-DSA-65 in probe mode) and the
ML-DSA-65 signing seed used for Pix orders. Both currently live in process
memory and, after this feature, at rest only inside platform-protected storage:

1. the transport identity key is generated and kept inside the platform
   keystore (StrongBox-backed `AndroidKeyStore`, Secure Enclave `SecKey`),
   never exported; the CSR is signed through the keystore handle;
2. the ML-DSA-65 signing seed (32 bytes, hardware keystores do not implement
   ML-DSA yet) is sealed at rest with a hardware-backed AES-256-GCM key and
   only unsealed into memory for the duration of a signature;
3. a platform attestation (Play Integrity / Key Attestation, App Attest) is
   sent with the CSR so the backend can record that the transport key is
   hardware-backed and refuse enrollment from an emulator or rooted device
   when policy demands it.

Constraint recorded from the platform analysis: `dart:io` loads the TLS client
key from bytes (`SecurityContext.usePrivateKeyBytes`), so a non-exportable
hardware key **cannot** be used for mTLS by the Dart TLS stack. Story 1 is
therefore delivered together with the native transport client (planned as
feature 015, OpenSSL 3.5 through `dart:ffi` or platform networking with a
client-certificate delegate). Stories 2 and 3 do not depend on it.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Transport identity never leaves the hardware (Priority: P1)

As the bank, a stolen device image or a compromised app process must not yield
a usable copy of the device's TLS identity.

**Independent Test**: Generate the identity on a StrongBox-capable device,
export attempts fail, the CSR verifies with the certified public key, and the
enrolled certificate authenticates a banking call through the native
transport.

**Acceptance Scenarios**:

1. **Given** a device with StrongBox or Secure Enclave, **When** the app
   enrolls, **Then** the ECDSA P-256 transport key is generated inside the
   hardware with `setIsStrongBoxBacked(true)` / `kSecAttrTokenIDSecureEnclave`,
   and any export attempt fails.
2. **Given** a device without hardware support, **When** the app enrolls,
   **Then** the policy (`HARDWARE_KEYS=required|preferred`) decides between
   refusing enrollment and falling back to the TEE-backed or software
   keystore, and the chosen level is reported to the backend.
3. **Given** a hardware-held key, **When** a banking call is made, **Then**
   the native transport client performs the mTLS handshake with the keystore
   handle (the key bytes are never in Dart memory).

---

### User Story 2 - The ML-DSA-65 signing seed is sealed at rest (Priority: P1)

As the bank, the post-quantum signing seed must be unreadable from the app
sandbox at rest and unsealed only while a Pix order is being signed.

**Independent Test**: Seal a seed, kill and restart the app, unseal and sign;
copy the sealed blob to another device and confirm it cannot be unsealed.

**Acceptance Scenarios**:

1. **Given** a generated signing seed, **When** it is stored, **Then** it is
   encrypted with a hardware-backed AES-256-GCM key bound to the app and
   device (user authentication optional by policy), and the plaintext seed
   is zeroized after use.
2. **Given** the sealed blob moved to another device or app, **When** an
   unseal is attempted, **Then** it fails.
3. **Given** a device without hardware keystore, **When** policy is
   `required`, **Then** the signing key is not created and the app stays in
   the gate screen with an explicit reason.

---

### User Story 3 - Enrollment carries a platform attestation (Priority: P2)

As the bank, I want to know at enrollment whether the transport key is
hardware-backed and whether the device passes platform integrity checks.

**Acceptance Scenarios**:

1. **Given** an enrollment, **When** the CSR is submitted, **Then** it carries
   an `attestation` object (Android Key Attestation certificate chain and Play
   Integrity token, or App Attest assertion) bound to the CSR fingerprint.
2. **Given** an attestation that fails verification, or a policy that
   requires hardware backing on a device that reports none, **When** the CSR
   is received, **Then** the backend rejects the enrollment with
   `400 attestation_rejected`, without consuming the OTK.
3. **Given** a verified attestation, **When** the certificate is issued,
   **Then** the backend stores the security level with the device signing key
   and exposes it in audit events.

---

### Edge Cases

- Key invalidation after biometric enrollment changes (Android
  `setInvalidatedByBiometricEnrollment`) must trigger re-enrollment, not a
  crash.
- iOS Secure Enclave keys are P-256 only, which matches the compatibility
  chain; a probe-mode (ML-DSA) transport identity can never be
  hardware-backed until the platform supports ML-DSA.
- Backups (iCloud, Android Auto Backup) must exclude the sealed blobs, or the
  blobs must be device-bound so a restored copy is useless.

## Requirements *(mandatory)*

- **FR-001**: The transport identity key MUST be generated inside the platform
  keystore when available and MUST NOT be exportable.
- **FR-002**: The signing seed MUST be encrypted at rest with a
  hardware-backed key and MUST be zeroized after each use.
- **FR-003**: The policy `HARDWARE_KEYS=required|preferred` MUST be an
  explicit build setting; `required` MUST fail closed.
- **FR-004**: The CSR submission MUST carry an attestation object and the
  backend MUST verify it before consuming the OTK.
- **FR-005**: The Dart layer MUST access keys only through a `DeviceKeyStore`
  abstraction with a software implementation for tests; no platform code in
  unit tests.
- **FR-006**: mTLS with a hardware-held key MUST go through the native
  transport client (feature 015); the app MUST not export the key to satisfy
  `dart:io`.

### Key Entities

- **DeviceKeyStore**: `generateTransportKey(policy)`, `signCsr(handle, der)`,
  `sealSeed(seed)`, `unsealSeed(blob)`, `attest(csrFingerprint)`.
- **SecurityLevel**: `strongbox | secure_enclave | tee | software`.
- **Attestation**: platform-specific chain or token plus the CSR fingerprint.

## Success Criteria *(mandatory)*

- **SC-001**: On a StrongBox/Secure Enclave device the transport key export
  fails and enrollment succeeds through the native transport.
- **SC-002**: The sealed seed blob cannot be unsealed on another device.
- **SC-003**: Enrollment from an emulator is refused when policy is
  `required`.
- **SC-004**: Unit coverage in `mobile-app` and `backend` stays at 100% with
  the software `DeviceKeyStore`.

## Assumptions

- Feature 015 (native transport client) ships before or with story 1.
- Platform work is validated on physical devices by the mobile team; the
  cloud CI keeps the software key store.
