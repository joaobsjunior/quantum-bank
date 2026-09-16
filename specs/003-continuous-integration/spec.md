# Feature Specification: Continuous Integration

**Feature Branch**: `003-continuous-integration`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `continuous-integration`

## Overview

Define the GitHub Actions CI pipeline for the whole solution: a build-test-cover
job per layer submodule, a superproject orchestrator that checks out submodules
and fans out to each layer, fail-closed gating of merges, and an optional
end-to-end job.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Every layer has a CI build-and-test job (Priority: P1)

Each layer submodule SHALL have a GitHub Actions job that builds the layer, runs
its tests or validation, and enforces its coverage or validation gate, covering
all five layers (mobile-app, backend, api-gateway, infrastructure, pki).

**Acceptance Scenarios**:

1. **When** a layer's CI job runs on a commit that builds and passes all tests and gates, **Then** the job reports success
2. **When** a layer's tests fail or its coverage/validation gate is not met, **Then** the layer's CI job fails, **And** the failure is surfaced as a status check

---

### User Story 2 - Superproject orchestrates all layers (Priority: P1)

The superproject SHALL provide a GitHub Actions workflow that checks out
submodules and runs every layer's job, so a single pipeline covers the whole
solution on pull requests and pushes to `main`.

**Acceptance Scenarios**:

1. **When** a pull request targets `main`, **Then** the superproject workflow runs each layer's build-test-coverage job
2. **When** the superproject workflow starts, **Then** it checks out the required submodule contents before running layer jobs

---

### User Story 3 - Coverage and validation failures block merges (Priority: P1)

The pipeline SHALL fail-closed: any layer job, coverage-threshold failure, or
validation failure blocks the overall pipeline result and the associated required
status check.

**Acceptance Scenarios**:

1. **When** any layer reports coverage below its configured threshold in CI, **Then** the pipeline result is failure, **And** the required status check does not pass
2. **When** every layer job, coverage gate, and validation gate passes, **Then** the pipeline result is success, **And** the required status check passes

---

### User Story 4 - Optional end-to-end validation job (Priority: P1)

The pipeline SHALL support an end-to-end job that brings up the solution via
Docker Compose to exercise the secure gateway path across layers.

**Acceptance Scenarios**:

1. **When** the end-to-end job is enabled and runs, **Then** it starts the Compose environment and exercises a protected request through KrakenD to the backend, **And** a failure in the end-to-end flow fails the job

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Each layer submodule SHALL have a GitHub Actions job that builds the layer, runs its tests or validation, and enforces its coverage or validation gate, covering all five layers (mobile-app, backend, api-gateway, infrastructure, pki).
- **FR-002**: The superproject SHALL provide a GitHub Actions workflow that checks out submodules and runs every layer's job, so a single pipeline covers the whole solution on pull requests and pushes to `main`.
- **FR-003**: The pipeline SHALL fail-closed: any layer job, coverage-threshold failure, or validation failure blocks the overall pipeline result and the associated required status check.
- **FR-004**: The pipeline SHALL support an end-to-end job that brings up the solution via Docker Compose to exercise the secure gateway path across layers.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
