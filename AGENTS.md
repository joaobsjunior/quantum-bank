@/Users/joaobsjunior/.codex/RTK.md

# Quantum Bank Agent Instructions

## Project

Quantum Bank is a mobile banking application with three initial user journeys:
Pix transfer, account statement, and customer registration data. The app is
built in Flutter 3.41 and communicates end to end with a Spring Boot Kotlin
backend through a KrakenD API gateway.

The v1 project prioritizes the secure communication foundation first: OAuth2
authentication, OTK-based onboarding, CSR-driven certificate generation, mTLS,
and PKI. Even if the first screens are simple, they must exercise the real
app -> gateway -> backend flow.

Core value: prove a secure end-to-end banking flow from Flutter through KrakenD
to the backend, including Pix success and error simulation.

## OpenSpec Workflow

This project has been migrated from GSD planning to OpenSpec.

- Use `openspec/` as the source of truth for product context, specs, and change
  proposals.
- Before implementing new behavior, inspect current specs with
  `openspec list --specs` and `openspec show <spec-id>`.
- For new work, create an OpenSpec change first with `/opsx:propose` or
  `openspec new change <change-id>`, then implement only after the proposal and
  tasks are clear.
- Use `/opsx:apply` or the `openspec-apply-change` skill when implementing an
  accepted OpenSpec change.
- Use `/opsx:archive` or `openspec archive <change-id>` after a completed
  change has been implemented and validated.
- Keep `.planning/` as read-only migrated GSD history unless the user explicitly
  asks to remove it.

## Current Specs

- `secure-gateway-communication`: gateway-only traffic, OAuth2 validation, mTLS,
  and fail-closed TLS.
- `onboarding-pki`: OTK onboarding, CSR issuance, and certificate lifecycle.
- `mobile-banking-journeys`: Flutter journeys and secure API configuration.
- `pix-simulation`: success/error Pix simulation without real Pix rails.
- `backend-simulation-api`: Spring Boot Kotlin backend and local simulation
  data.
- `deployment-infrastructure`: Docker Compose and Terraform paths.

## Constraints

- Mobile stack: Flutter 3.41.
- Backend stack: Spring Boot Kotlin 4.0.6.
- API gateway: KrakenD; all mobile/backend communication must pass through it.
- Persistence: H2 or memory database is acceptable for v1.
- Security: OAuth2, OTK, CSR, mTLS, and PKI are core v1 concerns.
- PKI: use KrakenD PKI features only if they satisfy the required certificate
  lifecycle; otherwise use an open source option such as OpenXPKI.
- Deployment: backend and gateway must be dockerized; Terraform paths are
  required for AWS, GCP, and Azure.

## Repository Layout

The superproject coordinates planning. Implementation belongs in the appropriate
layer repository when submodules are initialized:

- `mobile-app`
- `backend`
- `api-gateway`
- `infrastructure`
- `pki`

## Avoid

- Do not make direct app-to-backend calls.
- Do not accept all TLS certificates or use permissive certificate callbacks.
- Do not treat KrakenD as the full PKI lifecycle owner unless it proves it can
  satisfy issuance, renewal, and revocation.
- Do not integrate real Pix rails in v1.
- Do not hide AWS, GCP, and Azure differences behind one generic Terraform
  module.
