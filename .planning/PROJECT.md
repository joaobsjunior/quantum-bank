# Quantum Bank

## What This Is

Quantum Bank is a mobile banking application with three initial user journeys: Pix transfer, account statement, and customer registration data. The app is built in Flutter 3.41 and communicates end to end with a Spring Boot Kotlin backend through a KrakenD API gateway.

The v1 project prioritizes the secure communication foundation first: OAuth2 authentication, OTK-based onboarding, CSR-driven certificate generation, mTLS, and PKI. Even if the first screens are simple, they must exercise the real app -> gateway -> backend flow.

## Core Value

The app must prove a secure end-to-end banking flow from Flutter through KrakenD to the backend, including Pix success and error simulation.

## Requirements

### Validated

- [x] OAuth2 authentication gates backend/API access. Validated in Phase 2 with Keycloak local issuer configuration, KrakenD JWT policy, and backend Resource Server tests.
- [x] Mobile app communicates with backend only through KrakenD API Gateway. Validated in Phase 2 with gateway-only mobile config and automated bypass check.

### Active

- [ ] Flutter 3.41 mobile app exposes Pix transfer, statement, and customer registration data screens.
- [ ] Pix transfer flow lets the app select a success or error scenario and receives the corresponding backend response.
- [ ] Backend is implemented with Spring Boot Kotlin.
- [ ] Backend supports an in-memory database, such as H2, for v1 development and simulation data.
- [ ] App follows the OTK flow to initiate backend communication and certificate provisioning.
- [ ] Runtime CSR flow issues client certificates for mTLS communication.
- [ ] Solution includes PKI capability; if KrakenD does not provide the needed PKI feature, use an open source PKI such as OpenXPKI.
- [ ] Backend and API Gateway are dockerized.
- [ ] Infrastructure roadmap includes Terraform deployment paths for AWS, GCP, and Azure.

### Out of Scope

- Production-grade core banking ledger integration - v1 uses H2/memory data so end-to-end flows can be validated without external banking dependencies.
- Real Pix settlement with banking rails - v1 simulates Pix success and error responses.
- Rich UI polish before security foundation - OAuth2, OTK, CSR, mTLS, and PKI are prioritized even if initial screens stay simple.

## Context

The initial source document is `requisitos.md`. It defines a banking app with Pix transaction, account statement, and customer registration data screens. The user clarified that v1 must work end to end through the backend and that H2 or another memory database is acceptable.

The selected scope is security first. OAuth2, mTLS, CSR, OTK, and PKI should be planned early, even if the first screen implementations are minimal. Pix simulation must support both success and error paths: the app selects the scenario, and the backend returns the matching response through the gateway.

The repository is structured as a multi-repo workspace with Git submodules for each layer: `mobile-app`, `backend`, `api-gateway`, `infrastructure`, and `pki`. The superproject coordinates planning, while implementation commits should land in the appropriate layer repository.

## Constraints

- **Mobile stack**: Flutter 3.41 - requested app technology.
- **Backend stack**: Spring Boot Kotlin 4.0.6 - requested backend technology.
- **API gateway**: KrakenD - all mobile/backend communication should pass through the gateway.
- **Persistence**: H2 or memory database is acceptable for v1 - supports fast local end-to-end validation.
- **Security**: OAuth2, OTK, CSR, mTLS, and PKI are core v1 concerns - this is a banking app and security architecture is the primary risk.
- **PKI dependency**: Use KrakenD PKI features only if they satisfy the required certificate lifecycle; otherwise use an open source option such as OpenXPKI.
- **Deployment**: Backend and gateway must be dockerized; Terraform paths are required for AWS, GCP, and Azure.

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Prioritize security first for v1 | Banking communication security is the highest-risk foundation and should constrain the architecture early. | Phase 2 completed OAuth2 issuer, gateway JWT policy, and backend Resource Server validation before banking UI/business APIs. |
| Use H2 or memory database for v1 | Enables end-to-end backend behavior without external database setup or banking integrations. | - Pending |
| Pix simulation is app-selected and backend-enforced | The app can exercise success and error cases while the backend remains the source of response behavior. | - Pending |
| Route all backend access through KrakenD | Matches the requested architecture and keeps gateway policies central. | Phase 2 validated gateway-only mobile protected API config and backend direct-call rejection. |
| Use OpenXPKI if KrakenD cannot cover PKI requirements | Keeps PKI implementation open source while avoiding unsupported gateway assumptions. | - Pending |
| Use Git submodules per architecture layer | Keeps mobile, backend, gateway, infrastructure, and PKI work independently versioned while preserving a coordinated superproject. | - Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `$gsd-transition`):
1. Requirements invalidated? -> Move to Out of Scope with reason
2. Requirements validated? -> Move to Validated with phase reference
3. New requirements emerged? -> Add to Active
4. Decisions to log? -> Add to Key Decisions
5. "What This Is" still accurate? -> Update if drifted

**After each milestone** (via `$gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check - still the right priority?
3. Audit Out of Scope - reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-05-20 after Phase 2 OAuth2 and gateway authorization*
