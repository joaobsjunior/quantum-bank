## MODIFIED Requirements

### Requirement: Every network TLS hop is post-quantum only
Every TLS socket that crosses a container or host boundary SHALL be TLS 1.3
authenticated by a certificate the project PKI issued, in one of two tiers.
Strict hops (gateway-to-backend, gateway-to-issuer, external-service-to-gateway,
external-service-to-issuer, backend-to-issuer) SHALL negotiate only the
`X25519MLKEM768` hybrid group and authenticate peers only with ML-DSA
certificates using the `mldsa65` or `mldsa87` signature schemes. App-facing
listeners (issuer, gateway bootstrap, gateway banking) SHALL serve a dual
identity selected by the client's signature schemes (the ML-DSA-65 certificate
to clients that only offer ML-DSA schemes, the ECDSA P-256 compatibility
certificate otherwise), SHALL prefer `X25519MLKEM768` and accept `X25519`, and
SHALL refuse RSA signature schemes.

#### Scenario: Post-quantum client connects to any listener
- **WHEN** a client offers `X25519MLKEM768` and only ML-DSA signature schemes
- **THEN** the handshake completes with `X25519MLKEM768` and an ML-DSA peer
  signature over a certificate chained to the ML-DSA-87 CA

#### Scenario: Compatibility client connects to an app-facing listener
- **WHEN** a client offers ECDSA signature schemes (with or without ML-DSA)
- **THEN** the handshake completes with the ECDSA P-256 compatibility
  certificate chained to the ECDSA P-384 CA, using `X25519MLKEM768` when the
  client offers it and `X25519` otherwise

#### Scenario: Classical-only client connects to a strict hop
- **WHEN** a client offers only classical signature schemes or only classical
  groups to the backend port
- **THEN** the handshake fails before any HTTP byte is exchanged

#### Scenario: RSA client connects anywhere
- **WHEN** a client offers only RSA signature schemes to any listener
- **THEN** the handshake fails

### Requirement: Mutual TLS is enforced for secure channels
The system SHALL use mTLS for app-to-gateway and gateway-to-backend
communication where certificate material has been provisioned: the banking
listener SHALL accept client certificates from either PKI chain signed with an
accepted scheme, and the backend SHALL accept only ML-DSA client certificates
from the post-quantum chain.

#### Scenario: Trusted client certificate is presented
- **WHEN** a client presents an ML-DSA-65/87 certificate from the post-quantum
  chain or an ECDSA P-256 certificate from the compatibility chain to the
  banking listener and signs the handshake with an accepted scheme
- **THEN** the TLS handshake succeeds

#### Scenario: Untrusted client certificate is presented
- **WHEN** a client presents no certificate, a certificate outside both project
  chains, an expired certificate, or a certificate a project chain signed for
  an RSA or ML-DSA-44 key
- **THEN** the TLS handshake rejects the connection

#### Scenario: Compatibility identity reaches the backend
- **WHEN** a client presents an ECDSA compatibility certificate to the backend
  port
- **THEN** the TLS handshake rejects the connection
