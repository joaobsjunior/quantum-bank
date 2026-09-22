# Quickstart: Post-Quantum Application Envelope

## Validate locally

```sh
# mobile-app: unit tests (100% line coverage) and interop evidence
cd mobile-app
flutter pub get && flutter analyze && flutter test --coverage
./scripts/check-coverage.sh 100 coverage/lcov.info
./scripts/verify-pqc-envelope-interop.sh      # Dart ML-KEM/X25519 vs OpenSSL 3.5

# backend: unit + MockMvc tests, Dart fixture opened with BouncyCastle
cd ../backend
./gradlew check

# api-gateway: contract and gateway config
cd ../api-gateway
./scripts/ci-validate.sh
```

## Refresh the cross-implementation fixture

```sh
cd mobile-app
dart run tool/emit_envelope_fixture.dart ../backend/src/test/resources/pqc/envelope-fixture.json
```

## Run the stack

Unchanged from the README; the mobile app enrolls, receives the signed
envelope key set, and every banking call is enveloped and (for Pix) signed.
Service clients (`backend-client`, smoke tests) keep calling in plaintext over
the strict post-quantum TLS tier.

## Transport policy

```sh
flutter run --dart-define=PQC_TRANSPORT_POLICY=compatibility   # default
flutter run --dart-define=PQC_TRANSPORT_POLICY=probe           # feature 011 probe
```
