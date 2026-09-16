## 1. PKI (post-quantum only)

- [x] 1.1 Shared OpenSSL >= 3.5 resolver (`scripts/lib/openssl-pqc.sh`): host binary or pinned `alpine/openssl` container, fail closed otherwise
- [x] 1.2 ML-DSA-87 root and issuing CA; ML-DSA-65 runtime certificates, PEM bundles for HAProxy, PKCS#12 stores via OpenSSL (`-jdktrust`), no keytool
- [x] 1.3 `sign-csr.sh` accepts ML-DSA-65/87 CSR keys only and asserts ML-DSA-87 issuer signatures
- [x] 1.4 Negative fixtures: RSA and ML-DSA-44 client certificates signed by the real CA; negative mTLS matrix extended with classical-only key exchange
- [x] 1.5 `verify-trust-anchors.sh`, `verify-runtime-certs.sh`, `pqc-handshake-tests.sh`; CI gate runs the full PQC bootstrap

## 2. Gateway

- [x] 2.1 KrakenD binds `127.0.0.1` only, no `tls`/`client_tls`, loopback backend and JWKS egress
- [x] 2.2 HAProxy terminators (`tls/haproxy-bootstrap.cfg`, `tls/haproxy-banking.cfg`, `Dockerfile.tls`): TLS 1.3, `X25519MLKEM768`, `mldsa65:mldsa87`, banking `verify required`, mTLS egress to the backend
- [x] 2.3 `verify-pqc-gateway.sh` static policy and `ci-validate.sh` HAProxy config check with generated ML-DSA material

## 3. Infrastructure

- [x] 3.1 Compose: `keycloak-tls`, `gateway-bootstrap-tls`, `gateway-banking-tls` sidecars; Keycloak HTTP on loopback with `--proxy-headers`
- [x] 3.2 `pqc-handshake-tests` service; negative fixtures wired; `verify-local-e2e-config.sh` enforces the topology
- [x] 3.3 Terraform: terminator image/sidecar in the runtime conventions and cloud paths, passthrough notes

## 4. Backend

- [x] 4.1 BouncyCastle `bcprov`/`bcutil`/`bctls` 1.86; `PostQuantumTls` installer wired before `runApplication` and as `@Configuration`
- [x] 4.2 Tomcat TLS 1.3 only; `CsrValidator` ML-DSA-65/87 key policy; Alpine runtime image with OpenSSL 3.5 for the sign script
- [x] 4.3 Tests: ML-DSA test crypto, loopback ML-DSA mTLS handshake with classical rejection, Dart CSR interop fixture; Kover 100%

## 5. backend-client

- [x] 5.1 BCJSSE `MutualTlsClientFactory` (TLS 1.3, ML-DSA identity), installer wiring, ML-DSA test material and handshake tests; Kover 100%

## 6. Mobile

- [x] 6.1 `pqcrypto` ML-DSA-65 key pair, PKCS#8 encoding, PKCS#10 CSR with ML-DSA proof of possession
- [x] 6.2 `PqcTlsSupport` startup probe and fail-closed gating in `QuantumBankAppState` and the gate screen
- [x] 6.3 Tests (100% coverage), `tool/emit_ml_dsa_csr.dart`, `scripts/verify-pqc-csr-interop.sh`, ML-DSA-87 root asset

## 7. Superproject

- [x] 7.1 README, AGENTS, CI e2e job (PQC bootstrap, handshake tests, CSR interop), `docs/pqc-ml-dsa-transport.md`
