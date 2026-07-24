## ADDED Requirements

### Requirement: Gateway-only external service communication
The `backend-client` external service SHALL reach Quantum Bank banking
capabilities only through the KrakenD API gateway, never by calling a backend
service address directly.

#### Scenario: External service calls a protected capability
- **WHEN** the `backend-client` invokes a protected Quantum Bank capability
- **THEN** the request is sent to the KrakenD gateway origin
- **AND** no backend service address is configured or used as the outbound base
  URL

#### Scenario: Direct-to-backend configuration is rejected
- **WHEN** the `backend-client` is configured with a base URL that resolves to a
  backend service instead of the gateway
- **THEN** the configuration is treated as invalid and the service fails to
  start or refuses to issue the call

### Requirement: Service authenticates with its own OAuth2 client credentials
The `backend-client` SHALL authenticate as its own registered OAuth2 client
using the client-credentials grant and present the resulting bearer token to the
gateway; it MUST NOT reuse the mobile client identity or any end-user
credential.

#### Scenario: Service obtains and presents a token
- **WHEN** the `backend-client` prepares an outbound banking call
- **THEN** it obtains an access token from the OAuth2 issuer via the
  client-credentials grant for its own client
- **AND** it sends that token as a bearer credential to the gateway

#### Scenario: Missing or invalid token is fail-closed
- **WHEN** the `backend-client` has no valid access token
- **THEN** it does not send the protected request
- **AND** it surfaces an authentication error rather than calling without a
  token

### Requirement: Service presents a PKI-issued mTLS identity on the banking listener
The `backend-client` SHALL present a PKI-issued service client certificate for
mutual TLS on the gateway banking listener, and SHALL fail closed when the
certificate or trust material is missing or invalid.

#### Scenario: Banking call uses the service certificate
- **WHEN** the `backend-client` calls a banking-listener capability
- **THEN** it completes the mTLS handshake using its PKI-issued service client
  certificate
- **AND** it validates the gateway certificate against the configured trust
  anchors

#### Scenario: Missing certificate material blocks the call
- **WHEN** the service client certificate or trust anchors are absent or invalid
- **THEN** the `backend-client` does not complete the banking call
- **AND** it does not fall back to a permissive or certificate-ignoring TLS mode

### Requirement: External service exercises the banking flows
The `backend-client` SHALL be able to exercise the Pix transfer (with
app-selected success and error simulation), account statement, and profile /
registration-data capabilities through the gateway, as an external caller.

#### Scenario: Pix transfer with selected scenario
- **WHEN** the `backend-client` submits a Pix transfer selecting a success or
  error scenario
- **THEN** the request is routed through the gateway to the backend Pix
  simulation
- **AND** the simulated success or error outcome is returned to the
  `backend-client`

#### Scenario: Statement and profile retrieval
- **WHEN** the `backend-client` requests the account statement or profile data
- **THEN** the gateway returns the deterministic simulated response from the
  backend
- **AND** the response is associated with the request's correlation id

### Requirement: External service propagates correlation and handles problem details
The `backend-client` SHALL propagate a `correlationId` on outbound requests and
SHALL parse RFC 9457 problem-details error responses returned through the
gateway.

#### Scenario: Correlation id is propagated
- **WHEN** the `backend-client` issues a banking request
- **THEN** the request carries a `correlationId`
- **AND** the same correlation id is used to correlate the response

#### Scenario: Problem-details error is parsed
- **WHEN** the gateway or backend returns an RFC 9457 problem-details error
- **THEN** the `backend-client` parses the problem-details body
- **AND** it exposes the error type, status, and detail rather than a raw or
  swallowed failure
