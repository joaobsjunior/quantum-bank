# Feature Specification: Secure Gateway Communication

**Feature Branch**: `010-secure-gateway-communication`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `secure-gateway-communication`

## Overview

Define the secure communication contract between the Flutter mobile app,
KrakenD, and the Spring Boot backend. OAuth2 gates protected access and mobile
traffic reaches the backend only through KrakenD.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Gateway-only mobile communication (Priority: P1)

The mobile app SHALL communicate with backend capabilities only through the KrakenD API gateway.

**Acceptance Scenarios**:

1. **When** the mobile app calls a protected Quantum Bank backend capability, **Then** the request goes to KrakenD rather than directly to a backend service, **And** backend service addresses are not exposed as the mobile runtime API base URL
2. **When** a caller attempts to access a protected backend endpoint directly, **Then** the request is rejected unless it satisfies the backend's own security requirements

---

### User Story 2 - OAuth2 protects gateway and backend (Priority: P1)

Protected APIs SHALL require OAuth2 bearer token validation at KrakenD and at the Spring Boot backend.

**Acceptance Scenarios**:

1. **When** a request includes a valid access token with the expected issuer, audience, and claims, **Then** KrakenD accepts the edge request, **And** the backend resource server accepts the forwarded request
2. **When** a request omits the access token or presents an invalid token, **Then** KrakenD rejects the request before forwarding it, **And** the backend also rejects equivalent direct protected requests

---

### User Story 3 - Mutual TLS is enforced for secure channels (Priority: P1)

The system SHALL use mTLS for app-to-gateway and gateway-to-backend communication where certificate material has been provisioned.

**Acceptance Scenarios**:

1. **When** a client presents a certificate issued by the project trust chain, **Then** the TLS handshake succeeds, **And** the request can continue to OAuth2 and business validation
2. **When** a client presents no certificate or a certificate outside the project trust chain, **Then** the TLS handshake or gateway policy rejects the connection

---

### User Story 4 - TLS verification fails closed (Priority: P1)

The mobile app SHALL load trusted certificate authority material and SHALL NOT disable TLS verification with permissive certificate callbacks.

**Acceptance Scenarios**:

1. **When** the gateway presents a certificate chaining to the configured project CA, **Then** the mobile TLS client accepts the connection
2. **When** the gateway presents an unknown or invalid certificate, **Then** the mobile TLS client rejects the connection

---

### User Story 5 - Gateway exposes only the published app-facing route surface (Priority: P1)

KrakenD SHALL expose only the published app-facing routes (`POST /auth/otk`,
`POST /auth/csr`, `POST /pix/transfers`, `GET /statements`, `GET /profile`, and
`PUT /profile`) sourced from `api-gateway/openapi/quantum-bank-v1.yaml`, and
SHALL NOT expose backend hostnames, ports, or internal paths as the mobile
runtime origin.

**Acceptance Scenarios**:

1. **When** the mobile app resolves a protected banking route, **Then** the route is one of the published gateway paths, **And** no backend service host, port, or internal path is used as the origin
2. **When** a caller requests a path that is not part of the published gateway surface, **Then** the gateway does not route it to a backend service

---

### User Story 6 - Gateway enforces per-endpoint OAuth2 scopes (Priority: P1)

KrakenD SHALL require the OAuth2 scopes defined for each protected route before
routing to the backend: `openid`+`profile` for `/auth/otk` and `/auth/csr`,
`pix:write` for `POST /pix/transfers`, `statements:read` for `GET /statements`,
and `profile:read` for `GET /profile`.

**Acceptance Scenarios**:

1. **When** a request to `POST /pix/transfers` carries a valid token with the `pix:write` scope, **Then** the gateway routes the request to the backend
2. **When** a request to `POST /pix/transfers` carries a valid token without the `pix:write` scope, **Then** the gateway rejects it with `403` and `application/problem+json`, **And** the request is not routed to the backend

---

### User Story 7 - Gateway rejects invalid tokens fail-closed (Priority: P1)

KrakenD SHALL reject a request before backend routing when the bearer token is
missing, malformed, expired, or presents the wrong issuer or audience, returning
`401 application/problem+json`; a valid token that lacks the endpoint scope SHALL
return `403 application/problem+json`.

**Acceptance Scenarios**:

1. **When** a request omits the token or presents a malformed, expired, wrong-issuer, or wrong-audience token, **Then** the gateway returns `401` with `application/problem+json`, **And** the request is not forwarded to the backend
2. **When** a request carries a token with the expected issuer (`quantum-bank-local` realm), audience `quantum-bank-api`, `RS256` signature, and required scope, **Then** the gateway accepts and routes the request

---

### User Story 8 - Gateway uses a two-listener mTLS topology (Priority: P1)

The gateway SHALL run a bootstrap listener that is OAuth2-only (no app client
certificate) for `POST /auth/otk` and `POST /auth/csr`, and a banking listener
that requires both OAuth2 and app-to-gateway mTLS for `POST /pix/transfers`,
`GET /statements`, and `GET /profile`.

