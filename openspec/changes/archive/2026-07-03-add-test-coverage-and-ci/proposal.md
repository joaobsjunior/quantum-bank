## Why

Quantum Bank is a security-critical banking solution spread across five layer
submodules (mobile-app, backend, api-gateway, infrastructure, pki), yet it has
no automated quality gate: coverage is unmeasured and there is no CI. Regressions
in the secure end-to-end flow (OAuth2, mTLS, PKI, Pix simulation) can reach
`main` undetected. We need enforced test coverage and a GitHub Actions pipeline
so every change is built, tested, and validated automatically before merge.

## What Changes

- Introduce a project-wide **100% test coverage target** as an enforced quality
  gate for every layer that runs code (backend, mobile-app), and equivalent
  full-validation gates for config/script layers (api-gateway, infrastructure,
  pki).
- Add coverage tooling and thresholds per layer:
  - Backend (Spring Boot Kotlin): add **Kover** (or JaCoCo) with a 100%
    verification rule wired into the Gradle `check` lifecycle.
  - Mobile (Flutter): enforce `flutter test --coverage` with a 100% line-coverage
    gate (lcov-based).
  - api-gateway / infrastructure / pki: treat their existing `verify-*.sh`
    scripts, `terraform validate/fmt`, and KrakenD config checks as the coverage
    equivalent, each run in CI and required to pass.
- Add a **GitHub Actions CI pipeline** for the whole solution:
  - One reusable workflow per layer (build + test + coverage/validate).
  - A superproject orchestrator workflow that checks out submodules and fans out
    to each layer job, plus an optional end-to-end job using Docker Compose.
  - Coverage-threshold failure and any validation failure **block the pipeline**.
- Define the **activity flow** (in `tasks.md`) that sequences the rollout across
  all five submodules and the superproject.

## Capabilities

### New Capabilities
- `test-coverage-enforcement`: Coverage targets, per-layer measurement tooling,
  and the fail-closed thresholds that gate merges.
- `continuous-integration`: GitHub Actions pipeline structure — per-layer jobs,
  superproject orchestration, submodule checkout, and required status checks.

### Modified Capabilities
<!-- None. Existing capability specs describe runtime behavior; testing and CI are
     new cross-cutting quality capabilities layered on top and do not change the
     requirements of existing specs. -->

## Impact

- **New CI config**: `.github/workflows/` in the superproject and in each of the
  five layer submodules.
- **Backend**: `build.gradle.kts` gains a coverage plugin and a 100% verification
  rule bound to `check`.
- **Mobile**: test scripts/CI enforce `--coverage` with an lcov threshold check.
- **api-gateway / infrastructure / pki**: existing `scripts/verify-*.sh`,
  `terraform validate`, and KrakenD checks become CI-required.
- **Dependencies**: adds a coverage plugin (backend) and an lcov/coverage-check
  step (mobile); GitHub Actions runners for JDK, Flutter, Terraform, and Docker.
- **Process**: `main` gains required status checks; merges blocked until all
  layer jobs and coverage gates pass.
- **No runtime/product behavior change**: no application code paths, APIs, or the
  secure gateway flow are altered by this change.
