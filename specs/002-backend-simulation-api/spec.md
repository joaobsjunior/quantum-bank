# Feature Specification: Backend Simulation API

**Feature Branch**: `002-backend-simulation-api`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `backend-simulation-api`

## Overview

Define the Spring Boot Kotlin backend requirements for local simulation APIs,
OAuth2-protected banking data, and in-memory development persistence.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Backend uses Spring Boot Kotlin (Priority: P1)

The backend SHALL be implemented with Spring Boot Kotlin 4.0.6.

**Acceptance Scenarios**:

1. **When** the backend build configuration is inspected, **Then** it uses the Spring Boot Kotlin 4.0.6 line or the documented project equivalent

---

### User Story 2 - Backend supports v1 in-memory persistence (Priority: P1)

The backend SHALL support H2 or another in-memory database for v1 development and simulation data.

**Acceptance Scenarios**:

1. **When** the backend starts in local development mode, **Then** simulation data can be stored without requiring an external database

---

### User Story 3 - Backend exposes banking simulation APIs (Priority: P1)

The backend SHALL expose APIs for Pix simulation, account statement data, and customer registration data.

**Acceptance Scenarios**:

1. **When** an authenticated request asks for account statement data through KrakenD, **Then** the backend returns statement records from local simulation data
2. **When** an authenticated request asks for customer registration data through KrakenD, **Then** the backend returns the configured customer data fixture

---

### User Story 4 - Route-to-backend responsibility mapping (Priority: P1)

The backend SHALL implement the app-facing gateway routes with these
responsibilities: `POST /auth/otk` issues an OTK bound to the authenticated
subject, app, device, and profile; `POST /auth/csr` validates and consumes the
OTK and hands off the CSR; `POST /pix/transfers` returns a scenario-driven Pix
result; `GET /statements` returns statement entries for the subject;
`GET /profile` returns customer registration data; and `PUT /profile` edits it.
Backend behavior SHALL remain compatible with
`api-gateway/openapi/quantum-bank-v1.yaml`.

**Acceptance Scenarios**:

1. **When** an authenticated `GET /statements` request arrives through KrakenD, **Then** the backend returns statement entries for the authenticated subject matching the `StatementResponse` schema
2. **When** a backend route response is produced, **Then** its shape matches the corresponding schema in `api-gateway/openapi/quantum-bank-v1.yaml`

---

### User Story 5 - Pix simulation is scenario-driven and persisted (Priority: P1)

For `POST /pix/transfers` the backend SHALL interpret the caller-provided
`scenario`: `SUCCESS` persists and returns status `COMPLETED` with the success
schema, and `ERROR` persists status `FAILED` and returns
`application/problem+json` with stable `errorCode` `pix_simulated_error`. The
backend SHALL NOT rely on random failure to produce the error path.

**Acceptance Scenarios**:

1. **When** a Pix transfer is submitted with `scenario` `SUCCESS`, **Then** the backend persists a `COMPLETED` outcome and returns the success response
2. **When** a Pix transfer is submitted with `scenario` `ERROR`, **Then** the backend persists a `FAILED` outcome and returns `application/problem+json` with `errorCode` `pix_simulated_error`

---

### User Story 6 - Statement and profile responses are deterministic (Priority: P1)

`GET /statements` and `GET /profile` SHALL return deterministic local-v1 data for
the authenticated subject, each including a `correlationId` and matching the
gateway OpenAPI `StatementResponse` and `ProfileResponse` schemas.

**Acceptance Scenarios**:

1. **When** the authenticated subject requests `GET /profile`, **Then** the backend returns the configured customer data fixture with a `correlationId` matching `ProfileResponse`

---

### User Story 7 - Profile write requires the profile:write scope (Priority: P1)

`PUT /profile` SHALL update editable customer registration fields for the
authenticated subject and SHALL require the `profile:write` scope; `profile:read`
is not sufficient.

**Acceptance Scenarios**:

1. **When** a `PUT /profile` request carries only `profile:read`, **Then** the request is rejected and no profile change is persisted
2. **When** a `PUT /profile` request carries `profile:write`, **Then** the backend updates the editable profile fields

---

### User Story 8 - Backend app-facing errors use problem details (Priority: P1)

All app-facing backend failures routed through KrakenD SHALL use
`application/problem+json` including `type`, `title`, `status`, `errorCode`, and
`correlationId`, with optional `detail`, `instance`, and `fieldErrors` when safe
for display.

**Acceptance Scenarios**:

1. **When** a backend route fails for an app-facing request, **Then** the response is `application/problem+json` with `type`, `title`, `status`, `errorCode`, and `correlationId`

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The backend SHALL be implemented with Spring Boot Kotlin 4.0.6.
- **FR-002**: The backend SHALL support H2 or another in-memory database for v1 development and simulation data.
- **FR-003**: The backend SHALL expose APIs for Pix simulation, account statement data, and customer registration data.
- **FR-004**: The backend SHALL implement the app-facing gateway routes with these responsibilities: `POST /auth/otk` issues an OTK bound to the authenticated subject, app, device, and profile; `POST /auth/csr` validates and consumes the OTK and hands off the CSR; `POST /pix/transfers` returns a scenario-driven Pix result; `GET /statements` returns statement entries for the subject; `GET /profile` returns customer registration data; and `PUT /profile` edits it. Backend behavior SHALL remain compatible with `api-gateway/openapi/quantum-bank-v1.yaml`.
- **FR-005**: For `POST /pix/transfers` the backend SHALL interpret the caller-provided `scenario`: `SUCCESS` persists and returns status `COMPLETED` with the success schema, and `ERROR` persists status `FAILED` and returns `application/problem+json` with stable `errorCode` `pix_simulated_error`. The backend SHALL NOT rely on random failure to produce the error path.
- **FR-006**: `GET /statements` and `GET /profile` SHALL return deterministic local-v1 data for the authenticated subject, each including a `correlationId` and matching the gateway OpenAPI `StatementResponse` and `ProfileResponse` schemas.
- **FR-007**: `PUT /profile` SHALL update editable customer registration fields for the authenticated subject and SHALL require the `profile:write` scope; `profile:read` is not sufficient.
- **FR-008**: All app-facing backend failures routed through KrakenD SHALL use `application/problem+json` including `type`, `title`, `status`, `errorCode`, and `correlationId`, with optional `detail`, `instance`, and `fieldErrors` when safe for display.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
