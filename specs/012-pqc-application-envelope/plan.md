# Implementation Plan: Post-Quantum Application Envelope and Transaction Signatures

**Branch**: `012-pqc-application-envelope` | **Date**: 2026-09-18 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `/specs/012-pqc-application-envelope/spec.md`

## Summary

Add a transport-independent post-quantum layer between the Flutter app and the
Spring Boot backend: hybrid ML-KEM-768 + X25519 envelopes around every banking
request and response (forwarded opaquely by KrakenD), ML-DSA-65 signatures on
every Pix order with a device signing key registered at enrollment, a
PKI-signed envelope key set verified in pure Dart, and an explicit transport
policy instead of a certificate-loading probe. Nothing changes in the TLS tiers
of feature 011.

## Technical Context

**Language/Version**: Dart 3.11 / Flutter 3.41 (mobile); Kotlin 2.2 / JDK 17 /
Spring Boot 4.0.6 (backend); KrakenD 2.13 JSON + OpenAPI 3 (gateway)

**Primary Dependencies**: mobile: `pqcrypto` 0.4.1 (ML-KEM-768 encapsulation,
ML-DSA-65 sign/verify), `pointycastle` 4.0 (ASN.1, HKDF, AES-GCM),
`cryptography` 2.9 (X25519, RFC 7748); backend: BouncyCastle 1.86 (ML-KEM-768
`KEMGenerateSpec/KEMExtractSpec` with no KDF, ML-DSA-65 with
`ContextParameterSpec`, X25519, HKDF), JDK AES/GCM

**Storage**: backend H2 (`device_signing_keys`, `pix_transaction_signatures`,
new `signature` columns on `pix_transfers`); envelope keys in memory per
instance

**Testing**: `flutter test --coverage` + `scripts/check-coverage.sh 100`;
`gradle check` with Kover 100% line; cross-implementation fixture
(`src/test/resources/pqc/envelope-fixture.json`) emitted by
`tool/emit_envelope_fixture.dart`; OpenSSL 3.5 checks in
`scripts/verify-pqc-envelope-interop.sh`

**Target Platform**: Android / iOS app; Linux containers

**Project Type**: Mobile + API (multi-repository superproject)

**Performance Goals**: one ML-KEM-768 encapsulation + X25519 + AES-GCM per
request in pure Dart: well under 50 ms on a mid-range phone; ML-DSA-65 signing
of a Pix order in pure Dart: under 100 ms

**Constraints**: no RSA/EdDSA anywhere; hybrid only; fail closed on any
verification failure; the gateway sees only ciphertext; 100% line coverage in
both code bases

**Scale/Scope**: 3 banking routes + CSR submission; 2 repositories with code
changes, 1 with contract changes

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

| Principle | Status | Notes |
| --- | --- | --- |
| I. Gateway é a única rota | PASS | Envelopes travel through KrakenD unchanged; no new direct app → backend path; the new `X-Quantum-Envelope` header is added to the gateway's forwarded headers |
| II. TLS sem atalhos | PASS | TLS tiers untouched; the envelope is an additional layer; key set trust is anchored in the PKI (ML-DSA chain verified in Dart) |
| III. Segurança antes da tela | PASS | Only the gate screen text changes |
| IV. Pix v1 é simulação | PASS | Signature verification precedes the existing simulation; nothing touches real rails |
| V. Diferenças de nuvem explícitas | PASS | No Terraform change |
| Stack (Flutter 3.41, KrakenD 2.13, Spring Boot 4.0.6, H2) | PASS | One new pure-Dart dependency (`cryptography`) for X25519; justified below |

Complexity justification: a second cryptographic layer duplicates work the
TLS layer would do on a capable client. It is required because the client TLS
stack cannot be made post-quantum today and because a cloud ingress that
terminates TLS must not be able to read banking data. The layer is removable
by configuration once TLS reaches parity (`required-for-clients` empty).

## Project Structure

### Documentation (this feature)

```text
specs/012-pqc-application-envelope/
├── plan.md
├── research.md
├── data-model.md
├── quickstart.md
├── contracts/
│   └── envelope-v1.md
└── tasks.md
```

### Source Code (repository root)

```text
mobile-app/
├── lib/core/pqc/
│   ├── hybrid_envelope.dart          # key set, seal/open, KDF, AEAD
│   ├── mldsa_x509.dart               # minimal X.509 parsing + ML-DSA chain verification
│   ├── envelope_keys_verifier.dart   # signed key set → trusted EnvelopeKeySet
│   └── transaction_signer.dart       # device signing key + Pix signature
├── lib/core/api/banking_client.dart  # seals requests, opens responses
├── lib/core/api/live_gateway_banking_api.dart  # attaches Pix signatures
├── lib/core/tls/cert_state.dart      # ready state carries key set + signing key
├── lib/core/config/runtime_config.dart          # transport policy
├── lib/features/bootstrap/*          # signing key registration, key set verification
├── tool/emit_envelope_fixture.dart
├── scripts/verify-pqc-envelope-interop.sh
└── test/…

backend/
├── src/main/kotlin/com/quantumbank/backend/envelope/
│   ├── EnvelopeProperties.kt
│   ├── EnvelopeSigner.kt             # ML-DSA-65 identity from the TLS key store
│   ├── HybridEnvelopeKeyService.kt   # ML-KEM-768 + X25519 key sets, signed, rotated
│   ├── HybridEnvelopeCodec.kt        # open request / seal response
│   ├── EnvelopeFilter.kt             # policy + request/response wrapping
│   └── EnvelopeErrorCodes.kt
├── src/main/kotlin/com/quantumbank/backend/banking/
│   ├── TransactionSignature.kt       # request model + verifier + repository
│   └── PixController.kt / PixService.kt (signature verification, storage)
├── src/main/kotlin/com/quantumbank/backend/bootstrap/
│   ├── DeviceSigningKeys.kt          # registration validation + repository
│   └── CsrController.kt / OtkService.kt (signingKey, envelopeKeys)
├── src/main/resources/schema.sql
└── src/test/…, src/test/resources/pqc/envelope-fixture.json

api-gateway/
├── krakend-banking.json              # X-Quantum-Envelope forwarded
└── openapi/quantum-bank-v1.yaml      # envelope media type, new fields
```

**Structure Decision**: Mobile + API across the existing submodules; the
envelope code lives in a dedicated `envelope` package on the backend and under
`lib/core/pqc` on the app, next to the existing ML-DSA CSR code.

## Complexity Tracking

| Violation | Why Needed | Simpler Alternative Rejected Because |
|-----------|------------|-------------------------------------|
| New Dart dependency `cryptography` | X25519 (RFC 7748) is not in `pointycastle` or `pqcrypto` | A hand-written Montgomery ladder in a banking app would be unreviewed cryptography |
| Second cryptographic layer | Client TLS stack cannot be post-quantum; cloud ingress may terminate TLS | Waiting for Dart leaves "harvest now, decrypt later" open on the app edge |
