## 1. Create the backend-client repository and submodule

- [ ] 1.1 Create a new GitHub repository `quantum-bank-backend-client` (Spring Boot Kotlin 4.0.6, JDK 17) with README, `.gitignore`, and license consistent with the other layer repos
- [x] 1.2 Scaffold the Spring Boot app (Gradle Kotlin DSL): web + Thymeleaf, OAuth2 client, RestClient with mTLS support; commit the full Gradle wrapper (jar + properties) so Docker builds are hermetic
- [x] 1.3 Add a multi-stage `Dockerfile`: a build/test stage on a Gradle + JDK 17 image that runs `gradle check` (compile + tests + 100% Kover gate), and a slim JRE 17 runtime stage that runs the packaged jar
- [ ] 1.4 Add the submodule to the superproject at path `backend-client` with `branch = main` in `.gitmodules`
- [x] 1.5 Update superproject `README.md` and `AGENTS.md` layout tables to list `backend-client`

## 2. External-integration client (external-service-integration)

- [x] 2.1 Configure the OAuth2 client-credentials client `quantum-bank-backend-client` and a token provider; fail closed when no valid token is available
- [x] 2.2 Configure the outbound HTTP client (RestClient) to target the gateway origin only; reject/validate any base URL that resolves to a backend service (fail on direct-to-backend config)
- [x] 2.3 Configure mTLS using the PKI-issued `quantum-bank-service-client-v1` certificate and trust anchors; fail closed on missing/invalid certificate material (no permissive TLS)
- [x] 2.4 Implement the Pix transfer call with app-selected success/error scenario through `POST /pix/transfers`
- [x] 2.5 Implement the account statement call through `GET /statements`
- [x] 2.6 Implement the profile / registration-data call through `GET /profile`
- [x] 2.7 Propagate `correlationId` on every outbound request and parse RFC 9457 problem-details error responses into typed errors

## 3. Web console (backend-client-console)

- [x] 3.1 Build the console home page (Thymeleaf) listing the Pix, statement, and profile flows (parity with the mobile journeys)
- [x] 3.2 Add the Pix form with a success/error scenario selector
- [x] 3.3 Wire each console action to the external-integration client (no direct-to-backend or bypass control exposed)
- [x] 3.4 Display, per run, the request summary, gateway response, `correlationId`, and any problem-details error body

## 4. Local runtime and identity provisioning (deployment-infrastructure)

- [x] 4.1 Register the confidential `quantum-bank-backend-client` client (client-credentials, audience `quantum-bank-api`, required scopes) in the Keycloak realm import
- [x] 4.2 Issue the `quantum-bank-service-client-v1` service client certificate/keystore + truststore by extending the PKI runtime-cert bootstrap script
- [x] 4.3 Add the `backend-client` service to `infrastructure/compose.yaml` with a `build:` context, wired to enter through the gateway with its token and mTLS certificate, mounting its keystore/truststore and trust anchors
- [x] 4.4 Provide the `backend-client` runtime configuration via container env (gateway base URL, issuer/JWKS/audience, keystore/truststore paths, correlation-id propagation) without hard-coded secrets
- [x] 4.5 Confirm the gateway accepts the service certificate over mTLS (chains to the `root-ca` the banking listener already trusts) — no gateway config change expected

## 5. Validation and landing (all via Docker)

- [x] 5.1 Build the image with `docker build` and confirm the build stage runs the tests and fails below 100% coverage (no host toolchain used)
- [x] 5.2 Bring up the stack with `docker compose up` and exercise each console flow end to end (Pix success + error, statement, profile) through the gateway
- [x] 5.3 Confirm fail-closed behavior: missing token and missing/invalid certificate both block the call with no permissive fallback
- [x] 5.4 `openspec validate add-backend-client-service --strict` passes
- [ ] 5.5 Commit the submodule implementation, bump the superproject submodule pointer, and commit the OpenSpec change
- [ ] 5.6 Archive the change (`openspec archive add-backend-client-service`) so the delta specs sync into `openspec/specs/`
