## MODIFIED Requirements

### Requirement: External service speaks post-quantum mTLS
The external service (`backend-client`) SHALL remain a strict post-quantum
peer: TLS 1.3, `X25519MLKEM768` only, `mldsa65`/`mldsa87` only, the ML-DSA-65
service identity and the ML-DSA-87 anchors; because it offers ML-DSA schemes
only, the dual-identity gateway and issuer listeners SHALL serve it the ML-DSA
certificate.

#### Scenario: Service connects to the gateway
- **WHEN** `backend-client` opens a session to the banking listener or the
  issuer
- **THEN** the negotiated group is `X25519MLKEM768` and the peer signature is
  ML-DSA over the post-quantum chain
