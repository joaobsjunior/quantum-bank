# Feature Specification: Test Coverage Enforcement

**Feature Branch**: `011-test-coverage-enforcement`

**Created**: 2026-09-04

**Status**: Implemented

**Input**: Especificação consolidada da capacidade `test-coverage-enforcement`

## Overview

Define the enforced test-coverage quality gates: 100% line coverage on the code
layers (backend, mobile-app) and equivalent required validation gates on the
config/script layers (api-gateway, infrastructure, pki), all fail-closed.

## User Scenarios & Testing *(mandatory)*

<!-- Cada requisito desta capacidade é uma user story. As prioridades
     (P1) ainda não refletem uma ordenação real de valor — revise com
     /speckit:clarify. -->

### User Story 1 - Full coverage target for code layers (Priority: P1)

Code-bearing layers (backend and mobile-app) SHALL enforce 100% line coverage as
a required quality gate, failing the build when coverage is below the threshold.

**Acceptance Scenarios**:

1. **When** the backend `check` (or coverage verification) task runs with all tests passing at 100% line coverage, **Then** the coverage verification succeeds and the build passes
2. **When** backend line coverage is below 100%, **Then** the coverage verification task fails the build, **And** the report identifies the uncovered classes or lines
3. **When** `flutter test --coverage` produces line coverage below 100%, **Then** the coverage gate fails, **And** the uncovered Dart files are reported

---

### User Story 2 - Backend coverage tooling is wired into the build (Priority: P1)

The backend SHALL include a coverage tool (Kover or JaCoCo) with a verification
rule bound to the Gradle lifecycle so coverage is measured and enforced on every
build.

**Acceptance Scenarios**:

1. **When** the backend test task runs, **Then** a machine-readable coverage report (XML) is generated, **And** a verification rule asserts the configured 100% minimum

---

### User Story 3 - Mobile coverage tooling is wired into the build (Priority: P1)

The mobile-app SHALL produce lcov coverage from `flutter test --coverage` and
apply a threshold check that fails when coverage is below the configured target.

**Acceptance Scenarios**:

1. **When** the mobile test suite runs with coverage enabled, **Then** an `lcov.info` report is generated, **And** a threshold check evaluates it against the 100% target

---

### User Story 4 - Config and script layers have equivalent validation gates (Priority: P1)

Layers without unit-testable code (api-gateway, infrastructure, pki) SHALL define
validation gates — existing `verify-*.sh` scripts, KrakenD config checks, and
`terraform fmt`/`terraform validate` — that are required to pass as their
coverage-equivalent gate.

**Acceptance Scenarios**:

1. **When** the api-gateway validation gate runs, **Then** KrakenD configuration checks and `verify-*.sh` scripts execute, **And** any failure blocks the layer's gate
2. **When** the infrastructure validation gate runs, **Then** `terraform fmt -check` and `terraform validate` run for each supported cloud path, **And** the layer verify scripts run, **And** any failure blocks the layer's gate
3. **When** the pki validation gate runs, **Then** the local CA bootstrap and trust-anchor verification scripts execute, **And** any failure blocks the layer's gate
4. **When** the superproject end-to-end job runs (it requires a live runtime), **Then** the pki negative-mTLS verification script executes against the running gateway path, **And** any failure fails the end-to-end job

---

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: Code-bearing layers (backend and mobile-app) SHALL enforce 100% line coverage as a required quality gate, failing the build when coverage is below the threshold.
- **FR-002**: The backend SHALL include a coverage tool (Kover or JaCoCo) with a verification rule bound to the Gradle lifecycle so coverage is measured and enforced on every build.
- **FR-003**: The mobile-app SHALL produce lcov coverage from `flutter test --coverage` and apply a threshold check that fails when coverage is below the configured target.
- **FR-004**: Layers without unit-testable code (api-gateway, infrastructure, pki) SHALL define validation gates — existing `verify-*.sh` scripts, KrakenD config checks, and `terraform fmt`/`terraform validate` — that are required to pass as their coverage-equivalent gate.

## Success Criteria *(mandatory)*

<!-- Os critérios abaixo derivam diretamente dos requisitos acima;
     refine com métricas próprias ao revisar esta feature. -->

- **SC-001**: Todos os requisitos funcionais acima são verificados por testes automatizados.
- **SC-002**: Todos os cenários de aceitação listados passam sem intervenção manual.
