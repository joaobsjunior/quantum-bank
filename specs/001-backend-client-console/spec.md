# Feature Specification: Backend Client Console

**Feature Branch**: `001-backend-client-console`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `backend-client-console`

## Overview

TBD - created by archiving change add-backend-client-service. Update Purpose after archive.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Console drives every external-integration flow (Priority: P1)

The `backend-client` SHALL provide a human-facing web console that lets a person
manually trigger every external-integration flow — Pix transfer, account
statement, and profile / registration data — without writing code or crafting
requests by hand.

**Acceptance Scenarios**:

1. **When** a person selects a banking flow in the console and submits it, **Then** the console triggers the corresponding external-integration call through the gateway, **And** the outcome of that call is shown in the console
2. **When** a person opens the console, **Then** the Pix transfer, account statement, and profile flows are each available to run, **And** each flow maps to the same gateway-fronted capability the mobile app uses

---

### User Story 2 - Console lets the operator select the Pix scenario (Priority: P1)

The console SHALL let the operator choose the Pix success or error simulation
scenario before submitting a Pix transfer.

**Acceptance Scenarios**:

1. **When** the operator chooses a success or error scenario and submits a Pix transfer, **Then** the `backend-client` submits that selected scenario through the gateway, **And** the console displays the corresponding simulated success or error result

---

### User Story 3 - Console surfaces request, response, and correlation detail (Priority: P1)

For each executed flow the console SHALL display the outbound request summary,
the gateway response, the `correlationId`, and any RFC 9457 problem-details error
body, so the operator can observe the secure path end to end.

**Acceptance Scenarios**:

1. **When** a flow completes successfully, **Then** the console shows the request summary, the response, and the correlation id
2. **When** a flow returns a problem-details error, **Then** the console shows the error type, status, detail, and correlation id, **And** it does not hide the failure behind a generic message

---

### User Story 4 - Console never exposes a direct-to-backend path (Priority: P1)

The console SHALL route every flow through the gateway-fronted
external-integration client and MUST NOT offer any option that calls a backend
service directly or bypasses OAuth2 and mTLS.

**Acceptance Scenarios**:

1. **When** a person uses any control in the console, **Then** the resulting call goes through the gateway with the service token and mTLS identity, **And** no console action can reach the backend directly

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: The `backend-client` SHALL provide a human-facing web console that lets a person manually trigger every external-integration flow — Pix transfer, account statement, and profile / registration data — without writing code or crafting requests by hand.
- **FR-002**: The console SHALL let the operator choose the Pix success or error simulation scenario before submitting a Pix transfer.
- **FR-003**: For each executed flow the console SHALL display the outbound request summary, the gateway response, the `correlationId`, and any RFC 9457 problem-details error body, so the operator can observe the secure path end to end.
- **FR-004**: The console SHALL route every flow through the gateway-fronted external-integration client and MUST NOT offer any option that calls a backend service directly or bypasses OAuth2 and mTLS.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
