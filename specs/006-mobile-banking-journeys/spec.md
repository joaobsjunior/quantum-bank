# Feature Specification: Mobile Banking Journeys

**Feature Branch**: `006-mobile-banking-journeys`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `mobile-banking-journeys`

## Overview

Define the initial Flutter mobile banking journeys and their secure API
configuration expectations.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Mobile app exposes initial journeys (Priority: P1)

The Flutter mobile app SHALL expose screens for Pix transfer, account statement, and customer registration data.

**Acceptance Scenarios**:

1. **When** an authenticated user reaches the main banking experience, **Then** the user can navigate to Pix transfer, **And** the user can navigate to account statement, **And** the user can navigate to customer registration data

---

### User Story 2 - Mobile app uses Flutter 3.41 (Priority: P1)

The mobile application SHALL be built with Flutter 3.41.

**Acceptance Scenarios**:

1. **When** the mobile project toolchain is inspected, **Then** the Flutter version requirement is documented or pinned to the 3.41 line

---

### User Story 3 - Mobile app uses secure API configuration (Priority: P1)

The mobile app SHALL use gateway API configuration and trusted TLS material for banking requests.

**Acceptance Scenarios**:

1. **When** the app sends a banking request, **Then** the request targets the configured KrakenD endpoint, **And** the TLS client uses trusted CA and client certificate material when mTLS is required

---

### User Story 4 - Mobile uses gateway-named origins only (Priority: P1)

The mobile app SHALL configure only gateway-named origins for protected APIs —
`GATEWAY_BOOTSTRAP_BASE_URL` for OAuth2 bootstrap calls and `GATEWAY_BASE_URL`
for certificate-ready banking calls — and SHALL NOT configure backend service
hosts as protected origins. Forbidden origin strings include `BACKEND_BASE_URL`,
`backend:`, `localhost:8081`, and `http://backend`; the source and config SHALL
pass `scripts/verify-gateway-only.sh`.

**Acceptance Scenarios**:

1. **When** the app resolves the base URL for a protected banking call, **Then** it uses `GATEWAY_BASE_URL` pointing at the KrakenD banking listener, **And** no backend host, container alias, or backend port is used
2. **When** `scripts/verify-gateway-only.sh` runs against the mobile source and config, **Then** it fails if any forbidden backend origin string is present

---

### User Story 5 - Mobile client preconditions for protected calls (Priority: P1)

The mobile API client SHALL call protected banking APIs only when
`authenticated`, `certificateReady`, and `gatewayBaseUrlConfigured` are all true;
bootstrap calls MAY occur before `certificateReady` when the route contract
allows OAuth2 bearer authentication without mobile client mTLS.

**Acceptance Scenarios**:

1. **When** the app attempts a protected banking call while `certificateReady` is false, **Then** the client does not issue the protected call
2. **When** the app performs an OAuth2 bootstrap call while `certificateReady` is false, **Then** the call is allowed because the bootstrap route does not require mobile client mTLS

---

### User Story 6 - Mobile parses problem details into a stable error model (Priority: P1)

The mobile client SHALL parse `application/problem+json` responses into a stable
model with `type`, `title`, `status`, `errorCode`, `correlationId`, and optional
`detail`, `instance`, and `fieldErrors`, and SHALL NOT display token values,
private key material, CSR internals, backend hostnames, or PKI internals.

**Acceptance Scenarios**:

1. **When** the app receives an `application/problem+json` error, **Then** it maps the response into the stable error model, **And** it displays only safe fields, never tokens, keys, CSR, or backend hosts

---

### User Story 7 - Pix scenario is app-selected, not inferred (Priority: P1)

The Pix transfer flow SHALL send an explicit `scenario` value (`SUCCESS` or
`ERROR`) chosen by the app, and SHALL NOT infer Pix errors from random network
behavior during local-v1 testing.

**Acceptance Scenarios**:

1. **When** the app submits a Pix transfer, **Then** it sends an explicit `SUCCESS` or `ERROR` scenario, **And** it does not treat incidental network failures as the Pix error path

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The Flutter mobile app SHALL expose screens for Pix transfer, account statement, and customer registration data.
- **FR-002**: The mobile application SHALL be built with Flutter 3.41.
- **FR-003**: The mobile app SHALL use gateway API configuration and trusted TLS material for banking requests.
- **FR-004**: The mobile app SHALL configure only gateway-named origins for protected APIs — `GATEWAY_BOOTSTRAP_BASE_URL` for OAuth2 bootstrap calls and `GATEWAY_BASE_URL` for certificate-ready banking calls — and SHALL NOT configure backend service hosts as protected origins. Forbidden origin strings include `BACKEND_BASE_URL`, `backend:`, `localhost:8081`, and `http://backend`; the source and config SHALL pass `scripts/verify-gateway-only.sh`.
- **FR-005**: The mobile API client SHALL call protected banking APIs only when `authenticated`, `certificateReady`, and `gatewayBaseUrlConfigured` are all true; bootstrap calls MAY occur before `certificateReady` when the route contract allows OAuth2 bearer authentication without mobile client mTLS.
- **FR-006**: The mobile client SHALL parse `application/problem+json` responses into a stable model with `type`, `title`, `status`, `errorCode`, `correlationId`, and optional `detail`, `instance`, and `fieldErrors`, and SHALL NOT display token values, private key material, CSR internals, backend hostnames, or PKI internals.
- **FR-007**: The Pix transfer flow SHALL send an explicit `scenario` value (`SUCCESS` or `ERROR`) chosen by the app, and SHALL NOT infer Pix errors from random network behavior during local-v1 testing.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
