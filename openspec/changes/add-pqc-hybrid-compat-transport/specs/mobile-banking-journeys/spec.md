## MODIFIED Requirements

### Requirement: Mobile client preconditions for protected calls
The mobile API client SHALL call protected banking APIs only when
`authenticated`, `certificateReady` and `gatewayBaseUrlConfigured` are all
true. At startup the app SHALL probe the platform TLS stack for ML-DSA
support and select a transport mode: `postQuantum` (ML-DSA-65 identity, both
PKI roots trusted) when the stack loads ML-DSA material, `compatibility`
(ECDSA P-256 identity, compatibility root trusted) otherwise. The mode SHALL
be shown to the user and SHALL never block protected access by itself.

#### Scenario: Post-quantum transport available
- **WHEN** the probe loads the bundled ML-DSA-87 root anchor successfully
- **THEN** the gate screen states the post-quantum transport, the device
  enrolls an ML-DSA-65 identity and protected screens open once authentication
  and enrollment complete

#### Scenario: Post-quantum transport unavailable
- **WHEN** the probe reports that the TLS stack rejects ML-DSA material
- **THEN** the gate screen states the compatibility transport with the
  platform reason, both actions stay enabled, the device enrolls an ECDSA
  P-256 identity and protected screens open once authentication and
  enrollment complete
