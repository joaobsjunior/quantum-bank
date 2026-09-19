---

description: "Task list for the post-quantum application envelope and transaction signatures"
---

# Tasks: Post-Quantum Application Envelope and Transaction Signatures

**Input**: Design documents from `/specs/012-pqc-application-envelope/`

**Prerequisites**: plan.md, spec.md, research.md, data-model.md, contracts/envelope-v1.md

**Tests**: Required by the constitution's coverage gates (100% line coverage
in `mobile-app` and `backend`); cross-implementation fixture required by FR-010.

## Format: `[ID] [P?] [Story] Description`

## Phase 1: Setup

- [ ] T001 Add `cryptography: ^2.9.0` to `mobile-app/pubspec.yaml` (X25519)
- [ ] T002 [P] Add `envelope` package skeleton and `EnvelopeProperties` in `backend/src/main/kotlin/com/quantumbank/backend/envelope/`
- [ ] T003 [P] Add `device_signing_keys`, `pix_transaction_signatures` tables and `pix_transfers.signature_nonce` to `backend/src/main/resources/schema.sql`

## Phase 2: Foundational

- [ ] T004 [P] `mobile-app/lib/core/pqc/hybrid_envelope.dart`: `EnvelopeKeySet`, `EnvelopeMessage`, `HybridEnvelope.seal/openResponse`, HKDF + AES-256-GCM
- [ ] T005 [P] `mobile-app/lib/core/pqc/mldsa_x509.dart`: DER certificate parsing (TBS, SPKI, issuer/subject, validity, basicConstraints) and ML-DSA chain verification
- [ ] T006 [P] `backend/.../envelope/HybridEnvelopeCodec.kt`: open request / seal response with BC ML-KEM-768 (no KDF), X25519, HKDF, AES-GCM
- [ ] T007 [P] `backend/.../envelope/EnvelopeSigner.kt`: ML-DSA-65 identity + chain from the TLS key store; ephemeral self-signed signer only in `local`

## Phase 3: User Story 2 - Trusted key set (P1)

- [ ] T008 [US2] `backend/.../envelope/HybridEnvelopeKeyService.kt`: key set generation, `kid`, `notAfter`, signature, previous-set grace
- [ ] T009 [US2] `mobile-app/lib/core/pqc/envelope_keys_verifier.dart`: verify signature + chain + subject; produce `EnvelopeKeySet`
- [ ] T010 [US2] Extend `CsrSubmitResponse` with `envelopeKeys` in `backend/.../bootstrap/OtkService.kt`
- [ ] T011 [US2] `mobile-app/lib/features/bootstrap/*`: parse and verify `envelopeKeys`; `untrusted` on failure; `ReadyCertState.envelopeKeySet`

## Phase 4: User Story 4 - Signing key registration (P2)

- [ ] T012 [US4] `mobile-app/lib/core/pqc/transaction_signer.dart`: `DeviceSigningKey` generation, proof of possession, Pix signature
- [ ] T013 [US4] `mobile-app/features/bootstrap`: send `signingKey` in CSR submission; keep the key in `ReadyCertState.signingKey`
- [ ] T014 [US4] `backend/.../bootstrap/DeviceSigningKeys.kt`: registration model, proof verification (context `quantum-bank-signing-key-v1`), `DeviceSigningKeyRepository`
- [ ] T015 [US4] `backend/.../bootstrap/CsrController.kt` + `OtkService.kt`: accept/require `signingKey` by client policy, store on success

## Phase 5: User Story 1 - Confidential banking traffic (P1)

- [ ] T016 [US1] `mobile-app/lib/core/api/banking_client.dart`: seal bodies / `X-Quantum-Envelope` header, open envelope responses, pass through plaintext problems
- [ ] T017 [US1] `backend/.../envelope/EnvelopeFilter.kt`: client policy, open request (body or header), wrap response, problem codes
- [ ] T018 [US1] `api-gateway/krakend-banking.json`: forward `X-Quantum-Envelope`
- [ ] T019 [US1] `api-gateway/openapi/quantum-bank-v1.yaml`: envelope media type on banking operations, new CSR fields, Pix `signature`

## Phase 6: User Story 3 - Signed Pix orders (P1)

- [ ] T020 [US3] `mobile-app/lib/core/api/live_gateway_banking_api.dart`: canonical order, nonce, `issuedAt`, signature object
- [ ] T021 [US3] `backend/.../banking/TransactionSignature.kt`: request model, canonical message, verifier (device key, skew, nonce table)
- [ ] T022 [US3] `backend/.../banking/PixController.kt` + `PixService.kt`: verify before simulation, store signature with the attempt

## Phase 7: User Story 5 - Transport policy (P2)

- [ ] T023 [US5] `mobile-app/lib/core/config/runtime_config.dart`: `transportPolicy` from `PQC_TRANSPORT_POLICY`
- [ ] T024 [US5] `mobile-app/lib/main.dart` + `app_state.dart`: apply policy, show envelope status on the gate screen

## Phase 8: Evidence and polish

- [ ] T025 [P] `mobile-app/tool/emit_envelope_fixture.dart` + `scripts/verify-pqc-envelope-interop.sh` (OpenSSL 3.5 ML-KEM decapsulation + X25519 derive)
- [ ] T026 [P] `backend/src/test/resources/pqc/envelope-fixture.json` opened by `HybridEnvelopeCodecTest`
- [ ] T027 Tests to 100% line coverage in both repositories
- [ ] T028 [P] Docs: `docs/pqc-ml-dsa-transport.md` section, README pointers, `docs/pqc-transport.html` limitations
- [ ] T029 Commit per repository, draft PRs, submodule pointers in the superproject
