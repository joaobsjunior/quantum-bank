## ADDED Requirements

### Requirement: Mobile uses gateway-named origins only
The mobile app SHALL configure only gateway-named origins for protected APIs —
`GATEWAY_BOOTSTRAP_BASE_URL` for OAuth2 bootstrap calls and `GATEWAY_BASE_URL`
for certificate-ready banking calls — and SHALL NOT configure backend service
hosts as protected origins. Forbidden origin strings include `BACKEND_BASE_URL`,
`backend:`, `localhost:8081`, and `http://backend`; the source and config SHALL
pass `scripts/verify-gateway-only.sh`.

#### Scenario: Protected banking origin
- **WHEN** the app resolves the base URL for a protected banking call
- **THEN** it uses `GATEWAY_BASE_URL` pointing at the KrakenD banking listener
- **AND** no backend host, container alias, or backend port is used

#### Scenario: Gateway-only guard runs
- **WHEN** `scripts/verify-gateway-only.sh` runs against the mobile source and
  config
- **THEN** it fails if any forbidden backend origin string is present

### Requirement: Mobile client preconditions for protected calls
The mobile API client SHALL call protected banking APIs only when
`authenticated`, `certificateReady`, and `gatewayBaseUrlConfigured` are all true;
bootstrap calls MAY occur before `certificateReady` when the route contract
allows OAuth2 bearer authentication without mobile client mTLS.

#### Scenario: Protected call before certificate-ready
- **WHEN** the app attempts a protected banking call while `certificateReady` is
  false
- **THEN** the client does not issue the protected call

#### Scenario: Bootstrap call before certificate-ready
- **WHEN** the app performs an OAuth2 bootstrap call while `certificateReady` is
  false
- **THEN** the call is allowed because the bootstrap route does not require
  mobile client mTLS

### Requirement: Mobile parses problem details into a stable error model
The mobile client SHALL parse `application/problem+json` responses into a stable
model with `type`, `title`, `status`, `errorCode`, `correlationId`, and optional
`detail`, `instance`, and `fieldErrors`, and SHALL NOT display token values,
private key material, CSR internals, backend hostnames, or PKI internals.

#### Scenario: Problem response is parsed
- **WHEN** the app receives an `application/problem+json` error
- **THEN** it maps the response into the stable error model
- **AND** it displays only safe fields, never tokens, keys, CSR, or backend hosts

### Requirement: Pix scenario is app-selected, not inferred
The Pix transfer flow SHALL send an explicit `scenario` value (`SUCCESS` or
`ERROR`) chosen by the app, and SHALL NOT infer Pix errors from random network
behavior during local-v1 testing.

#### Scenario: App selects a Pix scenario
- **WHEN** the app submits a Pix transfer
- **THEN** it sends an explicit `SUCCESS` or `ERROR` scenario
- **AND** it does not treat incidental network failures as the Pix error path
