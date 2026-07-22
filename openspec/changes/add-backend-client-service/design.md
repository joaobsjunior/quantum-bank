## Context

Quantum Bank's secure path is currently validated only from the first-party
mobile app: Flutter → KrakenD → Spring Boot backend, with OAuth2, OTK/CSR
onboarding, and mTLS on the banking listener. There is no representation of a
*second-party* caller — an external company system integrating with the bank as
a partner would. The gateway already enforces OAuth2 + mTLS on the banking
listener (`8443`) and OAuth2 on the bootstrap listener (`8080`), and the backend
performs defense-in-depth token validation, so the boundary a partner would hit
already exists. What is missing is a client that exercises it from the outside
and a way for a person to drive those flows by hand.

This change adds `backend-client`, a Spring Boot Kotlin service that behaves like
an external service principal (not a device/end user) and ships a web console for
manual, observable flow execution. It reuses the existing gateway routes,
OAuth2 issuer, and PKI rather than introducing a new protocol surface.

## Goals / Non-Goals

**Goals:**
- Prove the gateway-fronted, OAuth2 + mTLS boundary works for machine-to-machine
  partner traffic, not only for the mobile app.
- Exercise the same banking capabilities the mobile app uses (Pix success/error
  simulation, statement, profile) as an external caller, through KrakenD only.
- Give a person a console to run every flow, pick the Pix scenario, and observe
  the request, gateway response, `correlationId`, and problem-details errors.
- Keep the new service fail-closed: no token or no service certificate means no
  call, never a permissive fallback.

**Non-Goals:**
- No real Pix settlement rails (v1 remains simulated).
- No new business logic in the backend; the backend contract is unchanged.
- No direct client-to-backend path and no new bypass of OAuth2/mTLS.
- No production partner-onboarding portal; this is a local validation client.

## Decisions

### D1: Service principal via OAuth2 client-credentials, not a device/user
The `backend-client` authenticates as a confidential Keycloak client
`quantum-bank-backend-client` using the client-credentials grant, with audience
`quantum-bank-api`. **Why:** a partner service is a machine identity; modeling it
as a device (OTK/CSR onboarding) or an end user (Authorization Code) would
misrepresent the trust relationship and duplicate the mobile flow.
*Alternatives:* (a) reuse `quantum-bank-mobile` — rejected, conflates identities
and the spec forbids reusing the mobile client; (b) OTK onboarding — rejected,
OTK is device-bound.

### D2: Service mTLS identity is PKI-issued and provisioned at deploy time
The service client certificate uses a dedicated PKI profile
`quantum-bank-service-client-v1`, issued via the existing CA/CSR path and
provisioned into the local runtime (infrastructure) rather than obtained through
the interactive OTK bootstrap. **Why:** the OTK bootstrap is designed for device
onboarding driven by a human first launch; a headless service should receive its
identity from the PKI/infrastructure pipeline. *Alternative:* have
`backend-client` self-onboard through `POST /auth/otk` + `POST /auth/csr` —
kept as an open question (see below), but not required for v1 validation.

### D3: Server-rendered console in the same Spring Boot app
The console is a server-rendered UI (Thymeleaf) served by the `backend-client`
Spring Boot app itself, calling internal handlers that invoke the
external-integration client. **Why:** keeps the service a single deployable unit,
avoids a separate SPA build/toolchain, and matches the "backend with a visual
interface" intent. *Alternative:* a bundled JS SPA — more moving parts for a
local validation tool; deferred.

### D4: Reuse existing gateway routes; no backend contract change
`backend-client` calls the existing banking routes (`POST /pix/transfers`,
`GET /statements`, `GET /profile`) on the `8443` listener with a bearer token and
the service client certificate. **Why:** the point is to validate the *existing*
boundary from a new caller; changing routes would defeat that. The only
infrastructure additions are the new Keycloak client, the service certificate
trust, and the Compose service.

### D5: New Git submodule tracking `main`
`backend-client` is its own repository, added to the superproject as a submodule
with `branch = main` (consistent with the other layer submodules), and listed in
the README/AGENTS layout tables.

### D6: Everything runs in Docker — no host toolchain
Build, test (including the 100% Kover coverage gate), and runtime all happen in
Docker. The repo ships a **multi-stage Dockerfile**: a build stage on a
Gradle + JDK 17 image runs `gradle check` (compile + tests + coverage verify),
and a slim runtime stage on a JRE 17 image runs the packaged Spring Boot jar.
The service joins the existing `infrastructure/compose.yaml` with a `build:`
context (like `backend` and `api-gateway`), so `docker compose up` brings up the
whole stack. **Why:** the user requires that nothing is installed on the host and
that the whole stack runs via the project's Docker Compose; a self-contained
image also makes CI reproducible. *Alternatives:* (a) host Gradle/JDK — rejected,
violates the no-host-install constraint; (b) prebuilt image in a registry —
deferred, the Compose `build:` context keeps the local flow self-contained.

### D7: Dockerized build/test workflow and CI
Local verification uses `docker build --target test .` (or the build stage of the
multi-stage image), and CI builds that same stage so the 100% coverage gate runs
in the container rather than on a host runner toolchain. The Gradle wrapper (jar
included) is committed so the Docker build is hermetic and version-pinned.
**Why:** keeps a single source of truth for the toolchain (the image) and avoids
host/CI drift.

## Risks / Trade-offs

- **Gateway/backend may reject the service token's claim shape** (e.g. missing
  `azp`/`scope` for a client-credentials token) → define the client's scope
  (`quantum-bank-api` audience + a service scope) in the realm import and verify
  against the existing claim-validation requirement before wiring flows.
- **Service certificate trust not accepted by the gateway** → issue under the
  same `root-ca` the banking listener already trusts (`enable_mtls` +
  `ca_certs: [root-ca.crt]`), so no gateway trust change is needed; verify the
  handshake in the Docker Compose stack.
- **Console could tempt a direct-to-backend "debug" shortcut** → the console spec
  forbids any bypass control; all flows go through the external-integration
  client only.
- **Scope creep into a real partner portal** → explicitly a local validation
  client (Non-Goals); no registration/self-service.

## Migration Plan

1. Create the `backend-client` repository (Spring Boot Kotlin) with the
   external-integration client, the Thymeleaf console, a committed Gradle wrapper,
   and the multi-stage Dockerfile (build/test/coverage + runtime stages).
2. Add the Keycloak `quantum-bank-backend-client` client + scope to the realm
   import; issue the `quantum-bank-service-client-v1` certificate in PKI (extend
   the runtime-cert bootstrap script); add the Compose service (with `build:`
   context) and trust material in infrastructure.
3. Confirm gateway mTLS trust: the service cert chains to the `root-ca` the
   banking listener already trusts, so no gateway change is expected.
4. Register the submodule in the superproject (`.gitmodules` with `branch = main`,
   README/AGENTS tables), bump the pointer.
5. Validate end-to-end **in Docker**: `docker build` runs the coverage gate, and
   `docker compose up` brings up the whole stack including `backend-client`; then
   archive the change.

Rollback: remove the Compose service, the realm client, and the submodule
pointer; no backend or mobile behavior changes to revert. Nothing is installed on
the host, so there is no host-level cleanup.

## Open Questions

- Should `backend-client` optionally self-onboard through the OTK/CSR flow to
  demonstrate service certificate issuance at runtime, or is deploy-time
  provisioning (D2) sufficient for v1?
- Does the client-credentials token need a dedicated scope beyond the
  `quantum-bank-api` audience to satisfy per-endpoint gateway scope enforcement,
  and if so which scopes map to Pix/statement/profile for a service caller?
