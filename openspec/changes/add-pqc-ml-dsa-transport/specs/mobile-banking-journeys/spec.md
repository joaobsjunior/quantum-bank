## MODIFIED Requirements

### Requirement: Mobile client preconditions for protected calls
The mobile API client SHALL call protected banking APIs only when
`authenticated`, `certificateReady`, `gatewayBaseUrlConfigured` and
`pqcTransportSupported` (the platform TLS stack can load ML-DSA certificate
material, probed at startup) are all true; bootstrap calls MAY occur before
`certificateReady` when the route contract allows OAuth2 bearer authentication
without mobile client mTLS. When the probe finds the TLS stack cannot load
ML-DSA material, the app SHALL keep protected screens closed and show the
reason instead of attempting a classical handshake.

#### Scenario: Protected call before certificate-ready
- **WHEN** the app attempts a protected banking call while `certificateReady` is
  false
- **THEN** the client does not issue the protected call

#### Scenario: Bootstrap call before certificate-ready
- **WHEN** the app performs an OAuth2 bootstrap call while `certificateReady` is
  false
- **THEN** the call is allowed because the bootstrap route does not require
  mobile client mTLS

#### Scenario: Post-quantum transport available
- **WHEN** the probe loads the bundled ML-DSA-87 root anchor as a certificate
  chain successfully
- **THEN** authentication and certificate activation are offered and protected
  screens open once both complete

#### Scenario: Post-quantum transport unavailable
- **WHEN** the probe reports that the TLS stack rejects ML-DSA material
- **THEN** the gate screen states that post-quantum transport is unavailable
  with the platform error, both actions stay disabled, and no banking call is
  attempted

