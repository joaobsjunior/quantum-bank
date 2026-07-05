## ADDED Requirements

### Requirement: Local runtime provides the required service topology
The local runtime SHALL provide `api-gateway`, `backend`, `pki`, an
`oauth2-issuer`, and an H2-backed backend runtime, wired so mobile traffic enters
through the gateway, the backend owns business and OTK logic, PKI owns
certificate lifecycle, and the OAuth2 issuer provides bearer tokens for gateway
and backend validation.

#### Scenario: Local topology is brought up
- **WHEN** the local runtime is started for end-to-end validation
- **THEN** the gateway, backend, PKI, OAuth2 issuer, and H2-backed backend are
  available
- **AND** mobile traffic reaches the backend only through the gateway

### Requirement: Local OAuth2 issuer is Keycloak with a defined realm and clients
The local-v1 OAuth2/OIDC issuer SHALL be Keycloak running from the infrastructure
Compose file with the imported realm `quantum-bank-local`, exposing a
host-facing issuer at `http://localhost:8180/realms/quantum-bank-local` and a
container-network issuer at `http://keycloak:8080/realms/quantum-bank-local`, a
public mobile client `quantum-bank-mobile` using Authorization Code + PKCE
(`S256`, direct access grants disabled), an isolated local test client
`quantum-bank-test` that MUST NOT appear in mobile configuration, and audience
`quantum-bank-api`. Keycloak MUST NOT use host port `8080`.

#### Scenario: Mobile authenticates against the local issuer
- **WHEN** the mobile app authenticates in local v1
- **THEN** it uses the `quantum-bank-mobile` public client with Authorization
  Code + PKCE (`S256`)
- **AND** it does not use the `quantum-bank-test` client

#### Scenario: Consumers validate issuer and audience
- **WHEN** the gateway or backend validates a token
- **THEN** it accepts the `quantum-bank-local` issuer and audience
  `quantum-bank-api`
- **AND** rejects tokens missing required claims (`iss`, `sub`, `aud`, `exp`,
  `iat`, `azp`, `scope`)

### Requirement: Local runtime provides required trust material and configuration
The local runtime SHALL provide the trust material (app-to-gateway and
gateway-to-backend mTLS trust anchors, the mobile client certificate chain, and
gateway/backend trust stores) and the runtime configuration (gateway base URL,
backend internal base URL, OAuth2 issuer/JWKS/audience, PKI integration path,
certificate profile `quantum-bank-mobile-client-v1`, H2 mode, and correlation-id
propagation) needed to exercise the secure path without hard-coded secrets or
direct mobile-to-backend access. Trust anchors are owned by PKI and consumed by
the runtime.

#### Scenario: Runtime configuration is present
- **WHEN** the local runtime is validated
- **THEN** the required trust material and configuration values are available
- **AND** no protected banking call can bypass the gateway to reach the backend

### Requirement: Gateway listener ports and paths are defined
The local runtime SHALL expose the bootstrap listener on port `8080`
(`POST /auth/otk`, `POST /auth/csr`) and the banking listener on port `8443`
(`POST /pix/transfers`, `GET /statements`, `GET /profile`), matching
`api-gateway/openapi/quantum-bank-v1.yaml`.

#### Scenario: Listener paths match the contract
- **WHEN** the gateway listeners are configured
- **THEN** the bootstrap paths are served on `8080` and the banking paths on
  `8443`
- **AND** the served paths match the gateway OpenAPI contract
