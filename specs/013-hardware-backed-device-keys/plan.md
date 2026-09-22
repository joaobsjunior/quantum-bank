# Implementation Plan: Hardware-Backed Device Keys

**Branch**: `013-hardware-backed-device-keys` | **Date**: 2026-09-19 | **Spec**: [spec.md](spec.md)

## Summary

Move the device's transport identity into the platform keystore and seal the
ML-DSA-65 signing seed with a hardware-backed key, through a `DeviceKeyStore`
abstraction implemented by a Flutter plugin (Kotlin on Android, Swift on iOS)
with a software implementation for tests. Attestation is added to the CSR
submission and verified by the backend. mTLS with a non-exportable key depends
on the native transport client (feature 015).

## Technical Context

**Language/Version**: Dart 3.11 / Flutter 3.41; Kotlin (Android Keystore,
StrongBox, Key Attestation, Play Integrity); Swift (Security framework, Secure
Enclave, App Attest); Kotlin/Spring Boot backend for attestation verification

**Primary Dependencies**: platform SDKs; backend: BouncyCastle for the Android
attestation chain, Apple App Attest verification (CBOR, X.509)

**Storage**: platform keystore; sealed seed blobs in app-private storage;
backend `device_signing_keys.security_level`

**Testing**: software `DeviceKeyStore` in unit tests (100% line coverage);
device tests on physical hardware (not in cloud CI)

**Constraints**: `dart:io` cannot use non-exportable keys for TLS; Secure
Enclave is P-256 only; no ML-DSA in hardware keystores yet

## Constitution Check

| Principle | Status | Notes |
| --- | --- | --- |
| I. Gateway única rota | PASS | Attestation travels with the CSR through the bootstrap listener |
| II. TLS sem atalhos | PASS (conditional) | Hardware-held TLS keys require feature 015; no permissive fallback is introduced |
| III. Segurança antes da tela | PASS | Gate screen reports the security level |
| IV. Pix simulação | PASS | Untouched |
| V. Nuvem explícita | PASS | Untouched |

## Project Structure

```text
mobile-app/
├── lib/core/keys/device_key_store.dart        # abstraction + SoftwareDeviceKeyStore
├── lib/core/keys/platform_device_key_store.dart  # MethodChannel client
├── android/src/main/kotlin/.../DeviceKeyStorePlugin.kt
├── ios/Classes/DeviceKeyStorePlugin.swift
└── lib/features/bootstrap/*                   # CSR signed via the store, attestation attached

backend/
├── src/main/kotlin/.../bootstrap/Attestation.kt   # verification, policy, security level
└── src/main/resources/schema.sql              # security_level column
```

**Structure Decision**: plugin code lives inside the app repository
(`android/`, `ios/`) to keep one review surface; the Dart abstraction keeps
unit tests platform-free.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| Native plugin code | Hardware keystores have no Dart API | `flutter_secure_storage` stores blobs, it does not generate non-exportable keys |
| Dependency on feature 015 | `dart:io` needs key bytes | Exporting the key would defeat the feature |
