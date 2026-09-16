## Why

Quantum Bank's secure foundation (OAuth2, OTK, CSR, mTLS, PKI) authenticates
every hop with classical RSA certificates and negotiates classical key
exchange. Both are exactly the primitives a cryptographically relevant quantum
computer breaks, and "harvest now, decrypt later" makes banking traffic
recorded today decryptable tomorrow. The project's core value is proving a
secure end-to-end flow, so the transport layer itself must move to the NIST
post-quantum standards: ML-DSA (FIPS 204) for every signature that
authenticates a peer and ML-KEM (FIPS 203, hybrid `X25519MLKEM768`) for every
key exchange.

## What Changes

- **PKI is post-quantum only.** The local root and issuing CAs use ML-DSA-87
  keys; every runtime server/client certificate and the
  `quantum-bank-mobile-client-v1` profile use ML-DSA-65 keys signed with
  ML-DSA-87. CSRs with RSA, EC, EdDSA or ML-DSA-44 keys are rejected by the
  backend validator and by the PKI sign script. PKCS#12 stores are built with
  OpenSSL >= 3.5 (no JDK needed); the scripts resolve OpenSSL >= 3.5 on the
  host or through a pinned `alpine/openssl` container and fail closed
  otherwise.
- **Every TLS socket that crosses a container or host boundary is TLS 1.3 with
  ML-DSA peer authentication and `X25519MLKEM768` key exchange, and refuses
  classical signature schemes and classical-only groups.** KrakenD (Go
  `crypto/tls`) and Keycloak (JDK 21 JSSE) cannot negotiate ML-DSA, so each is
  paired with a HAProxy + OpenSSL 3.5 terminator that shares its network
  namespace; the paired process listens on `127.0.0.1` only. The Spring Boot
  backend and `backend-client` run BouncyCastle BCJSSE in-process.
- **Fail-closed evidence.** New negative fixtures (RSA and ML-DSA-44 client
  certificates signed by the real CA, classical-only key exchange) must fail
  inside the TLS handshake, and a new `pqc-handshake-tests` service proves the
  negotiated group and peer signature scheme on every hop.
- **Mobile identity is ML-DSA.** The app generates an ML-DSA-65 key pair and a
  PKCS#10 request signed with ML-DSA in pure Dart, stores the key as PKCS#8,
  and probes the platform TLS stack at startup; when the stack cannot load
  ML-DSA material (Dart 3.11 BoringSSL), protected access stays closed with an
  explicit reason instead of a classical fallback.
- **BREAKING**: classical certificates, keys and CSRs are no longer accepted
  anywhere; KrakenD no longer terminates TLS; the local issuer is reached only
  through its terminator on `8443`; `keytool` is no longer used.

## Capabilities

### Modified Capabilities
- `secure-gateway-communication`: post-quantum-only TLS policy on every hop,
  terminator topology, JWKS retrieval through the loopback egress.
- `onboarding-pki`: ML-DSA CA hierarchy, ML-DSA-65/87 CSR key policy, PQC
  runtime material and negative fixtures, mobile ML-DSA key/CSR generation.
- `deployment-infrastructure`: Compose topology with terminators, Keycloak
  loopback HTTP behind its terminator, PQC handshake tests, cloud-path notes.
- `external-service-integration`: `backend-client` speaks post-quantum mTLS
  through BCJSSE with an ML-DSA-65 service identity.
- `mobile-banking-journeys`: post-quantum transport precondition and fail-closed
  gating in the app.

## Impact

- Submodules: `pki`, `api-gateway`, `infrastructure`, `backend`,
  `backend-client`, `mobile-app`; superproject README, AGENTS and CI.
- Tooling: OpenSSL >= 3.5 or Docker for PKI scripts; `haproxy:3.2-alpine` and
  `alpine/openssl:3.5.8` images in the local runtime.
- Known limitation (documented, fail-closed): `dart:io` cannot yet open an
  ML-DSA TLS session, so the Flutter app's transport waits on a native TLS
  engine; the smoke client exercises the mobile role end to end.
