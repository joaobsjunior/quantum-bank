# Feature Specification: External Service Integration

**Feature Branch**: `005-external-service-integration`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `external-service-integration`

## Overview

TBD - created by archiving change add-backend-client-service. Update Purpose after archive.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Gateway-only external service communication (Priority: P1)

The `backend-client` external service SHALL reach Quantum Bank banking
capabilities only through the KrakenD API gateway, never by calling a backend
service address directly.

**Acceptance Scenarios**:

1. **When** the `backend-client` invokes a protected Quantum Bank capability, **Then** the request is sent to the KrakenD gateway origin, **And** no backend service address is configured or used as the outbound base URL
2. **When** the `backend-client` is configured with a base URL that resolves to a backend service instead of the gateway, **Then** the configuration is treated as invalid and the service fails to start or refuses to issue the call

---

### User Story 2 - Service authenticates with its own OAuth2 client credentials (Priority: P1)

The `backend-client` SHALL authenticate as its own registered OAuth2 client
using the client-credentials grant and present the resulting bearer token to the
gateway; it MUST NOT reuse the mobile client identity or any end-user
credential.

**Acceptance Scenarios**:

1. **When** the `backend-client` prepares an outbound banking call, **Then** it obtains an access token from the OAuth2 issuer via the client-credentials grant for its own client, **And** it sends that token as a bearer credential to the gateway
2. **When** the `backend-client` has no valid access token, **Then** it does not send the protected request, **And** it surfaces an authentication error rather than calling without a token

---

### User Story 3 - Service presents a PKI-issued mTLS identity on the banking listener (Priority: P1)

The `backend-client` SHALL present a PKI-issued service client certificate for
mutual TLS on the gateway banking listener, and SHALL fail closed when the
certificate or trust material is missing or invalid.

**Acceptance Scenarios**:

1. **When** the `backend-client` calls a banking-listener capability, **Then** it completes the mTLS handshake using its PKI-issued service client certificate, **And** it validates the gateway certificate against the configured trust anchors
2. **When** the service client certificate or trust anchors are absent or invalid, **Then** the `backend-client` does not complete the banking call, **And** it does not fall back to a permissive or certificate-ignoring TLS mode

---

### User Story 4 - External service exercises the banking flows (Priority: P1)

The `backend-client` SHALL be able to exercise the Pix transfer (with
app-selected success and error simulation), account statement, and profile /
registration-data capabilities through the gateway, as an external caller.

**Acceptance Scenarios**:

1. **When** the `backend-client` submits a Pix transfer selecting a success or error scenario, **Then** the request is routed through the gateway to the backend Pix simulation, **And** the simulated success or error outcome is returned to the `backend-client`
2. **When** the `backend-client` requests the account statement or profile data, **Then** the gateway returns the deterministic simulated response from the backend, **And** the response is associated with the request's correlation id

---

### User Story 5 - External service propagates correlation and handles problem details (Priority: P1)

The `backend-client` SHALL propagate a `correlationId` on outbound requests and
SHALL parse RFC 9457 problem-details error responses returned through the
gateway.

**Acceptance Scenarios**:

1. **When** the `backend-client` issues a banking request, **Then** the request carries a `correlationId`, **And** the same correlation id is used to correlate the response
2. **When** the gateway or backend returns an RFC 9457 problem-details error, **Then** the `backend-client` parses the problem-details body, **And** it exposes the error type, status, and detail rather than a raw or swallowed failure

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The `backend-client` external service SHALL reach Quantum Bank banking capabilities only through the KrakenD API gateway, never by calling a backend service address directly.
- **FR-002**: The `backend-client` SHALL authenticate as its own registered OAuth2 client using the client-credentials grant and present the resulting bearer token to the gateway; it MUST NOT reuse the mobile client identity or any end-user credential.
- **FR-003**: The `backend-client` SHALL present a PKI-issued service client certificate for mutual TLS on the gateway banking listener, and SHALL fail closed when the certificate or trust material is missing or invalid.
- **FR-004**: The `backend-client` SHALL be able to exercise the Pix transfer (with app-selected success and error simulation), account statement, and profile / registration-data capabilities through the gateway, as an external caller.
- **FR-005**: The `backend-client` SHALL propagate a `correlationId` on outbound requests and SHALL parse RFC 9457 problem-details error responses returned through the gateway.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