**Acceptance Scenarios**:

1. **When** the app calls `POST /auth/otk` or `POST /auth/csr` before it has a client certificate, **Then** the bootstrap listener accepts the OAuth2-authenticated request without requiring an app client certificate
2. **When** the app calls a protected banking route on the banking listener, **Then** the listener requires a trusted mobile client certificate in addition to OAuth2, **And** a missing, untrusted, expired, or wrong-environment client certificate fails at the TLS handshake rather than returning a routed HTTP response

---

### User Story 9 - Gateway-to-backend mTLS is a distinct enforced boundary (Priority: P1)

Gateway-to-backend traffic SHALL be protected by mTLS on every configured hop,
including the bootstrap routes, as a boundary separate from app-to-gateway mTLS.

**Acceptance Scenarios**:

1. **When** the gateway forwards any accepted request to the backend, **Then** it uses its gateway client certificate over an mTLS connection
2. **When** a caller reaches the backend mTLS port without a trusted gateway client certificate, **Then** the connection fails at the TLS handshake

---

### User Story 10 - Backend independently validates OAuth2 and does not trust gateway headers (Priority: P1)

The backend SHALL validate the OAuth2 bearer token on every protected route as
defense in depth, derive user identity only from the validated JWT, and SHALL
NOT treat gateway-forwarded identity, scope, or certificate headers as
authentication.

**Acceptance Scenarios**:

1. **When** a protected backend route receives a request without a valid bearer token, **Then** the backend rejects it regardless of any forwarded gateway headers
2. **When** a request carries gateway-forwarded subject, scope, or certificate metadata, **Then** the backend uses that metadata only after independently validating the JWT, and derives user identity from the JWT

---

### User Story 11 - Authorization failures use RFC 9457 problem details (Priority: P1)

Gateway and backend authentication and authorization failures SHALL use RFC 9457
problem details with media type `application/problem+json`, including stable
`status`, `title`, `errorCode`, and `correlationId` fields, and SHALL NOT expose
token values, backend hostnames, stack traces, route internals, or PKI internals.

**Acceptance Scenarios**:

1. **When** the gateway or backend rejects a request for auth reasons, **Then** the response media type is `application/problem+json`, **And** it includes `status`, `title`, `errorCode`, and `correlationId`, **And** it does not include token values, backend hostnames, stack traces, or PKI internals

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The mobile app SHALL communicate with backend capabilities only through the KrakenD API gateway.
- **FR-002**: Protected APIs SHALL require OAuth2 bearer token validation at KrakenD and at the Spring Boot backend.
- **FR-003**: The system SHALL use mTLS for app-to-gateway and gateway-to-backend communication where certificate material has been provisioned.
- **FR-004**: The mobile app SHALL load trusted certificate authority material and SHALL NOT disable TLS verification with permissive certificate callbacks.
- **FR-005**: KrakenD SHALL expose only the published app-facing routes (`POST /auth/otk`, `POST /auth/csr`, `POST /pix/transfers`, `GET /statements`, `GET /profile`, and `PUT /profile`) sourced from `api-gateway/openapi/quantum-bank-v1.yaml`, and SHALL NOT expose backend hostnames, ports, or internal paths as the mobile runtime origin.
- **FR-006**: KrakenD SHALL require the OAuth2 scopes defined for each protected route before routing to the backend: `openid`+`profile` for `/auth/otk` and `/auth/csr`, `pix:write` for `POST /pix/transfers`, `statements:read` for `GET /statements`, and `profile:read` for `GET /profile`.
- **FR-007**: KrakenD SHALL reject a request before backend routing when the bearer token is missing, malformed, expired, or presents the wrong issuer or audience, returning `401 application/problem+json`; a valid token that lacks the endpoint scope SHALL return `403 application/problem+json`.
- **FR-008**: The gateway SHALL run a bootstrap listener that is OAuth2-only (no app client certificate) for `POST /auth/otk` and `POST /auth/csr`, and a banking listener that requires both OAuth2 and app-to-gateway mTLS for `POST /pix/transfers`, `GET /statements`, and `GET /profile`.
- **FR-009**: Gateway-to-backend traffic SHALL be protected by mTLS on every configured hop, including the bootstrap routes, as a boundary separate from app-to-gateway mTLS.
- **FR-010**: The backend SHALL validate the OAuth2 bearer token on every protected route as defense in depth, derive user identity only from the validated JWT, and SHALL NOT treat gateway-forwarded identity, scope, or certificate headers as authentication.
- **FR-011**: Gateway and backend authentication and authorization failures SHALL use RFC 9457 problem details with media type `application/problem+json`, including stable `status`, `title`, `errorCode`, and `correlationId` fields, and SHALL NOT expose token values, backend hostnames, stack traces, route internals, or PKI internals.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
