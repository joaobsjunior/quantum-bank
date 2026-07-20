## Why

Quantum Bank already uses OpenSpec for planning, and the `planning-governance`
spec declares that the repository SHALL NOT keep parallel non-OpenSpec planning
artifact trees. But a second, richer spec-driven-development (SDD) tree still
lives inside the five layer submodules under `*/docs/contracts/`: ten contract
files that carry the authoritative implementation detail (OTK state machine,
CSR validation steps, OAuth2 scope matrices, negative token/mTLS matrices, the
two-listener gateway topology, route responsibility tables, certificate
lifecycle, and mobile bootstrap states) using their own requirement vocabulary
(`CONT-01/02`, `AUTH-01/02/03`, `PKI-01/02/03`, `MTLS-01/02`, `GATE-01`,
`PIX-API-01`, `STMT-API-01`, `PROF-API-01`, and `D-09..D-28` decisions).

None of those requirement IDs are referenced anywhere under `openspec/`, so the
detailed contract knowledge is currently outside the OpenSpec source of truth.
This change migrates that SDD content into the OpenSpec specs and removes the
parallel contract tree so OpenSpec becomes the single source of truth in fact,
not just in policy.

## What Changes

- Fold the substantive requirements from the ten `docs/contracts/*.md` files
  into the existing OpenSpec specs as new, granular requirements:
  - `secure-gateway-communication`: published route surface, per-endpoint OAuth2
    scope enforcement, fail-closed negative-token behavior, the two-listener
    (bootstrap OAuth2-only / banking OAuth2+mTLS) topology, gateway-to-backend
    mTLS as a distinct boundary, backend defense-in-depth token validation and
    the gateway-header trust boundary, and RFC 9457 problem-details error shape.
  - `onboarding-pki`: OTK binding fields and TTL, the OTK state machine,
    CSR-submission validation and private-key rejection, bootstrap audit events,
    stable OTK/CSR error codes, PKI-owned certificate profiles and lifecycle,
    the local CA adapter with an OpenXPKI swap-in boundary, revocation
    representation, trust-anchor publication, and the mobile runtime
    key/certificate-ready state model.
  - `backend-simulation-api`: the route-to-backend responsibility mapping,
    scenario-driven Pix persistence (`SUCCESS`/`ERROR`), deterministic statement
    and profile responses with `correlationId`, and the `profile:write` scope on
    `PUT /profile`.
  - `mobile-banking-journeys`: gateway-named origins only (with the forbidden
    origin strings and the `verify-gateway-only.sh` gate), the client
    preconditions for protected calls, problem-details parsing, and app-selected
    Pix scenarios.
  - `deployment-infrastructure`: the required local service topology, the local
    Keycloak issuer (realm, clients, scopes, audience, ports), required trust
    material and runtime configuration, and the gateway listener ports/paths.
- Delete the ten `*/docs/contracts/*.md` files from the five submodules once
  their content lives in OpenSpec, so no parallel SDD tree remains.
- Keep `api-gateway/openapi/quantum-bank-v1.yaml` as the API contract of record;
  the OpenSpec specs reference it rather than duplicate it.

## Capabilities

### Modified Capabilities
- `secure-gateway-communication`: adds gateway scope enforcement, negative-token
  and mTLS-topology requirements, and the backend trust-boundary requirement.
- `onboarding-pki`: adds OTK/CSR, certificate-lifecycle, and mobile-bootstrap
  detail migrated from the backend, PKI, and mobile contracts.
- `backend-simulation-api`: adds the route responsibility mapping and
  scenario-driven simulation persistence detail.
- `mobile-banking-journeys`: adds gateway-origin configuration, client
  preconditions, and problem-details parsing detail.
- `deployment-infrastructure`: adds the local issuer, service topology, and
  runtime configuration detail.

### New Capabilities
<!-- None. This change migrates existing contract detail into existing specs; it
     introduces no new capability that was not already governed by a contract. -->

## Impact

- **OpenSpec**: five delta specs enriching existing specs; no spec is removed.
- **Submodules** (`api-gateway`, `backend`, `infrastructure`, `mobile-app`,
  `pki`): the ten `docs/contracts/*.md` files are deleted; each submodule keeps
  its code, `openapi/`, and `scripts/` unchanged.
- **README pointers**: any "Phase 1 Contract Ownership" references in the layer
  READMEs that point at `docs/contracts/` are repointed to the OpenSpec specs.
- **No runtime/product behavior change**: this is a documentation-source
  migration; no application code, API surface, or the secure gateway flow is
  altered.
