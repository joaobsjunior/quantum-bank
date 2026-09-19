---

description: "Task list for hardware-backed device keys"
---

# Tasks: Hardware-Backed Device Keys

**Input**: Design documents from `/specs/013-hardware-backed-device-keys/`

**Status**: not started; depends on feature 015 for story 1.

## Phase 1: Abstraction (no platform code)

- [ ] T001 `mobile-app/lib/core/keys/device_key_store.dart`: `DeviceKeyStore` interface, `SecurityLevel`, `SoftwareDeviceKeyStore` (wraps today's `KeypairService` and `DeviceSigningKey`)
- [ ] T002 `mobile-app/lib/features/bootstrap/enrollment_orchestrator.dart`: sign the CSR through the store handle; keep the sealed seed, not the raw seed, in `ReadyCertState`
- [ ] T003 `mobile-app/lib/core/config/runtime_config.dart`: `HARDWARE_KEYS=required|preferred` policy
- [ ] T004 Tests to 100% with the software store

## Phase 2: Platform plugin

- [ ] T005 [P] Android: `DeviceKeyStorePlugin.kt` (StrongBox-preferred ECDSA P-256 key, AES-GCM seal key, Key Attestation chain, Play Integrity token)
- [ ] T006 [P] iOS: `DeviceKeyStorePlugin.swift` (Secure Enclave P-256, keychain-sealed seed, App Attest)
- [ ] T007 `platform_device_key_store.dart`: MethodChannel client, zeroization, error mapping

## Phase 3: Attestation on the backend

- [ ] T008 `backend/.../bootstrap/Attestation.kt`: verify Android attestation chain / App Attest, bind to the CSR fingerprint, policy `quantum-bank.attestation.required`
- [ ] T009 `schema.sql` + `DeviceSigningKeyRepository`: `security_level`
- [ ] T010 Audit events and problem codes (`attestation_rejected`)

## Phase 4: Hardware-held mTLS (with feature 015)

- [ ] T011 Native transport client uses the keystore handle for the client certificate
- [ ] T012 Device test protocol (physical devices) and evidence in `docs/`
