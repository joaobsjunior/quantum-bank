## Why

Validating Quantum Bank end to end requires runnable local infrastructure and a
credible cloud deployment story. This capability defines the local container
setup (backend + KrakenD, plus the security components needed for e2e) and the
Terraform paths for the major clouds so the solution can be validated locally and
deployed per provider.

## What Changes

- **Dockerize** the backend and KrakenD gateway so the local environment starts
  with the backend reachable by KrakenD and KrakenD as the mobile-facing entry.
- Provide a **Docker Compose** setup that supports end-to-end validation of
  backend, gateway, OAuth2, mTLS, and optional PKI components together.
- Add **Terraform paths for AWS, GCP, and Azure**, each validating independently
  without hiding provider-specific differences behind a single generic module.

## Impact

- **infrastructure**: Dockerfiles/compose for local e2e and per-cloud Terraform
  (aws/gcp/azure) with fmt/validate tooling.
- **Enables**: the superproject e2e job and cross-layer validation.
