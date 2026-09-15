# Feature Specification: Deployment Infrastructure

**Feature Branch**: `004-deployment-infrastructure`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `deployment-infrastructure`

## Overview

Define local container and cloud infrastructure expectations for validating
Quantum Bank end to end across backend, gateway, security, and deployment
paths.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Backend and gateway are dockerized (Priority: P1)

The backend, the KrakenD API gateway, and the `backend-client` external service
SHALL be runnable through local container configuration.

**Acceptance Scenarios**:

1. **When** the local container environment is started, **Then** the backend service is available to KrakenD, **And** KrakenD is available as the mobile-facing API entrypoint, **And** the `backend-client` service runs as a container that reaches the backend only through KrakenD

---

### User Story 2 - Compose supports end-to-end validation (Priority: P1)

The local Docker Compose setup SHALL support validating backend, gateway, OAuth2, mTLS, and optional PKI components together.

**Acceptance Scenarios**:

1. **When** the local environment is launched for validation, **Then** protected requests can be exercised through KrakenD to the backend, **And** security components needed by the selected validation scope are available

---

### User Story 3 - Terraform paths exist for major clouds (Priority: P1)

Infrastructure SHALL include Terraform deployment paths for AWS, GCP, and Azure.

**Acceptance Scenarios**:

1. **When** Terraform validation is run for a supported cloud path, **Then** the AWS, GCP, or Azure configuration validates independently without hiding provider-specific differences behind a single generic module

---

### User Story 4 - Local runtime provides the required service topology (Priority: P1)

The local runtime SHALL provide `api-gateway`, `backend`, `backend-client`,
`pki`, an `oauth2-issuer`, and an H2-backed backend runtime, wired so mobile
traffic and `backend-client` external-service traffic both enter through the
gateway, the backend owns business and OTK logic, PKI owns certificate
lifecycle, and the OAuth2 issuer provides bearer tokens for gateway and backend
validation.

**Acceptance Scenarios**:

1. **When** the local runtime is started for end-to-end validation, **Then** the gateway, backend, `backend-client`, PKI, OAuth2 issuer, and H2-backed backend are available, **And** mobile traffic reaches the backend only through the gateway
2. **When** the `backend-client` service issues a banking call in the local runtime, **Then** the call reaches the backend only through the gateway, **And** no `backend-client`-to-backend direct path is available

---

### User Story 5 - Local OAuth2 issuer is Keycloak with a defined realm and clients (Priority: P1)

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

**Acceptance Scenarios**:

1. **When** the mobile app authenticates in local v1, **Then** it uses the `quantum-bank-mobile` public client with Authorization Code + PKCE (`S256`), **And** it does not use the `quantum-bank-test` client
2. **When** the `backend-client` service authenticates in local v1, **Then** it uses the confidential `quantum-bank-backend-client` client with the client-credentials grant, **And** the issued token carries audience `quantum-bank-api`
3. **When** the gateway or backend validates a token, **Then** it accepts the `quantum-bank-local` issuer and audience `quantum-bank-api`, **And** rejects tokens missing required claims (`iss`, `sub`, `aud`, `exp`, `iat`, `azp`, `scope`)

---

### User Story 6 - Local runtime provides required trust material and configuration (Priority: P1)

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

**Acceptance Scenarios**:

1. **When** the local runtime is validated, **Then** the required trust material and configuration values are available, **And** no protected banking call can bypass the gateway to reach the backend
2. **When** the `backend-client` service is validated in the local runtime, **Then** its PKI-issued service client certificate chain and trust anchors are available under the `quantum-bank-service-client-v1` profile, **And** the `backend-client` uses them for gateway mTLS rather than a permissive TLS mode

---

### User Story 7 - Gateway listener ports and paths are defined (Priority: P1)

The local runtime SHALL expose the bootstrap listener on port `8080`
(`POST /auth/otk`, `POST /auth/csr`) and the banking listener on port `8443`
(`POST /pix/transfers`, `GET /statements`, `GET /profile`), matching
`api-gateway/openapi/quantum-bank-v1.yaml`.

**Acceptance Scenarios**:

1. **When** the gateway listeners are configured, **Then** the bootstrap paths are served on `8080` and the banking paths on `8443`, **And** the served paths match the gateway OpenAPI contract

---

### User Story 8 - backend-client is built and tested in Docker (Priority: P1)

The `backend-client` SHALL be built, tested, and coverage-verified inside a
multi-stage Docker image so that no Java or Gradle toolchain is required on the
host, and its runtime image SHALL join `infrastructure/compose.yaml` with a
build context like the other layer services.

**Acceptance Scenarios**:

1. **When** the `backend-client` Docker image is built, **Then** the build stage compiles the service and runs its tests with the 100% coverage gate, **And** the build fails if coverage is below the enforced minimum
2. **When** `docker compose up` is run for the local runtime, **Then** the `backend-client` image is built from its build context and started as a service, **And** no host-installed Java or Gradle toolchain is needed to build, test, or run it

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend, the KrakenD API gateway, and the `backend-client` external service SHALL be runnable through local container configuration.
- **FR-002**: The local Docker Compose setup SHALL support validating backend, gateway, OAuth2, mTLS, and optional PKI components together.
- **FR-003**: Infrastructure SHALL include Terraform deployment paths for AWS, GCP, and Azure.
- **FR-004**: The local runtime SHALL provide `api-gateway`, `backend`, `backend-client`, `pki`, an `oauth2-issuer`, and an H2-backed backend runtime, wired so mobile traffic and `backend-client` external-service traffic both enter through the gateway, the backend owns business and OTK logic, PKI owns certificate lifecycle, and the OAuth2 issuer provides bearer tokens for gateway and backend validation.
- **FR-005**: The local-v1 OAuth2/OIDC issuer SHALL be Keycloak running from the infrastructure Compose file with the imported realm `quantum-bank-local`, exposing a host-facing issuer at `http://localhost:8180/realms/quantum-bank-local` and a container-network issuer at `http://keycloak:8080/realms/quantum-bank-local`, a public mobile client `quantum-bank-mobile` using Authorization Code + PKCE (`S256`, direct access grants disabled), a confidential service client `quantum-bank-backend-client` using the client-credentials grant for the external `backend-client` service, an isolated local test client `quantum-bank-test` that MUST NOT appear in mobile configuration, and audience `quantum-bank-api`. Keycloak MUST NOT use host port `8080`.
- **FR-006**: The local runtime SHALL provide the trust material (app-to-gateway, `backend-client`-to-gateway, and gateway-to-backend mTLS trust anchors, the mobile client certificate chain, the `backend-client` service client certificate chain, and gateway/backend trust stores) and the runtime configuration (gateway base URL, backend internal base URL, OAuth2 issuer/JWKS/audience, PKI integration path, certificate profiles `quantum-bank-mobile-client-v1` and `quantum-bank-service-client-v1`, H2 mode, and correlation-id propagation) needed to exercise the secure path without hard-coded secrets or direct client-to-backend access. Trust anchors are owned by PKI and consumed by the runtime.
- **FR-007**: The local runtime SHALL expose the bootstrap listener on port `8080` (`POST /auth/otk`, `POST /auth/csr`) and the banking listener on port `8443` (`POST /pix/transfers`, `GET /statements`, `GET /profile`), matching `api-gateway/openapi/quantum-bank-v1.yaml`.
- **FR-008**: The `backend-client` SHALL be built, tested, and coverage-verified inside a multi-stage Docker image so that no Java or Gradle toolchain is required on the host, and its runtime image SHALL join `infrastructure/compose.yaml` with a build context like the other layer services.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
