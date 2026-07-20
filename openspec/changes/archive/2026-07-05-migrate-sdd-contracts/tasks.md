## 1. Author delta specs (migrate contract detail into OpenSpec)

- [x] 1.1 `secure-gateway-communication`: migrate `gateway-boundary.md`, `oauth2-gateway-policy.md`, and `oauth2-backend-policy.md` (route surface, scope matrix, negative-token matrix, two-listener mTLS, gateway-to-backend mTLS, backend trust boundary, problem details)
- [x] 1.2 `onboarding-pki`: migrate `otk-csr-contract.md`, `certificate-lifecycle.md`, and `client-bootstrap.md` (OTK binding/TTL, state machine, CSR validation, audit events, error codes, PKI profiles/issuance, local CA adapter + OpenXPKI boundary, revocation, trust anchors, mobile key/cert-ready states)
- [x] 1.3 `backend-simulation-api`: migrate `api-implementation-map.md` (route responsibility mapping, scenario-driven Pix persistence, deterministic statement/profile, `profile:write` scope, problem details)
- [x] 1.4 `mobile-banking-journeys`: migrate `api-client-contract.md` (gateway-named origins, client preconditions, problem-details parsing, app-selected Pix scenario)
- [x] 1.5 `deployment-infrastructure`: migrate `local-runtime-checklist.md` and `oauth2-local-issuer.md` (service topology, Keycloak realm/clients/scopes/ports, trust material and runtime config, listener ports/paths)

## 2. Validate the change

- [x] 2.1 `openspec validate migrate-sdd-contracts --strict` passes
- [x] 2.2 Confirm every migrated contract concern is represented by at least one requirement/scenario

## 3. Remove the parallel contract tree

- [x] 3.1 Delete `api-gateway/docs/contracts/{gateway-boundary,oauth2-gateway-policy}.md`
- [x] 3.2 Delete `backend/docs/contracts/{api-implementation-map,oauth2-backend-policy,otk-csr-contract}.md`
- [x] 3.3 Delete `infrastructure/docs/contracts/{local-runtime-checklist,oauth2-local-issuer}.md`
- [x] 3.4 Delete `mobile-app/docs/contracts/{api-client-contract,client-bootstrap}.md`
- [x] 3.5 Delete `pki/docs/contracts/certificate-lifecycle.md`
- [x] 3.6 Repoint the five layer README contract sections to the OpenSpec specs
- [x] 3.7 Remove the `docs/contracts` reference in `backend` `BackendMtlsX509Test.kt` (doc-assertion test now covered by the `secure-gateway-communication` spec) and clean residual `PKI-02`/`AUTH-01` README mentions

## 4. Land the change

- [ ] 4.1 Commit the deletions and README edits in each affected submodule
- [ ] 4.2 Bump the superproject submodule pointers and commit the OpenSpec change
- [ ] 4.3 Archive the change (`openspec archive migrate-sdd-contracts`) so the delta specs sync into `openspec/specs/`
