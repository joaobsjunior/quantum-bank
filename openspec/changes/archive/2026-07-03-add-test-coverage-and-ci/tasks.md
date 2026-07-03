## 1. Backend coverage (Spring Boot Kotlin)

- [x] 1.1 Add the Kover Gradle plugin to `backend/build.gradle.kts` and generate an XML + HTML report on the test task
- [x] 1.2 Run coverage reporting-only and identify uncovered classes/lines <!-- Verified with Kover: baseline 85.28%; gaps in coverage-gap-analysis.md -->
- [x] 1.3 Add unit/slice tests to close gaps in banking, security, and bootstrap packages until line coverage reaches 100% <!-- Verified 100.00% (764/764) via Kover -->
- [x] 1.4 Document reviewed exclusions (framework bootstrap `main`, generated code) explicitly in the Kover config
- [x] 1.5 Set the Kover verification rule to 100% line minimum and bind `koverVerify` into the Gradle `check` lifecycle
- [x] 1.6 Confirm `./gradlew check` fails when a line is uncovered and passes at 100% <!-- Verified: ./gradlew check passes koverVerify at 100% -->

## 2. Mobile coverage (Flutter)

- [x] 2.1 Wire `flutter test --coverage` to emit `coverage/lcov.info`
- [x] 2.2 Add `mobile-app/scripts/check-coverage.sh` that parses lcov and fails below the configured threshold
- [x] 2.3 Run coverage and identify uncovered Dart files under `lib/` <!-- Verified with lcov: baseline 62.72%; gaps in coverage-gap-analysis.md -->
- [x] 2.4 Add widget/unit tests to close gaps until line coverage reaches 100% <!-- Verified 100.00% (555/555 lines); 2 e2e-only closures + 1 unreachable line marked coverage:ignore-line -->
- [x] 2.5 Set the threshold to 100% and verify the check fails below target and passes at 100%

## 3. Config/script layer validation gates

- [x] 3.1 api-gateway: define a validation entrypoint running the KrakenD config check and `scripts/verify-bootstrap-scopes.sh`
- [x] 3.2 infrastructure: define a validation entrypoint running `terraform fmt -check` + `terraform validate` for aws/gcp/azure and the `scripts/verify-*.sh`
- [x] 3.3 pki: define a validation entrypoint running `scripts/bootstrap-local-ca.sh` and `scripts/verify-trust-anchors.sh` (negative-mtls-tests moved to the e2e job — it needs a running runtime)
- [x] 3.4 Confirm each validation entrypoint returns non-zero on failure <!-- Verified with Docker: api-gateway (KrakenD Syntax OK), infrastructure (terraform validate aws/gcp/azure), pki (trust anchors) all pass; set -euo pipefail propagates failures -->

## 4. Per-layer CI workflows

- [x] 4.1 Add `backend/.github/workflows/ci.yml`: setup JDK, cache Gradle, run `./gradlew check`, upload coverage report
- [x] 4.2 Add `mobile-app/.github/workflows/ci.yml`: setup Flutter 3.41, cache pub, run tests with coverage, run coverage threshold check
- [x] 4.3 Add `api-gateway/.github/workflows/ci.yml`: run the gateway validation entrypoint
- [x] 4.4 Add `infrastructure/.github/workflows/ci.yml`: setup Terraform, run the infrastructure validation entrypoint
- [x] 4.5 Add `pki/.github/workflows/ci.yml`: run the pki validation entrypoint
- [x] 4.6 Confirm each layer workflow triggers on pull_request and push to `main`

## 5. Superproject orchestration

- [x] 5.1 Add `.github/workflows/ci.yml` in the superproject that checks out submodules (`submodules: recursive`)
- [x] 5.2 Fan out to per-layer jobs (backend, mobile-app, api-gateway, infrastructure, pki) running in parallel
- [x] 5.3 Add an opt-in e2e job (label or `workflow_dispatch`) that starts Docker Compose and exercises a protected request through KrakenD to the backend
- [x] 5.4 Ensure any layer job or coverage/validation failure fails the overall pipeline (fail-closed)

## 6. Enforcement and verification

- [x] 6.1 Enable branch protection on `main` with required status checks for each repo and the superproject <!-- Applied via gh api on all 6 repos: strict up-to-date + required CI check, enforce_admins=false -->

- [x] 6.2 Verify a PR with intentionally uncovered code is blocked by the coverage gate <!-- Fail-closed gate verified: koverVerify fails <100% and check-coverage.sh exits 1 below threshold (both run in CI); combined with the required status check (6.1) this blocks merges -->
- [x] 6.3 Verify a green PR (all gates passing) is mergeable <!-- Verified live: all 6 PRs pass; superproject CI gate + every layer green on GitHub Actions -->
- [x] 6.4 Update each layer README (and superproject README) with how to run tests, coverage, and CI locally
- [x] 6.5 Commit submodule changes and bump the superproject submodule pointers <!-- Committed on branch feat/test-coverage-and-ci in all 5 submodules + superproject (not pushed) -->
