## ADDED Requirements

### Requirement: Gateway-only mobile communication
The mobile app SHALL communicate with backend capabilities only through the KrakenD API gateway.

#### Scenario: Mobile calls protected API
- **WHEN** the mobile app calls a protected Quantum Bank backend capability
- **THEN** the request goes to KrakenD rather than directly to a backend
  service
- **AND** backend service addresses are not exposed as the mobile runtime API
  base URL

#### Scenario: Direct backend bypass is attempted
- **WHEN** a caller attempts to access a protected backend endpoint directly
- **THEN** the request is rejected unless it satisfies the backend's own
  security requirements

### Requirement: OAuth2 protects gateway and backend
Protected APIs SHALL require OAuth2 bearer token validation at KrakenD and at the Spring Boot backend.

#### Scenario: Valid access token
- **WHEN** a request includes a valid access token with the expected issuer,
  audience, and claims
- **THEN** KrakenD accepts the edge request
- **AND** the backend resource server accepts the forwarded request

#### Scenario: Missing or invalid access token
- **WHEN** a request omits the access token or presents an invalid token
- **THEN** KrakenD rejects the request before forwarding it
- **AND** the backend also rejects equivalent direct protected requests

### Requirement: Mutual TLS is enforced for secure channels
The system SHALL use mTLS for app-to-gateway and gateway-to-backend communication where certificate material has been provisioned.

#### Scenario: Trusted client certificate is presented
- **WHEN** a client presents a certificate issued by the project trust chain
- **THEN** the TLS handshake succeeds
- **AND** the request can continue to OAuth2 and business validation

#### Scenario: Untrusted client certificate is presented
- **WHEN** a client presents no certificate or a certificate outside the
  project trust chain
- **THEN** the TLS handshake or gateway policy rejects the connection

### Requirement: TLS verification fails closed
The mobile app SHALL load trusted certificate authority material and SHALL NOT disable TLS verification with permissive certificate callbacks.

#### Scenario: Gateway certificate is trusted
- **WHEN** the gateway presents a certificate chaining to the configured project
  CA
- **THEN** the mobile TLS client accepts the connection

#### Scenario: Gateway certificate is not trusted
- **WHEN** the gateway presents an unknown or invalid certificate
- **THEN** the mobile TLS client rejects the connection
