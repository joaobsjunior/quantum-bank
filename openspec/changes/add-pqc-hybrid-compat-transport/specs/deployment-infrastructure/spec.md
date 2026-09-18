## MODIFIED Requirements

### Requirement: Local runtime transport is post-quantum on every hop
The Compose runtime SHALL front Keycloak and both KrakenD listeners with
HAProxy + OpenSSL 3.5 terminators sharing their network namespace, SHALL keep
the paired processes on loopback, and SHALL configure the terminators'
app-facing binds as dual-identity listeners (compatibility certificate first,
ML-DSA certificate second, `X25519MLKEM768:X25519`, ML-DSA and ECDSA signature
schemes, no RSA) while keeping every egress strict.

#### Scenario: Runtime evidence
- **WHEN** the `smoke`, `negative-mtls` and `pqc-handshake` services run
- **THEN** both device roles (ML-DSA-65 and ECDSA P-256 certificates and
  enrollment CSRs) complete the banking flow, RSA / ML-DSA-44 /
  untrusted-compatibility client certificates are refused on the banking
  listener, classical-only groups and compatibility identities are refused on
  the backend port, and every listener reports the expected group and peer
  signature per client class

### Requirement: Cloud paths carry the transport policy
The Terraform runtime conventions SHALL expose the strict transport policy and
the compatibility values the app-facing listeners additionally serve, and the
cloud ingress SHALL pass TLS through to the terminator sidecar.

#### Scenario: Policy variable
- **WHEN** `pqc_transport` is read
- **THEN** it carries `X25519MLKEM768`, `mldsa65`/`mldsa87`, ML-DSA-87 CA,
  ML-DSA-65 leaves and the `compat_*` key exchange, signature schemes and
  ECDSA CA/leaf algorithms
