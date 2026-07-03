## Purpose

Define the enforced test-coverage quality gates: 100% line coverage on the code
layers (backend, mobile-app) and equivalent required validation gates on the
config/script layers (api-gateway, infrastructure, pki), all fail-closed.

## Requirements

### Requirement: Full coverage target for code layers
Code-bearing layers (backend and mobile-app) SHALL enforce 100% line coverage as
a required quality gate, failing the build when coverage is below the threshold.

#### Scenario: Backend coverage meets target
- **WHEN** the backend `check` (or coverage verification) task runs with all tests passing at 100% line coverage
- **THEN** the coverage verification succeeds and the build passes

#### Scenario: Backend coverage below target
- **WHEN** backend line coverage is below 100%
- **THEN** the coverage verification task fails the build
- **AND** the report identifies the uncovered classes or lines

#### Scenario: Mobile coverage below target
- **WHEN** `flutter test --coverage` produces line coverage below 100%
- **THEN** the coverage gate fails
- **AND** the uncovered Dart files are reported

### Requirement: Backend coverage tooling is wired into the build
The backend SHALL include a coverage tool (Kover or JaCoCo) with a verification
rule bound to the Gradle lifecycle so coverage is measured and enforced on every
build.

#### Scenario: Coverage report is produced
- **WHEN** the backend test task runs
- **THEN** a machine-readable coverage report (XML) is generated
- **AND** a verification rule asserts the configured 100% minimum

### Requirement: Mobile coverage tooling is wired into the build
The mobile-app SHALL produce lcov coverage from `flutter test --coverage` and
apply a threshold check that fails when coverage is below the configured target.

#### Scenario: Coverage artifact is produced
- **WHEN** the mobile test suite runs with coverage enabled
- **THEN** an `lcov.info` report is generated
- **AND** a threshold check evaluates it against the 100% target

### Requirement: Config and script layers have equivalent validation gates
Layers without unit-testable code (api-gateway, infrastructure, pki) SHALL define
validation gates — existing `verify-*.sh` scripts, KrakenD config checks, and
`terraform fmt`/`terraform validate` — that are required to pass as their
coverage-equivalent gate.

#### Scenario: Gateway validation runs
- **WHEN** the api-gateway validation gate runs
- **THEN** KrakenD configuration checks and `verify-*.sh` scripts execute
- **AND** any failure blocks the layer's gate

#### Scenario: Infrastructure validation runs
- **WHEN** the infrastructure validation gate runs
- **THEN** `terraform fmt -check` and `terraform validate` run for each supported cloud path
- **AND** the layer verify scripts run
- **AND** any failure blocks the layer's gate

#### Scenario: PKI validation runs
- **WHEN** the pki validation gate runs
- **THEN** the local CA bootstrap and trust-anchor verification scripts execute
- **AND** any failure blocks the layer's gate

#### Scenario: Negative-mTLS checks run in the e2e job
- **WHEN** the superproject end-to-end job runs (it requires a live runtime)
- **THEN** the pki negative-mTLS verification script executes against the running gateway path
- **AND** any failure fails the end-to-end job
