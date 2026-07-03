## Context

Quantum Bank is a Git superproject with five layer submodules, each in its own
GitHub repository:

| Layer | Tech | Build/test today | Coverage today |
| --- | --- | --- | --- |
| `backend` | Spring Boot Kotlin 4.0.6, Gradle (Kotlin DSL), JDK | JUnit via `spring-boot-starter-test`, ~11 test files | none |
| `mobile-app` | Flutter 3.41 / Dart | `flutter test`, ~11 test files | none |
| `api-gateway` | KrakenD CE config (JSON) + shell | `scripts/verify-bootstrap-scopes.sh` | n/a (config) |
| `infrastructure` | Terraform (aws/gcp/azure) + Docker Compose + shell | `scripts/verify-*.sh`, compose smoke | n/a (config) |
| `pki` | Local CA + shell adapters | `scripts/verify-trust-anchors.sh`, `negative-mtls-tests.sh` | n/a (scripts) |

No coverage tooling and no CI exist. Because each submodule is a separate repo,
CI must work both per-repo (fast feedback on a single layer) and at the
superproject level (whole-solution gate). Constraints from `project.md` apply:
gateway-only communication, no permissive TLS, real secure path exercised.

## Goals / Non-Goals

**Goals:**
- Enforce 100% line coverage on code layers (backend, mobile-app) as a
  fail-closed gate.
- Provide equivalent required validation gates for config/script layers
  (api-gateway, infrastructure, pki).
- Deliver a GitHub Actions pipeline: one reusable workflow per layer plus a
  superproject orchestrator, with required status checks blocking merges to
  `main`.
- Keep feedback fast: layers run in parallel; the e2e job is opt-in.

**Non-Goals:**
- No change to application/runtime behavior, APIs, or the secure gateway flow.
- No new product features or spec changes to existing capabilities.
- No cloud deployment from CI (Terraform is validated, not applied).
- No mutation/branch-coverage mandate in v1 (line coverage is the enforced
  metric; branch coverage may be added later).

## Decisions

### Decision 1: Coverage metric = 100% line coverage, fail-closed
Enforce line coverage at 100% on code layers. Rationale: a single, unambiguous,
tool-supported metric that maps directly to the spec's "full coverage target".
- **Alternative considered**: branch/mutation coverage — higher assurance but
  slower, noisier, and harder to reach 100% deterministically; defer to a later
  change.
- **Escape hatch**: explicit, reviewed per-file exclusions (generated code,
  framework bootstrap `main`) documented in the build config, so 100% stays
  honest rather than gamed.

### Decision 2: Backend uses Kover
Use JetBrains **Kover** Gradle plugin over JaCoCo. Rationale: Kotlin-native,
understands Kotlin constructs (inline, data classes) more accurately than JaCoCo,
and binds a `koverVerify` rule to `check`.
- **Alternative considered**: JaCoCo — mature but weaker Kotlin fidelity and more
  boilerplate for the same result.

### Decision 3: Mobile enforces lcov threshold in a small script/step
Run `flutter test --coverage` to emit `coverage/lcov.info`, then a threshold
check (CI step or `scripts/check-coverage.sh`) computes line coverage and fails
below target. Rationale: Flutter has no built-in coverage gate; an explicit lcov
check is portable across local and CI.
- **Alternative considered**: a third-party coverage-enforcer package — avoided
  to keep dependencies minimal; a ~20-line lcov parser suffices.

### Decision 4: Reusable per-layer workflows + superproject orchestrator
Each submodule repo gets its own `.github/workflows/ci.yml` (fast per-layer
feedback). The superproject gets an orchestrator workflow that checks out
submodules (`actions/checkout` with `submodules: recursive`) and runs each
layer's job, plus an optional e2e job.
- **Alternative considered**: only a superproject workflow — loses fast per-repo
  feedback and forces full checkout for a one-line gateway config change.
- **Alternative considered**: `workflow_call` reuse across repos — powerful but
  cross-repo reusable workflows add auth/permission complexity; start with
  per-repo `ci.yml` duplicating a small, stable job shape.

### Decision 5: E2E job is opt-in, Compose-based
The Docker Compose e2e job (protected request through KrakenD to backend) is
gated (label or manual/`workflow_dispatch`) rather than on every PR. Rationale:
it is the slowest, most flaki-prone job; per-layer gates already catch most
regressions.

### Decision 6: Required status checks enforce the gate on `main`
Branch protection on `main` marks the pipeline jobs as required status checks so
merges block until green. This is where "fail-closed" is actually enforced at the
process level.

## Risks / Trade-offs

- **100% coverage is hard to reach on the first pass** → Roll out per layer:
  first add tooling reporting-only, raise threshold to 100% once gaps are filled;
  use documented, reviewed exclusions for genuinely untestable bootstrap code.
- **100% invites low-value tests that game the metric** → Pair the gate with code
  review; coverage is necessary-not-sufficient, reviewers still judge test
  quality.
- **Submodule SHA drift** → superproject CI tests the pinned submodule SHAs; a
  layer fix must be committed in the submodule and the pointer bumped in the
  superproject (already the repo's workflow).
- **E2E flakiness (TLS/cert/timing)** → keep e2e opt-in and out of the required
  merge gate initially; stabilize before promoting to required.
- **Cross-repo secrets for e2e** (certs, Keycloak) → use ephemeral local CA and
  Compose fixtures; never commit real secrets; no real Pix rails.
- **Runner cost/time** → parallelize layer jobs; cache Gradle and Flutter/pub
  dependencies to keep runtime low.

## Migration Plan

1. Backend: add Kover, set verification to 100% (reporting-only first), close
   gaps, then bind `koverVerify` to `check`.
2. Mobile: add `--coverage` + lcov threshold script; close gaps to 100%.
3. api-gateway / infrastructure / pki: wrap existing `verify-*.sh` +
   `terraform fmt/validate` + KrakenD checks as the required gate.
4. Add per-repo `.github/workflows/ci.yml` to each submodule.
5. Add superproject orchestrator workflow (submodule checkout + fan-out + opt-in
   e2e).
6. Enable branch protection / required status checks on `main` for each repo and
   the superproject.
- **Rollback**: thresholds and workflows are additive config; disable a workflow
  or lower a threshold to revert without touching product code.

## Open Questions

- Kover vs JaCoCo: confirm Kover compatibility with Spring Boot 4 / Kotlin 2.2.21
  toolchain (fallback: JaCoCo).
- Should the e2e job become a required check before or after the first stable
  release?
- Are cross-repo reusable workflows (`workflow_call`) worth adopting once the
  per-repo job shape stabilizes?
