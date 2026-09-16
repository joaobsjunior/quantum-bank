## MODIFIED Requirements

### Requirement: Service presents a PKI-issued mTLS identity on the banking listener
The `backend-client` SHALL present its PKI-issued ML-DSA-65 service client
certificate for mutual TLS on the gateway banking listener over TLS 1.3
negotiated by a post-quantum JSSE provider (`mldsa65`/`mldsa87` signature
schemes, `X25519MLKEM768`), SHALL validate the gateway and issuer against the
ML-DSA-87 trust anchors, and SHALL fail closed when the certificate or trust
material is missing or invalid, never using a permissive TLS mode or a
classical fallback.

#### Scenario: Banking call uses the service certificate
- **WHEN** the `backend-client` calls a banking-listener capability
- **THEN** it completes the mTLS handshake with `X25519MLKEM768` using its
  PKI-issued ML-DSA-65 service client certificate
- **AND** it validates the gateway certificate against the configured trust
  anchors

#### Scenario: Missing certificate material blocks the call
- **WHEN** the service client certificate or trust anchors are absent or invalid
- **THEN** the `backend-client` does not complete the banking call
- **AND** it does not fall back to a permissive or certificate-ignoring TLS mode

#### Scenario: Service has a classical certificate
- **WHEN** the service keystore carries an RSA identity
- **THEN** the handshake fails and no request reaches the gateway

