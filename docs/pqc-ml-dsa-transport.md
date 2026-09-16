# Post-Quantum Transport (ML-DSA + ML-KEM)

Every Quantum Bank communication that crosses a container or host boundary is
TLS 1.3 with **ML-DSA** peer authentication (FIPS 204) and the
**X25519MLKEM768** hybrid key exchange (FIPS 203). ML-DSA is a signature
algorithm: it is what authenticates every certificate, CSR and handshake.
Confidentiality comes from the ML-KEM hybrid key exchange that the same TLS
sessions negotiate. Classical signature schemes, classical-only groups and
classical keys are refused on every hop.

## Policy

| Item | Value |
| --- | --- |
| Root CA / issuing CA | ML-DSA-87 keys and signatures |
| Runtime and mobile certificates | ML-DSA-65 keys (ML-DSA-87 accepted for CSRs), `digitalSignature` only |
| TLS protocol | TLS 1.3 only |
| Key exchange | `X25519MLKEM768` only |
| Signature schemes (both directions) | `mldsa65`, `mldsa87` only |
| Rejected | RSA, EC, EdDSA, ML-DSA-44 keys; RSA/ECDSA/EdDSA schemes; X25519/secp* groups alone |
| Application-layer tokens | JWT stays RS256 (Keycloak/KrakenD have no ML-DSA JWS); tokens only travel inside post-quantum TLS |

## Per-layer implementation

| Layer | Runtime | How it speaks post-quantum |
| --- | --- | --- |
| `pki` | OpenSSL >= 3.5 (host or `alpine/openssl:3.5.8`) | ML-DSA CA hierarchy, runtime certs, PEM bundles, PKCS#12 via OpenSSL, ML-DSA-only CSR policy, negative fixtures, handshake evidence script |
| `api-gateway` | KrakenD 2.13 (Go, no ML-DSA) + HAProxy 3.2 / OpenSSL 3.5 | KrakenD binds loopback only; HAProxy terminator in the same network namespace owns `8080`/`8443`, enforces app mTLS, carries the gateway identity to the backend and verifies the issuer for JWKS |
| `infrastructure` | Compose, Terraform | `keycloak-tls`, `gateway-*-tls` sidecars (`network_mode: service:`), Keycloak on `127.0.0.1:8080` with proxy headers, `pqc-handshake-tests` service, terminator sidecar in the cloud paths |
| `backend` | Spring Boot on JDK 17 + BouncyCastle BCJSSE | In-process TLS 1.3 with the ML-DSA-65 server certificate, ML-DSA client auth required, ML-DSA-only CSR validation, Alpine runtime with OpenSSL 3.5 for the sign script |
| `backend-client` | Spring Boot on JDK 17 + BCJSSE | HttpClient with the ML-DSA-65 service identity, fail-closed |
| `mobile-app` | Flutter 3.41 / Dart 3.11 | ML-DSA-65 key pair, PKCS#8, PKCS#10 CSR in pure Dart (`pqcrypto`); ML-DSA-87 root asset; startup probe of the platform TLS stack with fail-closed gating |

Loopback traffic between a terminator and the process it fronts never leaves
the container network namespace and is the equivalent of an in-process hop.

## Evidence gates

- `pki/scripts/ci-validate.sh`: ML-DSA-87 anchors, ML-DSA-65 runtime material, PKCS#12 stores, negative fixtures with their forbidden algorithms.
- `api-gateway/scripts/ci-validate.sh`: `krakend check`, `haproxy -c` with generated ML-DSA material, `verify-pqc-gateway.sh`.
- `infrastructure/scripts/verify-local-e2e-config.sh`: terminator topology, loopback bindings, forbidden classical/plaintext settings.
- Backend / backend-client `gradle check`: real loopback ML-DSA mutual handshakes over `X25519MLKEM768`, RSA identity and anonymous client refused inside the handshake, Dart CSR interop fixture, 100% line coverage.
- Mobile `flutter test --coverage`: ML-DSA key/CSR structure and proof of possession, transport probe, fail-closed gating, 100% line coverage; `scripts/verify-pqc-csr-interop.sh` with OpenSSL.
- Runtime (`e2e` job): `smoke-tests`, `negative-mtls-tests` (incl. RSA / ML-DSA-44 client certs and classical-only key exchange), `pqc-handshake-tests` (negotiated group and peer signature on every hop; classical-only clients refused).

## Known limitation: mobile socket layer

`dart:io` delegates TLS to the platform BoringSSL build, which in Dart 3.11
rejects ML-DSA keys and certificates (`UNSUPPORTED_ALGORITHM`) and cannot offer
ML-DSA signature schemes. The app's identity, CSR, trust anchor and gating are
post-quantum, and the probe fails closed on that runtime; the device transport
itself needs a TLS engine with ML-DSA support (a Dart/Flutter BoringSSL update
or a native TLS plugin). Until then the local runtime's `smoke-tests` service
(curl + OpenSSL 3.5) exercises the exact mobile role (OTK, ML-DSA CSR
enrollment, mTLS banking calls) end to end.

## Cloud ingress

The Terraform paths deploy the terminator sidecar next to every internet-facing
gateway container. The deployment is post-quantum only where the cloud ingress
passes TLS through to the sidecar (AWS task ENI / NLB TCP passthrough, Azure
Container Apps TCP ingress); Cloud Run's managed HTTPS ingress terminates TLS at
Google's edge with classical certificates and needs a TCP passthrough in front.
