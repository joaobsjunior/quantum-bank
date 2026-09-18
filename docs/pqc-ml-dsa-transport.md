# Transport Cryptography: Post-Quantum First, Compatibility at the App Edge

Every Quantum Bank communication that crosses a container or host boundary is
TLS 1.3 authenticated by a certificate the local PKI issued. The policy has two
tiers, chosen per listener class rather than per environment, so the same
configuration is production-viable on every platform the project targets:

| Tier | Where | Authentication | Key exchange |
| --- | --- | --- | --- |
| **Strict (post-quantum only)** | backend mTLS port; gateway → backend egress; gateway → issuer JWKS egress; `backend-client` → gateway and issuer; backend → issuer JWKS | **ML-DSA-65** leaves under an **ML-DSA-87** CA (FIPS 204), `mldsa65`/`mldsa87` schemes only | **`X25519MLKEM768`** only (FIPS 203 hybrid) |
| **App-facing (dual identity)** | issuer `:8443`, gateway bootstrap `:8080`, gateway banking `:8443` | dual server identity selected by the client's `signature_algorithms`: ML-DSA-65 for clients that only offer ML-DSA schemes, **ECDSA P-256** (compatibility chain, ECDSA P-384 CA) otherwise; client certificates from either chain; RSA and ML-DSA-44 refused | `X25519MLKEM768` preferred, `X25519` accepted |

ML-DSA is a signature algorithm: it authenticates certificates, CSRs and
handshakes. Confidentiality comes from the ML-KEM hybrid key exchange, which
every peer that can offer it negotiates on every hop, including the app edge.

## Why two tiers

The strict tier is what the project ran before, and it stays untouched where
both peers are under our control (OpenSSL 3.5 in the HAProxy terminators,
BouncyCastle BCJSSE in the JVM services, curl 8.16 / OpenSSL 3.5 in the test
clients).

The app edge is where the environments differ. These are the capabilities of
the TLS stacks the app edge must serve, verified in their sources:

| Client TLS stack | ML-DSA certificates | ML-DSA in `signature_algorithms` | `X25519MLKEM768` by default |
| --- | --- | --- | --- |
| `dart:io` on Dart 3.11 (Flutter 3.41, current pin) | rejected (`UNSUPPORTED_ALGORITHM`) | no | no (`X25519`, `P-256`, `P-384`) |
| `dart:io` on Dart 3.13 (Flutter 3.47) | parsed and X.509-verified (BoringSSL `EVP_PKEY_ML_DSA_*`) | no (BoringSSL `kVerifySignatureAlgorithms` is classical only, and `SecurityContext` cannot change it) | no (same default group list, not configurable from Dart) |
| Browsers (Keycloak login pages in production) | no | no | yes (Chrome, Firefox, Safari 26) |
| HAProxy 3.2 / OpenSSL 3.5, BCJSSE 1.86, curl 8.16 | yes | yes | yes |

A strict-only app edge therefore cannot be reached by the mobile app on any
Dart release, nor by a browser. A classical-only app edge would waste the
post-quantum capability of the services that also use it (the external
`backend-client`, the gateway JWKS egress). The dual identity serves both:
HAProxy 3.2 selects the certificate from the client's signature schemes (a
client that advertises ECDSA gets the ECDSA chain; a client that only offers
ML-DSA gets the ML-DSA chain, which is exactly what BCJSSE and the gateway
egress send), and OpenSSL selects the hybrid group whenever the client offers
it. This was validated locally with HAProxy 3.2.0 built against OpenSSL 3.5.8
before the configuration was committed.

The most compatible algorithms per environment are therefore:

- **ML-KEM-768 in the `X25519MLKEM768` hybrid** for key exchange everywhere:
  it is the only post-quantum group every stack above implements, and the
  hybrid keeps X25519 security if ML-KEM were weakened.
- **ML-DSA-65** for authentication wherever the peer verifies it (services,
  gateway egress, test clients), under an **ML-DSA-87** CA.
- **ECDSA P-256** under an **ECDSA P-384** CA for the app edge peers that
  cannot verify ML-DSA yet (Dart/BoringSSL, browsers). RSA is never used.

## Per-layer implementation

