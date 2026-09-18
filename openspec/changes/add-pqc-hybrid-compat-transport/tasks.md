## 1. PKI

- [x] 1.1 `scripts/lib/openssl-pqc.sh`: compatibility policy constants, `pqc_key_family`, `pqc_require_family`, `pqc_ensure_ec_key`
- [x] 1.2 `bootstrap-local-ca.sh`: ECDSA P-384 root and issuing CA (`trust/*-compat.crt`), mobile root assets kept in sync
- [x] 1.3 `bootstrap-runtime-certs.sh`: ECDSA P-256 identities for `gateway-server-compat`, `keycloak-server-compat`, `mobile-smoke-client-compat`; `mobile-smoke-enroll-compat.csr`; `ca-chain-compat.crt`, `ca-chain-all.crt`, `trust-anchors.crt`; `untrusted-compat-client` fixture
- [x] 1.4 `sign-csr.sh`: ML-DSA-65/87 or ECDSA P-256 keys, chain by family, `<leaf>.issuer`
- [x] 1.5 `verify-trust-anchors.sh`, `verify-runtime-certs.sh`: both chains, no cross-signing, no compatibility service identity
- [x] 1.6 `pqc-handshake-tests.sh`: post-quantum and compatibility client classes per listener, RSA refused, strict backend refusals; `negative-mtls-tests.sh` updated

## 2. Gateway

- [x] 2.1 `tls/haproxy-bootstrap.cfg`, `tls/haproxy-banking.cfg`: dual-identity app-facing binds, strict global defaults and egress
- [x] 2.2 `verify-pqc-gateway.sh`: widened values only on the app-facing binds; `ci-validate.sh` generates both chains for `haproxy -c`

## 3. Infrastructure

- [x] 3.1 `keycloak/haproxy-keycloak.cfg` dual identity; Compose test services carry both device roles and trust bundles
- [x] 3.2 `local-e2e-smoke.sh`: ECDSA enrollment, ECDSA banking calls (hybrid and classical group), compatibility identity refused by the backend
- [x] 3.3 `verify-local-e2e-config.sh`, Terraform `pqc_transport` compatibility values, READMEs

## 4. Backend

- [x] 4.1 `CsrValidator`: ECDSA P-256 named curve accepted, other curves/explicit parameters rejected, `keyAlgorithmName` families
- [x] 4.2 `ScriptPkiAdapter`: chain from `<leaf>.issuer`; tests and ECDSA Dart interop fixture; Kover 100%

## 5. backend-client

- [x] 5.1 Documented as a strict ML-DSA-only peer (no code change)

## 6. Mobile

- [x] 6.1 `TransportMode`, probe selects the mode, `trustAnchorsFor`
- [x] 6.2 `DeviceKeyPair` (ML-DSA-65 seed-only PKCS#8, ECDSA P-256 PKCS#8/RFC 5915), algorithm-agnostic `CsrService`, orchestrator by mode
- [x] 6.3 Gate screen shows the mode; compatibility root asset; tests (100% coverage); `verify-pqc-csr-interop.sh` for both families

## 7. Superproject

- [x] 7.1 `docs/pqc-ml-dsa-transport.md`, README, AGENTS constraints
