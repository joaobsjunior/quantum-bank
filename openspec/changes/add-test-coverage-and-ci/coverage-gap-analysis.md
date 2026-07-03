# Coverage Gap Analysis (tasks 1.2 / 2.3)

Static analysis of production code vs. existing tests, produced without a local
toolchain (no JDK/Gradle/Flutter/Docker available). It maps exactly which lines
and branches are likely uncovered so the test-writing phase (tasks 1.3 / 2.4) is
turnkey once a toolchain can measure and verify 100%.

> These gaps are inferred from source reading, not from a Kover/lcov run. Treat
> as a precise plan to verify, not a measurement.

## Backend (Spring Boot Kotlin — Kover, 100% line, excludes only `QuantumBankApplication`)

Already ~100% (no action): `BankingRepositories`, `PixService`, `PixController`,
`StatementController`, `OtkController`, `CsrController`, `JwtSubject`,
`SecurityProperties`, `BootstrapConfiguration`, `BootstrapErrorCodes`,
`BankingModels`.

Note: `BackendMtlsX509Test` reads source files as text and does NOT contribute
bytecode coverage of `SecurityConfig`.

### Tests to write (priority order)

**P0 — largest uncovered mass: `OtkService.submitCsr` + `BootstrapAuditEvents`**
- `OtkServiceTest.submitCsr_happyPath_consumesOtkAndReturnsSignedCertificate` — real OTK + valid CSR + stub `PkiAdapter`; covers `Consumed` branch, `csrAcceptedForPkiHandoff`, `pkiAdapter.sign` success, `otkConsumed`.
- `OtkServiceTest.submitCsr_rejectsPrivateKeyMaterial` — CSR with `BEGIN PRIVATE KEY` → 400 `private_key_rejected` + `csrRejectedPrivateKeyMaterial`.
- `OtkServiceTest.submitCsr_policyMismatch_profileOrEnvironment` — covers `validateCsrPolicy` catch.
- `OtkServiceTest.submitCsr_subjectMismatch` — covers `validateSubject` throw + `CsrValidator.validateSubject`.
- `OtkServiceTest.submitCsr_otkNotFound / _expired / _replayed / _subjectMismatch / _deviceMismatch / _certificateProfileMismatch` (6 cases) — each forces the matching `OtkConsumeOutcome` and asserts status/errorCode + audit event.
- `OtkServiceTest.submitCsr_pkiHandoffFailure` — stub throwing `PkiHandoffException` → 502 `pki_handoff_failed` + `pkiHandoffFailed`.
- `BootstrapAuditEventsTest.emitsRevokedEvent` — covers `otkRevoked` (not reachable via the above).

**P1 — PKI/security error branches**
- `ScriptPkiAdapterTest.returnsSingleElementChainWhenIssuingCertificateAbsent` — `else` chain-of-1.
- `ScriptPkiAdapterTest.throwsPkiHandoffWhenScriptExitsNonZero` — script `exit 1`.
- `ScriptPkiAdapterTest.throwsPkiHandoffOnTimeout` — script sleeps past timeout (may need configurable timeout).
- `SecurityConfigTest.x509UserDetailsServiceMapsClientCertToMtlsRole` — exercise the x509 lambda.
- `SecurityConfigTest.audienceValidator_acceptsAndRejects` — test the real `jwtDecoder()`/`audienceValidator` (not the `@Primary` test decoder).

**P2 — profile-not-found, invalid request, correlationId header branch**
- `ProfileApiTest.getProfile_returns404WhenSubjectHasNoFixture` and PUT variant — covers `profileNotFound()` + `?: throw`.
- `BootstrapProblemDetailsTest.invalidRequestBodyReturns400RequestInvalid` — invalid JSON / blank `@NotBlank` → `invalidRequest` handler.
- `CorrelationIdTest.echoesProvidedCorrelationIdHeader` — send `X-Correlation-Id` to error endpoints; covers the "header present" branch in `BankingProblemDetails`, `BootstrapProblemDetails`, and both security handlers.

**P3 — residual validators**
- `BootstrapCsrValidationTest.validateSubject_acceptsMatchingAndRejectsMismatch`.
- `BootstrapCsrValidationTest.parse_wrapsUnexpectedExceptionAsCsrInvalid` — corrupt PEM → generic catch.
- `OtkRepositoryTest.consumeOnce_revokedRecordMarksReplayed` — REVOKED record branch.

## Mobile (Flutter/Dart — lcov, 100% line)

Already ~covered: `gateway_api.dart`, `csr_service.dart`.

Biggest holes: the real network layer (`banking_client.dart`,
`bootstrap_client.dart`, `keycloak_auth_client.dart`) only runs in the SKIPPED
`live_gateway_runtime_test.dart` (needs `QUANTUM_BANK_LIVE_TESTS=true` + Docker);
`enrollment_orchestrator.dart` has no real test; `app_state.dart` error branches;
`RuntimeConfig.fromEnvironment()` and `main()` only verified as source text.

### Tests to write (priority order)

**P1 — biggest gain**
- `enrollment_orchestrator_test.dart` — error mapping: fake `BootstrapGateway` throwing `BootstrapProblem('otk_expired'|'otk_replayed'|'csr_invalid'|'private_key_rejected'|'certificate_profile_mismatch'|unknown)` → assert `otkExpired`/`otkReplayed`/`csrRejected` + default (124-131); happy path returns `ReadyCertState` with injected keypair/csr fakes (81-122, `encodePrivateKeyPem`).
- `bootstrap_client_test.dart` — inject fake `HttpClient`: 200 (parse OTK, `certificateChain` fallback to `certificate`), >=400 → `BootstrapProblem`; both `_buildHttpClient` branches.
- `keycloak_auth_client_test.dart` — fake `HttpClient`: success (extract `sub`), 401 → `AuthException`, missing `access_token`, token without `.` → `invalid_access_token`, `preferred_username`/`azp` fallback, `expires_in` default 300, `client_secret` present adds to form.
- `app_state_test.dart` — error branches: `authenticate` throws (40-43); `markCertificateReady` no session (51-56); enrollment non-ready (71-73); enrollment throws → `csrRejected` (74-76).
- `live_gateway_banking_api_test.dart` (extend) — `_callGateway` null/expired session → `StateError` (106-108); `_double` string amount (141); error response missing fields → `??` fallbacks (115-122).

**P2 — low effort residual branches**
- `banking_client_test.dart` — `_httpClientFor` throws for missing/expired cert (144-147).
- `runtime_config_test.dart` — `RuntimeConfig.fromEnvironment()` defaults (30-85).
- `cert_state_test.dart` (extend) — default getters null + `isReadyAt == false`.
- `auth_client_test.dart` — `isValidAt == false`; `AuthException.toString()`.
- `secure_context_factory_test.dart` (extend) — success path with valid mTLS fixtures (19-23).

**P3 — widgets / transient branches**
- `app_gate_test.dart` (extend) — render `lastError`; "Autenticando..."/"Ativando..." disabled buttons; NavigationBar switch to Extrato/Perfil (187).
- `statement_screen_test.dart` (extend) — `pump()` without settle to hit `LinearProgressIndicator` (20-21).
- `pix_screen_test.dart` (extend) — in-flight "Enviando" disabled button.
- `profile_screen_test.dart` (extend) — initial `CircularProgressIndicator`; `save()` null-profile guard (may need refactor).

**Hardest for true 100%**: `main()` (main.dart:16-48) needs `main` invoked with
`rootBundle`/`TestDefaultBinaryMessenger` mocking the CA asset. The skipped
`live_gateway_runtime_test.dart` should not be counted as deterministic coverage.