| Layer | Runtime | What it does |
| --- | --- | --- |
| `pki` | OpenSSL >= 3.5 (host or `alpine/openssl:3.5.8`) | Two CA hierarchies (`trust/root-ca.crt`, `trust/issuing-ca.crt` ML-DSA-87; `trust/root-ca-compat.crt`, `trust/issuing-ca-compat.crt` ECDSA P-384) that never cross-sign; ML-DSA-65 runtime identities for every service, ECDSA P-256 identities only for `gateway-server-compat` and `keycloak-server-compat`; union bundles (`ca-chain-all.crt`, `trust-anchors.crt`); `sign-csr.sh` accepts ML-DSA-65/87 or ECDSA P-256 CSR keys, issues under the chain of the key family and writes `<leaf>.issuer`; negative fixtures; handshake evidence script for both client classes |
| `api-gateway` | KrakenD 2.13 (loopback only) + HAProxy 3.2 / OpenSSL 3.5 | App-facing binds: `crt gateway-server-compat.pem crt gateway-server.pem`, `curves X25519MLKEM768:X25519`, `sigalgs`/`client-sigalgs` `mldsa65:mldsa87:ecdsa_secp256r1_sha256:ecdsa_secp384r1_sha384`, banking `ca-file ca-chain-all.crt verify required`; global (egress) defaults stay strict; `verify-pqc-gateway.sh` enforces both |
| `infrastructure` | Compose, Terraform | `keycloak-tls` dual identity; smoke, negative and handshake test services exercise both device roles (ML-DSA-65 and ECDSA P-256 client certificates and enrollment CSRs); `pqc_transport` Terraform policy carries the strict and the `compat_*` values; the cloud ingress must pass TLS through |
| `backend` | Spring Boot on JDK 17 + BCJSSE | Strict in-process TLS unchanged; `CsrValidator` accepts ML-DSA-65/87 and ECDSA P-256 (`secp256r1` named curve only) and rejects RSA, EdDSA, ML-DSA-44 and other curves; `ScriptPkiAdapter` returns the chain the sign script selected |
| `backend-client` | Spring Boot on JDK 17 + BCJSSE | Strict, unchanged: offers ML-DSA only, so the dual listeners always serve it the ML-DSA identity |
| `mobile-app` | Flutter 3.41 / Dart 3.11 | Startup probe selects `TransportMode.postQuantum` (ML-DSA-65 identity, seed-only PKCS#8 per RFC 9881, both roots trusted) or `TransportMode.compatibility` (ECDSA P-256 identity, ECDSA root trusted); the mode is shown to the user; protected access opens in either mode once the device certificate is ready |

Loopback traffic between a terminator and the process it fronts never leaves
the container network namespace and is the equivalent of an in-process hop.

The compatibility certificate is listed first on every app-facing bind, so it
is also the default for clients that send no SNI (a physical device reaching
the gateway by LAN IP). ML-DSA-only clients must connect by hostname, which
every service does.

## Evidence gates

- `pki/scripts/ci-validate.sh`: both CA chains, no cross-signing, ML-DSA-65
  identities for every service and no compatibility identity for any of them,
  ECDSA P-256 identities for the app-facing servers only, PKCS#12 stores,
  negative fixtures with their forbidden algorithms.
- `api-gateway/scripts/ci-validate.sh`: `krakend check`, `haproxy -c` with
  generated material of both chains, `verify-pqc-gateway.sh` (strict egress
  defaults, dual-identity app-facing binds, no RSA anywhere).
- `infrastructure/scripts/verify-local-e2e-config.sh`: terminator topology,
  loopback bindings, dual-identity issuer terminator, both device roles in the
  smoke services.
- Backend / backend-client `gradle check`: real loopback ML-DSA mutual
  handshakes over `X25519MLKEM768`, RSA identity and anonymous client refused
  inside the handshake, Dart CSR interop fixtures for both key families, 100%
  line coverage.
- Mobile `flutter test --coverage`: both key families (structure, PKCS#8
  encodings, proof of possession), transport-mode selection and trust anchor
  selection, gate screen in both modes, 100% line coverage;
  `scripts/verify-pqc-csr-interop.sh` proves both CSRs and both PKCS#8 keys
  with OpenSSL 3.5.
- Runtime (`e2e` job): `smoke-tests` (ML-DSA and ECDSA device certificates,
  both enrollments, classical-group banking call), `negative-mtls-tests`
  (RSA / ML-DSA-44 / untrusted-compat client certificates refused on the
  banking listener; classical groups and compatibility identities refused on
  the backend), `pqc-handshake-tests` (negotiated group and peer signature per
  client class on every hop; RSA refused everywhere; classical-only clients
  refused on the strict hop).

## Application-layer tokens

JWTs stay RS256: Keycloak and KrakenD have no ML-DSA JWS. Tokens only travel
inside the TLS sessions described above.

## Cloud ingress

The Terraform paths deploy the terminator sidecar next to every internet-facing
gateway container. The hybrid key exchange, the ML-DSA identities and the
mutual TLS of the banking listener reach the client only where the cloud
ingress passes TLS through to the sidecar (AWS task ENI / NLB TCP passthrough,
Azure Container Apps TCP ingress); Cloud Run's managed HTTPS ingress terminates
TLS at Google's edge with classical certificates and needs a TCP passthrough in
front.

## Path to a fully post-quantum app edge

The compatibility tier exists because of the client stacks, not the servers.
When a Dart/Flutter release offers ML-DSA in `signature_algorithms` and
`X25519MLKEM768` in its default groups (or exposes them on `SecurityContext`),
the app's startup probe succeeds, the device enrolls an ML-DSA-65 identity, the
dual listeners serve it the ML-DSA chain, and no server-side change is needed.
The compatibility chain can then be retired from the gateway binds and kept for
browsers on the issuer only.
