## Why

Quantum Bank proves a secure end-to-end path from a first-party mobile app
through KrakenD to the backend, but it has no representation of a *second-party*
caller: an external company service that consumes the bank's internal
capabilities the way a real partner integration would. Adding a `backend-client`
service closes that gap — it exercises the exact same gateway-fronted,
OAuth2- and mTLS-protected boundary from the outside, and gives a person a visual
console to drive every flow (like the mobile app), so the security foundation is
validated for machine-to-machine partner traffic, not only for the mobile app.

## What Changes

- Add a new submodule/repository `backend-client`: a Spring Boot Kotlin service
  that simulates an external company system calling Quantum Bank's internal
  capabilities **only through the KrakenD API gateway** (never directly).
- Run the **entire stack in Docker**: `backend-client` is built, tested (with its
  100% coverage gate), and run inside containers, and it joins the existing
  `infrastructure/compose.yaml` alongside `keycloak`, `backend`, `api-gateway`,
  and `pki`. No Java/Gradle toolchain is installed on the host — build and test
  happen in a multi-stage Docker image.
- The `backend-client` authenticates as its own registered OAuth2 client
  (client-credentials grant) and presents a PKI-issued **service** mTLS identity
  on the banking listener — it is a service principal, not a device/end user.
- The `backend-client` exercises the same banking flows the mobile app does —
  Pix transfer (success/error simulation), account statement, and profile /
  registration data — consuming them as an external caller and propagating
  `correlationId`, parsing RFC 9457 problem-details errors.
- Add a human-facing web **console** inside `backend-client` so a person can
  manually trigger every flow, choose the Pix success/error scenario, and
  observe the outbound request, the gateway response, correlation IDs, and error
  bodies — parity with the mobile journeys from the external-caller perspective.
- Wire the new service into the local runtime: Docker Compose service, an OAuth2
  client registration in the local IdP, and service certificate/trust material
  for gateway mTLS.
- Register the new submodule in the superproject (`.gitmodules`, README and
  AGENTS layout tables) tracking its `main` branch.

## Capabilities

### New Capabilities
- `external-service-integration`: an external service consuming Quantum Bank
  internal banking capabilities exclusively through KrakenD, using its own
  OAuth2 client-credentials token and a PKI-issued service mTLS identity,
  exercising the Pix / statement / profile flows, propagating correlation IDs,
  and handling problem-details errors — with a fail-closed posture when token or
  certificate material is missing or invalid.
- `backend-client-console`: a web console in the `backend-client` that lets a
  person manually run each external-integration flow, select the Pix scenario,
  and inspect the request, gateway response, correlation ID, and error payload,
  without ever exposing a direct-to-backend path.

### Modified Capabilities
- `deployment-infrastructure`: add the `backend-client` service to the Docker
  Compose local runtime and service topology, register its OAuth2
  client-credentials client (and scopes) in the local IdP, and provision its
  service certificate and trust anchors for gateway mTLS.

## Impact

- **New repo/submodule** `backend-client` (Spring Boot Kotlin 4.0.6, web console
  UI) with a **multi-stage Dockerfile** that builds, runs the 100%-coverage test
  gate, and packages the runtime image — no host toolchain required. Added to the
  superproject and its layout/docs tables.
- **api-gateway**: banking routes must accept the registered service client over
  the OAuth2 + mTLS banking listener; the service certificate is issued under the
  CA the gateway already trusts, so no gateway trust change is expected.
- **infrastructure**: new Docker Compose service (with build context) in
  `compose.yaml`, IdP client + scopes, and service certificate/trust material in
  the local runtime.
- **pki**: issue a service client certificate for `backend-client` via the
  existing CA / CSR issuance path.
- **Superproject**: new submodule pointer, `.gitmodules` entry (`branch = main`),
  and README/AGENTS layout updates.
- No real Pix rails and no direct client-to-backend calls are introduced.
