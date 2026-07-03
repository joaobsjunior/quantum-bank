## ADDED Requirements

### Requirement: Every layer has a CI build-and-test job
Each layer submodule SHALL have a GitHub Actions job that builds the layer, runs
its tests or validation, and enforces its coverage or validation gate, covering
all five layers (mobile-app, backend, api-gateway, infrastructure, pki).

#### Scenario: Layer job passes
- **WHEN** a layer's CI job runs on a commit that builds and passes all tests and gates
- **THEN** the job reports success

#### Scenario: Layer job fails on test failure
- **WHEN** a layer's tests fail or its coverage/validation gate is not met
- **THEN** the layer's CI job fails
- **AND** the failure is surfaced as a status check

### Requirement: Superproject orchestrates all layers
The superproject SHALL provide a GitHub Actions workflow that checks out
submodules and runs every layer's job, so a single pipeline covers the whole
solution on pull requests and pushes to `main`.

#### Scenario: Pull request triggers the pipeline
- **WHEN** a pull request targets `main`
- **THEN** the superproject workflow runs each layer's build-test-coverage job

#### Scenario: Submodules are checked out
- **WHEN** the superproject workflow starts
- **THEN** it checks out the required submodule contents before running layer jobs

### Requirement: Coverage and validation failures block merges
The pipeline SHALL fail-closed: any layer job, coverage-threshold failure, or
validation failure blocks the overall pipeline result and the associated required
status check.

#### Scenario: A coverage gate fails in CI
- **WHEN** any layer reports coverage below its configured threshold in CI
- **THEN** the pipeline result is failure
- **AND** the required status check does not pass

#### Scenario: All gates pass
- **WHEN** every layer job, coverage gate, and validation gate passes
- **THEN** the pipeline result is success
- **AND** the required status check passes

### Requirement: Optional end-to-end validation job
The pipeline SHALL support an end-to-end job that brings up the solution via
Docker Compose to exercise the secure gateway path across layers.

#### Scenario: End-to-end job runs
- **WHEN** the end-to-end job is enabled and runs
- **THEN** it starts the Compose environment and exercises a protected request through KrakenD to the backend
- **AND** a failure in the end-to-end flow fails the job
