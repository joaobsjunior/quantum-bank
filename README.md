# Quantum Bank

Quantum Bank is a mobile banking project with Flutter, Spring Boot Kotlin,
KrakenD, PKI/mTLS, OAuth2, and Terraform deployment paths. The v1 goal is to
prove a secure end-to-end flow from the mobile app through KrakenD to the
backend, including Pix success and error simulation.

All communication is **post-quantum first**: every service-to-service TLS hop
authenticates peers with ML-DSA certificates (FIPS 204; ML-DSA-87 CA, ML-DSA-65
leaves) and negotiates only the `X25519MLKEM768` hybrid key exchange (FIPS 203)
over TLS 1.3. The app-facing listeners (issuer and gateway) serve a dual
identity: the same ML-DSA chain to clients that offer ML-DSA signature schemes,
and an ECDSA P-256 compatibility chain to clients whose TLS stack cannot verify
ML-DSA yet (the Dart/BoringSSL mobile transport, browsers), preferring the
hybrid group and accepting `X25519`; RSA is refused everywhere. See
[docs/pqc-ml-dsa-transport.md](docs/pqc-ml-dsa-transport.md) for the tiers,
the per-stack capability matrix that motivates them, the per-layer design and
the evidence gates.

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
- Docker and Docker Compose (also used to run OpenSSL >= 3.5 for the PKI
  scripts when the host OpenSSL is older).
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

- Keycloak local OAuth2 issuer on `https://localhost:8180` (HAProxy TLS
  terminator in front of Keycloak, dual PKI-issued identity: ML-DSA-65 and
  ECDSA P-256)
- Spring Boot backend inside the Compose network (BCJSSE, ML-DSA only)
- KrakenD bootstrap listener on `https://localhost:8080` (HAProxy terminator, dual identity)
- KrakenD banking listener on `https://localhost:8443` (HAProxy terminator, dual identity, app mTLS from either PKI chain)

Generate local runtime certificates first:

```sh
cd pki
scripts/bootstrap-runtime-certs.sh
cd ../infrastructure
```

Build and start the runtime:

```sh
cp .env.example .env   # then replace every change-me-local-only / changeit value
docker compose --env-file .env build
docker compose --env-file .env up -d keycloak backend gateway-bootstrap gateway-banking backend-client
```

All host ports bind to `127.0.0.1` by default (`BIND_ADDRESS` in `.env`). The
external-service console on `http://localhost:8090` requires the operator
credentials from `BACKEND_CLIENT_CONSOLE_USERNAME` / `BACKEND_CLIENT_CONSOLE_PASSWORD`.

Run the local smoke test:

```sh
docker compose --env-file .env --profile smoke run --rm smoke-tests
docker compose --env-file .env --profile smoke run --rm negative-mtls-tests
docker compose --env-file .env --profile smoke run --rm pqc-handshake-tests
```

Expected final output:

```text
local-e2e-smoke-ok
negative-mtls-ok
pqc-handshake-ok
```

Stop the stack:

```sh
docker compose --env-file .env down
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
bash scripts/ci-validate.sh
```

Run PKI checks (OpenSSL >= 3.5 or Docker):

```sh
cd pki
scripts/ci-validate.sh
scripts/negative-mtls-tests.sh      # needs the running local runtime
scripts/pqc-handshake-tests.sh      # needs the running local runtime and OpenSSL >= 3.5
```

Run the mobile CSR interoperability check (Dart or Docker, OpenSSL >= 3.5 or Docker):

```sh
cd mobile-app
bash scripts/verify-pqc-csr-interop.sh
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
| Keycloak | `https://localhost:8180` | Local OAuth2 issuer for development (post-quantum TLS only; trust `pki/local-ca/trust/root-ca.crt`). |
| Gateway bootstrap | `https://localhost:8080` | OTK and CSR bootstrap routes before mobile client cert provisioning (post-quantum TLS). |
| Gateway banking | `https://localhost:8443` | Protected banking APIs requiring OAuth2 and app-to-gateway post-quantum mTLS (ML-DSA client certificate). |

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

Every origin (issuer and gateways) must be `https`; the app refuses plaintext
origins at startup. No password is compiled into the binary: pass the local
test user's password at build/run time, for example
`flutter run --dart-define=KEYCLOAK_PASSWORD=<value from .env>`.

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
  ignored by Git. `pki/scripts/bootstrap-local-ca.sh` re-issues the tracked
  ML-DSA-87 trust anchors whenever they do not match the local private keys
  (and copies the root into `mobile-app/assets/local-ca/root-ca.crt`), so
  commit the regenerated `pki/local-ca/trust/*.crt` and the mobile asset
  together.
- Classical (RSA/EC) keys, certificates and CSRs are rejected everywhere;
  keys from before the post-quantum migration are moved aside as
  `*.pre-pqc.<timestamp>` by the PKI scripts.
- The backend container needs read access to `pki/local-ca/private/issuing-ca.key`
  for the local sign script; set `PKI_GID` in `.env` to the group that owns that
  directory (see `.env.example`).
- v1 Pix behavior is simulated. It does not call real Pix settlement rails.

## Testing & CI

Every layer has a GitHub Actions workflow (`.github/workflows/ci.yml` in each
submodule) that builds/tests and enforces its gate:

| Layer | Gate |
| --- | --- |
| `backend` | `./gradlew check` — Kover 100% line-coverage verification |
| `mobile-app` | `flutter test --coverage` + `scripts/check-coverage.sh` (100%) |
| `api-gateway` | `scripts/ci-validate.sh` — KrakenD config check + HAProxy terminator check with ML-DSA material + bootstrap scopes + PQC policy |
| `infrastructure` | `scripts/ci-validate.sh` — terraform fmt/validate + config checks |
| `pki` | `scripts/ci-validate.sh` — bootstrap ML-DSA-87 CA + trust-anchor verify + runtime material + PQC policy |

The superproject workflow (`.github/workflows/ci.yml`) checks out all submodules
(`submodules: recursive`) and runs every layer gate in parallel; a final `gate`
job aggregates them so a single required status check enforces fail-closed
behavior on `main`. An opt-in `e2e` job (manual `workflow_dispatch` or the `e2e`
PR label) brings the solution up via Docker Compose and exercises the secure
gateway path, the negative mTLS matrix, the post-quantum handshake evidence and
the mobile CSR interoperability check.

Submodule checkout of these private repos needs a `SUBMODULES_TOKEN` secret (a
PAT with read access) or SSH deploy keys. Enable branch protection with the CI
checks as required to block merges until green.
