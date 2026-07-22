## MODIFIED Requirements

### Requirement: Backend and gateway are dockerized
The backend, the KrakenD API gateway, and the `backend-client` external service
SHALL be runnable through local container configuration.

#### Scenario: Local environment starts
- **WHEN** the local container environment is started
- **THEN** the backend service is available to KrakenD
- **AND** KrakenD is available as the mobile-facing API entrypoint
- **AND** the `backend-client` service runs as a container that reaches the
  backend only through KrakenD

### Requirement: Local runtime provides the required service topology
The local runtime SHALL provide `api-gateway`, `backend`, `backend-client`,
`pki`, an `oauth2-issuer`, and an H2-backed backend runtime, wired so mobile
traffic and `backend-client` external-service traffic both enter through the
gateway, the backend owns business and OTK logic, PKI owns certificate
lifecycle, and the OAuth2 issuer provides bearer tokens for gateway and backend
validation.

#### Scenario: Local topology is brought up
- **WHEN** the local runtime is started for end-to-end validation
- **THEN** the gateway, backend, `backend-client`, PKI, OAuth2 issuer, and
  H2-backed backend are available
- **AND** mobile traffic reaches the backend only through the gateway

#### Scenario: External service traffic enters through the gateway
- **WHEN** the `backend-client` service issues a banking call in the local
  runtime
- **THEN** the call reaches the backend only through the gateway
- **AND** no `backend-client`-to-backend direct path is available

### Requirement: Local OAuth2 issuer is Keycloak with a defined realm and clients
The local-v1 OAuth2/OIDC issuer SHALL be Keycloak running from the infrastructure
Compose file with the imported realm `quantum-bank-local`, exposing a
host-facing issuer at `http://localhost:8180/realms/quantum-bank-local` and a
container-network issuer at `http://keycloak:8080/realms/quantum-bank-local`, a
public mobile client `quantum-bank-mobile` using Authorization Code + PKCE
(`S256`, direct access grants disabled), a confidential service client
`quantum-bank-backend-client` using the client-credentials grant for the
external `backend-client` service, an isolated local test client
`quantum-bank-test` that MUST NOT appear in mobile configuration, and audience
`quantum-bank-api`. Keycloak MUST NOT use host port `8080`.

#### Scenario: Mobile authenticates against the local issuer
- **WHEN** the mobile app authenticates in local v1
- **THEN** it uses the `quantum-bank-mobile` public client with Authorization
  Code + PKCE (`S256`)
- **AND** it does not use the `quantum-bank-test` client

#### Scenario: External service authenticates with client credentials
- **WHEN** the `backend-client` service authenticates in local v1
- **THEN** it uses the confidential `quantum-bank-backend-client` client with the
  client-credentials grant
- **AND** the issued token carries audience `quantum-bank-api`

#### Scenario: Consumers validate issuer and audience
- **WHEN** the gateway or backend validates a token
- **THEN** it accepts the `quantum-bank-local` issuer and audience
  `quantum-bank-api`
- **AND** rejects tokens missing required claims (`iss`, `sub`, `aud`, `exp`,
  `iat`, `azp`, `scope`)

### Requirement: Local runtime provides required trust material and configuration
The local runtime SHALL provide the trust material (app-to-gateway,
`backend-client`-to-gateway, and gateway-to-backend mTLS trust anchors, the
mobile client certificate chain, the `backend-client` service client certificate
chain, and gateway/backend trust stores) and the runtime configuration (gateway
base URL, backend internal base URL, OAuth2 issuer/JWKS/audience, PKI integration
path, certificate profiles `quantum-bank-mobile-client-v1` and
`quantum-bank-service-client-v1`, H2 mode, and correlation-id propagation) needed
to exercise the secure path without hard-coded secrets or direct
client-to-backend access. Trust anchors are owned by PKI and consumed by the
runtime.

#### Scenario: Runtime configuration is present
- **WHEN** the local runtime is validated
- **THEN** the required trust material and configuration values are available
- **AND** no protected banking call can bypass the gateway to reach the backend

#### Scenario: Service certificate material is present
- **WHEN** the `backend-client` service is validated in the local runtime
- **THEN** its PKI-issued service client certificate chain and trust anchors are
  available under the `quantum-bank-service-client-v1` profile
- **AND** the `backend-client` uses them for gateway mTLS rather than a
  permissive TLS mode

## ADDED Requirements

### Requirement: backend-client is built and tested in Docker
The `backend-client` SHALL be built, tested, and coverage-verified inside a
multi-stage Docker image so that no Java or Gradle toolchain is required on the
host, and its runtime image SHALL join `infrastructure/compose.yaml` with a
build context like the other layer services.

#### Scenario: Image build runs the coverage gate
- **WHEN** the `backend-client` Docker image is built
- **THEN** the build stage compiles the service and runs its tests with the 100%
  coverage gate
- **AND** the build fails if coverage is below the enforced minimum

#### Scenario: Stack runs through Docker Compose
- **WHEN** `docker compose up` is run for the local runtime
- **THEN** the `backend-client` image is built from its build context and started
  as a service
- **AND** no host-installed Java or Gradle toolchain is needed to build, test, or
  run it
