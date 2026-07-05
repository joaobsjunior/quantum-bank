## ADDED Requirements

### Requirement: Gateway exposes only the published app-facing route surface
KrakenD SHALL expose only the published app-facing routes (`POST /auth/otk`,
`POST /auth/csr`, `POST /pix/transfers`, `GET /statements`, `GET /profile`, and
`PUT /profile`) sourced from `api-gateway/openapi/quantum-bank-v1.yaml`, and
SHALL NOT expose backend hostnames, ports, or internal paths as the mobile
runtime origin.

#### Scenario: Mobile resolves an app-facing route
- **WHEN** the mobile app resolves a protected banking route
- **THEN** the route is one of the published gateway paths
- **AND** no backend service host, port, or internal path is used as the origin

#### Scenario: Unpublished route is requested
- **WHEN** a caller requests a path that is not part of the published gateway
  surface
- **THEN** the gateway does not route it to a backend service

### Requirement: Gateway enforces per-endpoint OAuth2 scopes
KrakenD SHALL require the OAuth2 scopes defined for each protected route before
routing to the backend: `openid`+`profile` for `/auth/otk` and `/auth/csr`,
`pix:write` for `POST /pix/transfers`, `statements:read` for `GET /statements`,
and `profile:read` for `GET /profile`.

#### Scenario: Request carries the required scope
- **WHEN** a request to `POST /pix/transfers` carries a valid token with the
  `pix:write` scope
- **THEN** the gateway routes the request to the backend

#### Scenario: Request is missing the required scope
- **WHEN** a request to `POST /pix/transfers` carries a valid token without the
  `pix:write` scope
- **THEN** the gateway rejects it with `403` and `application/problem+json`
- **AND** the request is not routed to the backend

### Requirement: Gateway rejects invalid tokens fail-closed
KrakenD SHALL reject a request before backend routing when the bearer token is
missing, malformed, expired, or presents the wrong issuer or audience, returning
`401 application/problem+json`; a valid token that lacks the endpoint scope SHALL
return `403 application/problem+json`.

#### Scenario: Missing or invalid token
- **WHEN** a request omits the token or presents a malformed, expired,
  wrong-issuer, or wrong-audience token
- **THEN** the gateway returns `401` with `application/problem+json`
- **AND** the request is not forwarded to the backend

#### Scenario: Valid token accepted
- **WHEN** a request carries a token with the expected issuer
  (`quantum-bank-local` realm), audience `quantum-bank-api`, `RS256` signature,
  and required scope
- **THEN** the gateway accepts and routes the request

### Requirement: Gateway uses a two-listener mTLS topology
The gateway SHALL run a bootstrap listener that is OAuth2-only (no app client
certificate) for `POST /auth/otk` and `POST /auth/csr`, and a banking listener
that requires both OAuth2 and app-to-gateway mTLS for `POST /pix/transfers`,
`GET /statements`, and `GET /profile`.

#### Scenario: Bootstrap call before certificate-ready state
- **WHEN** the app calls `POST /auth/otk` or `POST /auth/csr` before it has a
  client certificate
- **THEN** the bootstrap listener accepts the OAuth2-authenticated request
  without requiring an app client certificate

#### Scenario: Banking call requires client certificate
- **WHEN** the app calls a protected banking route on the banking listener
- **THEN** the listener requires a trusted mobile client certificate in addition
  to OAuth2
- **AND** a missing, untrusted, expired, or wrong-environment client certificate
  fails at the TLS handshake rather than returning a routed HTTP response

### Requirement: Gateway-to-backend mTLS is a distinct enforced boundary
Gateway-to-backend traffic SHALL be protected by mTLS on every configured hop,
including the bootstrap routes, as a boundary separate from app-to-gateway mTLS.

#### Scenario: Gateway forwards to backend over mTLS
- **WHEN** the gateway forwards any accepted request to the backend
- **THEN** it uses its gateway client certificate over an mTLS connection

#### Scenario: Direct backend call without gateway certificate
- **WHEN** a caller reaches the backend mTLS port without a trusted gateway
  client certificate
- **THEN** the connection fails at the TLS handshake

### Requirement: Backend independently validates OAuth2 and does not trust gateway headers
The backend SHALL validate the OAuth2 bearer token on every protected route as
defense in depth, derive user identity only from the validated JWT, and SHALL
NOT treat gateway-forwarded identity, scope, or certificate headers as
authentication.

#### Scenario: Direct protected request to the backend
- **WHEN** a protected backend route receives a request without a valid bearer
  token
- **THEN** the backend rejects it regardless of any forwarded gateway headers

#### Scenario: Forwarded headers do not replace token validation
- **WHEN** a request carries gateway-forwarded subject, scope, or certificate
  metadata
- **THEN** the backend uses that metadata only after independently validating the
  JWT, and derives user identity from the JWT

### Requirement: Authorization failures use RFC 9457 problem details
Gateway and backend authentication and authorization failures SHALL use RFC 9457
problem details with media type `application/problem+json`, including stable
`status`, `title`, `errorCode`, and `correlationId` fields, and SHALL NOT expose
token values, backend hostnames, stack traces, route internals, or PKI internals.

#### Scenario: Authorization failure response shape
- **WHEN** the gateway or backend rejects a request for auth reasons
- **THEN** the response media type is `application/problem+json`
- **AND** it includes `status`, `title`, `errorCode`, and `correlationId`
- **AND** it does not include token values, backend hostnames, stack traces, or
  PKI internals
