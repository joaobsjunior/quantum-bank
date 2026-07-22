# Quantum Bank

Quantum Bank is a mobile banking project with Flutter, Spring Boot Kotlin,
KrakenD, PKI/mTLS, OAuth2, and Terraform deployment paths. The v1 goal is to
prove a secure end-to-end flow from the mobile app through KrakenD to the
backend, including Pix success and error simulation.

## Repository Layout

This repository is a superproject. The implementation lives in Git submodules:

| Path | Purpose |
| --- | --- |
| `mobile-app` | Flutter 3.41 mobile app for Pix, statement, and profile journeys. |
| `backend` | Spring Boot Kotlin backend with OAuth2-protected banking APIs. |
| `backend-client` | Spring Boot Kotlin external-service simulator with a web console; calls the bank through KrakenD via OAuth2 client-credentials + mTLS. |
| `api-gateway` | KrakenD gateway configs for bootstrap and mTLS banking traffic. |
| `pki` | Local CA, runtime certificates, CSR signing, and PKI lifecycle scripts. |
| `infrastructure` | Docker Compose local runtime and Terraform cloud paths. |
| `openspec` | OpenSpec project context, specs, and change workflow. |

## Prerequisites

- Git with SSH access to the submodule repositories.
- Docker and Docker Compose.
- Java 17 for local backend commands outside Docker.
- Flutter 3.41 / Dart 3.11.x for local mobile commands outside Docker.
- Optional: Terraform. If Terraform is not installed, the infrastructure
  validation script can run the official Terraform Docker image.

## Clone

Fresh clone:

```sh
git clone --recursive git@github.com:joaobsjunior/quantum-bank.git
cd quantum-bank
```

If the repository was cloned without submodules:

```sh
git submodule update --init --recursive
```

Check submodule state:

```sh
git submodule status --recursive
```

## Run the Local End-to-End Stack

The executable local runtime is in `infrastructure/compose.yaml`. It starts:

- Keycloak local OAuth2 issuer on `http://localhost:8180`
- Spring Boot backend inside the Compose network
- KrakenD bootstrap listener on `https://localhost:8080`
- KrakenD banking listener on `https://localhost:8443`

Generate local runtime certificates first:

```sh
cd pki
scripts/bootstrap-runtime-certs.sh
cd ../infrastructure
```

Build and start the runtime:

```sh
docker compose --env-file .env.example build
docker compose --env-file .env.example up -d keycloak backend gateway-bootstrap gateway-banking
```

Run the local smoke test:

```sh
docker compose --env-file .env.example --profile smoke run --rm smoke-tests
```

Expected final output:

```text
local-e2e-smoke-ok
```

Stop the stack:

```sh
docker compose --env-file .env.example down
```

## Local Validation Commands

Run OpenSpec validation from the repository root:

```sh
openspec validate --all --strict
openspec list --specs
```

Run backend tests:

```sh
cd backend
./gradlew test
```

Run focused backend OTK/CSR tests:

```sh
cd backend
./gradlew test --tests "*Otk*" --tests "*Csr*"
```

Run mobile checks:

```sh
cd mobile-app
flutter pub get
dart test
bash scripts/verify-gateway-only.sh
```

Run gateway/source configuration checks:

```sh
cd api-gateway
bash scripts/verify-bootstrap-scopes.sh
```

Run PKI checks:

```sh
cd pki
scripts/verify-trust-anchors.sh
scripts/negative-mtls-tests.sh
```

Run infrastructure checks:

```sh
cd infrastructure
scripts/verify-keycloak-config.sh
scripts/verify-local-e2e-config.sh
scripts/verify-terraform-config.sh
```

## API Entrypoints

Use KrakenD, not the backend, for app-facing traffic:

| Entrypoint | URL | Purpose |
| --- | --- | --- |
| Keycloak | `http://localhost:8180` | Local OAuth2 issuer for development. |
| Gateway bootstrap | `https://localhost:8080` | OTK and CSR bootstrap routes before mobile client cert provisioning. |
| Gateway banking | `https://localhost:8443` | Protected banking APIs requiring OAuth2 and app-to-gateway mTLS. |

The backend is intentionally reachable only inside the Compose network in the
local end-to-end runtime.

## Mobile Local Configuration

The mobile app uses gateway-named origins from:

```text
mobile-app/config/api.env.example
```

Local defaults:

- `GATEWAY_BOOTSTRAP_BASE_URL=https://localhost:8080`
- `GATEWAY_BASE_URL=https://localhost:8443`

Do not configure the mobile app to call the backend directly.

## Terraform

Terraform roots live under `infrastructure/terraform`:

| Path | Purpose |
| --- | --- |
| `infrastructure/terraform/aws` | AWS ECS/Fargate deployment path. |
| `infrastructure/terraform/gcp` | Google Cloud Run v2 deployment path. |
| `infrastructure/terraform/azure` | Azure Container Apps deployment path. |

Validate all Terraform paths:

```sh
cd infrastructure
scripts/verify-terraform-config.sh
```

## OpenSpec Workflow

OpenSpec is the planning source of truth.

Useful commands:

```sh
openspec list --specs
openspec show secure-gateway-communication
openspec validate --all --strict
openspec new change <change-id>
```

## Notes

- Runtime private keys and generated certificate material are intentionally
  ignored by Git.
- v1 Pix behavior is simulated. It does not call real Pix settlement rails.

## Testing & CI

Every layer has a GitHub Actions workflow (`.github/workflows/ci.yml` in each
submodule) that builds/tests and enforces its gate:

| Layer | Gate |
| --- | --- |
| `backend` | `./gradlew check` — Kover 100% line-coverage verification |
| `mobile-app` | `flutter test --coverage` + `scripts/check-coverage.sh` (100%) |
| `api-gateway` | `scripts/ci-validate.sh` — KrakenD config check + bootstrap scopes |
| `infrastructure` | `scripts/ci-validate.sh` — terraform fmt/validate + config checks |
| `pki` | `scripts/ci-validate.sh` — bootstrap local CA + trust-anchor verify |

The superproject workflow (`.github/workflows/ci.yml`) checks out all submodules
(`submodules: recursive`) and runs every layer gate in parallel; a final `gate`
job aggregates them so a single required status check enforces fail-closed
behavior on `main`. An opt-in `e2e` job (manual `workflow_dispatch` or the `e2e`
PR label) brings the solution up via Docker Compose and exercises the secure
gateway path.

Submodule checkout of these private repos needs a `SUBMODULES_TOKEN` secret (a
PAT with read access) or SSH deploy keys. Enable branch protection with the CI
checks as required to block merges until green.
