## ADDED Requirements

### Requirement: Every network TLS hop is post-quantum only
Every TLS socket that crosses a container or host boundary (app-to-issuer,
app-to-gateway, gateway-to-issuer, gateway-to-backend, external-service-to-gateway,
external-service-to-issuer) SHALL negotiate TLS 1.3 with the `X25519MLKEM768`
hybrid key-exchange group and authenticate peers with ML-DSA certificates using
the `mldsa65` or `mldsa87` signature schemes, and SHALL refuse classical
signature schemes and classical-only key-exchange groups.

#### Scenario: Post-quantum client connects
- **WHEN** a client offers `X25519MLKEM768` and ML-DSA signature schemes to
  any listener
- **THEN** the handshake completes with `X25519MLKEM768` as the negotiated group
  and an ML-DSA peer signature over a certificate chained to the ML-DSA-87 CA

#### Scenario: Classical-only client connects
- **WHEN** a client offers only classical signature schemes (RSA, ECDSA, EdDSA)
  or only classical groups (X25519, secp256r1, secp384r1)
- **THEN** the handshake fails before any HTTP byte is exchanged

### Requirement: Components without post-quantum TLS are fronted by a terminator in their network namespace
A component whose runtime cannot negotiate ML-DSA (KrakenD, Keycloak) SHALL
listen on the loopback interface only and SHALL be paired with a post-quantum
TLS terminator sharing its network namespace that owns every socket the
component exposes to or opens towards the network.

#### Scenario: KrakenD listener
- **WHEN** the gateway container starts
- **THEN** KrakenD binds `127.0.0.1` only and the terminator publishes the
  bootstrap and banking ports with the PKI-issued ML-DSA-65 server certificate
- **AND** the banking terminator requires an ML-DSA client certificate chained
  to the local PKI

#### Scenario: KrakenD reaches the backend or the issuer
- **WHEN** KrakenD forwards a request to the backend or retrieves the JWK set
- **THEN** it uses a loopback egress of the terminator, which presents the
  `gateway-client` ML-DSA identity to the backend and verifies the issuer's
  ML-DSA certificate

## MODIFIED Requirements

### Requirement: Mutual TLS is enforced for secure channels
The system SHALL use post-quantum mTLS (ML-DSA certificates, `X25519MLKEM768`)
for app-to-gateway and gateway-to-backend communication where certificate
material has been provisioned.

#### Scenario: Trusted client certificate is presented
- **WHEN** a client presents an ML-DSA-65 or ML-DSA-87 certificate issued by
  the project trust chain and signs the handshake with an accepted scheme
- **THEN** the TLS handshake succeeds
- **AND** the request can continue to OAuth2 and business validation

#### Scenario: Untrusted client certificate is presented
- **WHEN** a client presents no certificate, a certificate outside the project
  trust chain, an expired certificate, or a certificate the trust chain signed
  for a classical (RSA/EC) or ML-DSA-44 key
- **THEN** the TLS handshake rejects the connection

### Requirement: Gateway-to-backend mTLS is a distinct enforced boundary
Gateway-to-backend traffic SHALL be protected by post-quantum mTLS on every
configured hop, including the bootstrap routes, as a boundary separate from
app-to-gateway mTLS; the backend SHALL terminate it in-process with a
post-quantum JSSE provider (TLS 1.3, ML-DSA-65 server certificate, ML-DSA
client authentication required).

#### Scenario: Gateway forwards to backend over mTLS
- **WHEN** the gateway forwards any accepted request to the backend
- **THEN** its terminator uses the `gateway-client` ML-DSA certificate over a
  TLS 1.3 `X25519MLKEM768` connection

#### Scenario: Direct backend call without gateway certificate
- **WHEN** a caller reaches the backend mTLS port without a trusted ML-DSA
  gateway client certificate
- **THEN** the connection fails at the TLS handshake

