# Graph Report - .  (2026-07-05)

## Corpus Check
- Corpus is ~49,217 words - fits in a single context window. You may not need a graph.

## Summary
- 1118 nodes · 1545 edges · 90 communities (68 shown, 22 thin omitted)
- Extraction: 94% EXTRACTED · 6% INFERRED · 0% AMBIGUOUS · INFERRED: 95 edges (avg confidence: 0.82)
- Token cost: 288,208 input · 0 output

## Community Hubs (Navigation)
- [[_COMMUNITY_Project Overview & Specs|Project Overview & Specs]]
- [[_COMMUNITY_Mobile App Core (Flutter)|Mobile App Core (Flutter)]]
- [[_COMMUNITY_Backend Banking Models|Backend Banking Models]]
- [[_COMMUNITY_Bootstrap Audit Events|Bootstrap Audit Events]]
- [[_COMMUNITY_PKI Adapter (Backend)|PKI Adapter (Backend)]]
- [[_COMMUNITY_Secure Gateway Concepts|Secure Gateway Concepts]]
- [[_COMMUNITY_Mobile App State|Mobile App State]]
- [[_COMMUNITY_Mobile Banking Client|Mobile Banking Client]]
- [[_COMMUNITY_Backend Simulation Capability|Backend Simulation Capability]]
- [[_COMMUNITY_OTKCSR Controllers (Backend)|OTK/CSR Controllers (Backend)]]
- [[_COMMUNITY_Mobile Cert State|Mobile Cert State]]
- [[_COMMUNITY_iOS Runner (Flutter)|iOS Runner (Flutter)]]
- [[_COMMUNITY_Backend Security Config|Backend Security Config]]
- [[_COMMUNITY_Mobile Gateway API DTOs|Mobile Gateway API DTOs]]
- [[_COMMUNITY_Backend JWT Security Tests|Backend JWT Security Tests]]
- [[_COMMUNITY_CSREnrollment Tests (Mobile)|CSR/Enrollment Tests (Mobile)]]
- [[_COMMUNITY_Mobile TLS Context & Errors|Mobile TLS Context & Errors]]
- [[_COMMUNITY_Backend Problem Details|Backend Problem Details]]
- [[_COMMUNITY_Mobile App Gate Tests|Mobile App Gate Tests]]
- [[_COMMUNITY_Backend Layer Docs|Backend Layer Docs]]
- [[_COMMUNITY_Mobile Screen Tests|Mobile Screen Tests]]
- [[_COMMUNITY_Mobile Enrollment Orchestrator|Mobile Enrollment Orchestrator]]
- [[_COMMUNITY_Mobile Pix Screen|Mobile Pix Screen]]
- [[_COMMUNITY_Live Gateway API Tests|Live Gateway API Tests]]
- [[_COMMUNITY_Coverage Gap Analysis|Coverage Gap Analysis]]
- [[_COMMUNITY_KrakenD Banking Listener|KrakenD Banking Listener]]
- [[_COMMUNITY_Backend Pix API Tests|Backend Pix API Tests]]
- [[_COMMUNITY_StatementProfile API Tests|Statement/Profile API Tests]]
- [[_COMMUNITY_Mobile Bootstrap Client|Mobile Bootstrap Client]]
- [[_COMMUNITY_Mobile Auth Error Tests|Mobile Auth Error Tests]]
- [[_COMMUNITY_Live Banking API Test Doubles|Live Banking API Test Doubles]]
- [[_COMMUNITY_KrakenD Root Config|KrakenD Root Config]]
- [[_COMMUNITY_Bootstrap Problem Details|Bootstrap Problem Details]]
- [[_COMMUNITY_Profile Not Found Tests|Profile Not Found Tests]]
- [[_COMMUNITY_Mobile Auth Client|Mobile Auth Client]]
- [[_COMMUNITY_Mobile CSR Service|Mobile CSR Service]]
- [[_COMMUNITY_KrakenD Bootstrap Listener|KrakenD Bootstrap Listener]]
- [[_COMMUNITY_Keycloak Auth Client (Mobile)|Keycloak Auth Client (Mobile)]]
- [[_COMMUNITY_Continuous Integration Spec|Continuous Integration Spec]]
- [[_COMMUNITY_Bootstrap Problem Details Tests|Bootstrap Problem Details Tests]]
- [[_COMMUNITY_Mobile Real Flow Tests|Mobile Real Flow Tests]]
- [[_COMMUNITY_Onboarding & PKI Spec|Onboarding & PKI Spec]]
- [[_COMMUNITY_Backend CSR Validator|Backend CSR Validator]]
- [[_COMMUNITY_JWT Subject Tests|JWT Subject Tests]]
- [[_COMMUNITY_Mobile Keypair Service|Mobile Keypair Service]]
- [[_COMMUNITY_Access Denied Handler|Access Denied Handler]]
- [[_COMMUNITY_Auth Entry Point (Backend)|Auth Entry Point (Backend)]]
- [[_COMMUNITY_Gateway Banking API Impls|Gateway Banking API Impls]]
- [[_COMMUNITY_Terraform Multi-Cloud Paths|Terraform Multi-Cloud Paths]]
- [[_COMMUNITY_Keycloak Auth Client Tests|Keycloak Auth Client Tests]]
- [[_COMMUNITY_Runtime Config Tests|Runtime Config Tests]]
- [[_COMMUNITY_Project Vision (project.md)|Project Vision (project.md)]]
- [[_COMMUNITY_Deployment Infrastructure Spec|Deployment Infrastructure Spec]]
- [[_COMMUNITY_Mobile Banking Journeys Spec|Mobile Banking Journeys Spec]]
- [[_COMMUNITY_Backend Statement Controller|Backend Statement Controller]]
- [[_COMMUNITY_OTK Validation Tests|OTK Validation Tests]]
- [[_COMMUNITY_Local E2E Smoke Test|Local E2E Smoke Test]]
- [[_COMMUNITY_Bootstrap Client Tests|Bootstrap Client Tests]]
- [[_COMMUNITY_Enrollment Test Doubles|Enrollment Test Doubles]]
- [[_COMMUNITY_Mobile Domain Models|Mobile Domain Models]]
- [[_COMMUNITY_Flutter LLDB Helper|Flutter LLDB Helper]]
- [[_COMMUNITY_Backend App Entrypoint|Backend App Entrypoint]]
- [[_COMMUNITY_CSR Validation Tests|CSR Validation Tests]]
- [[_COMMUNITY_Docker Compose Services|Docker Compose Services]]
- [[_COMMUNITY_Local E2E Config Verify|Local E2E Config Verify]]
- [[_COMMUNITY_Bootstrap Configuration|Bootstrap Configuration]]
- [[_COMMUNITY_Backend mTLS X509 Tests|Backend mTLS X509 Tests]]
- [[_COMMUNITY_PKI Runtime Cert Issuance|PKI Runtime Cert Issuance]]
- [[_COMMUNITY_PKI Negative mTLS Tests|PKI Negative mTLS Tests]]
- [[_COMMUNITY_Gateway CI Validate|Gateway CI Validate]]
- [[_COMMUNITY_JWT Subject Resolver|JWT Subject Resolver]]
- [[_COMMUNITY_Terraform Config Verify|Terraform Config Verify]]
- [[_COMMUNITY_Mobile Coverage Check|Mobile Coverage Check]]
- [[_COMMUNITY_Bootstrap Scopes Verify|Bootstrap Scopes Verify]]
- [[_COMMUNITY_Bootstrap Error Codes|Bootstrap Error Codes]]
- [[_COMMUNITY_Infra CI Validate|Infra CI Validate]]
- [[_COMMUNITY_Keycloak Config Verify|Keycloak Config Verify]]
- [[_COMMUNITY_Flutter Export Env|Flutter Export Env]]
- [[_COMMUNITY_Gateway-Only Guard|Gateway-Only Guard]]
- [[_COMMUNITY_Local CA Bootstrap|Local CA Bootstrap]]
- [[_COMMUNITY_PKI CI Validate|PKI CI Validate]]
- [[_COMMUNITY_Cert Revocation Script|Cert Revocation Script]]
- [[_COMMUNITY_CSR Signing Script|CSR Signing Script]]
- [[_COMMUNITY_Trust Anchors Verify|Trust Anchors Verify]]
- [[_COMMUNITY_Customer Profile Type|Customer Profile Type]]
- [[_COMMUNITY_List Type|List Type]]
- [[_COMMUNITY_Map Type|Map Type]]
- [[_COMMUNITY_String Type|String Type]]

## God Nodes (most connected - your core abstractions)
1. `OtkRecord` - 19 edges
2. `BootstrapAuditEvents` - 14 edges
3. `OtkServiceTest` - 14 edges
4. `OtkConsumeOutcome` - 11 edges
5. `SucceedingPkiAdapter` - 11 edges
6. `BackendJwtSecurityTest` - 11 edges
7. `CertState` - 11 edges
8. `Onboarding PKI Spec` - 10 edges
9. `BankingRepositoryTest` - 9 edges
10. `Quantum Bank Project` - 9 edges

## Surprising Connections (you probably didn't know these)
- `Local Runtime Service Topology` --conceptually_related_to--> `Compose Service: backend`  [INFERRED]
  openspec/specs/deployment-infrastructure/spec.md → infrastructure/compose.yaml
- `secure_context_factory.dart fail-closed mTLS` --semantically_similar_to--> `Backend mTLS SSL config (PKCS12, client-auth NEED)`  [INFERRED] [semantically similar]
  mobile-app/README.md → backend/src/main/resources/application.yml
- `Secure app->gateway->backend end-to-end flow` --semantically_similar_to--> `Gateway-only app-facing traffic rule`  [INFERRED] [semantically similar]
  AGENTS.md → README.md
- `Gateway-only origin config (no direct backend)` --semantically_similar_to--> `Gateway-only app-facing traffic rule`  [INFERRED] [semantically similar]
  mobile-app/README.md → README.md
- `Quantum Bank Backend Layer` --conceptually_related_to--> `Spring Boot Kotlin Backend`  [INFERRED]
  backend/README.md → openspec/project.md

## Import Cycles
- None detected.

## Hyperedges (group relationships)
- **Secure Mobile-to-Backend Banking Path** — openspec_changes_archive_2026_06_17_add_mobile_banking_journeys_proposal_flutter_mobile_app, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_proposal_krakend_gateway, openspec_changes_archive_2026_06_17_add_backend_simulation_api_proposal_spring_boot_kotlin_backend [EXTRACTED 0.85]
- **Protected Access Security Controls** — openspec_changes_archive_2026_06_17_add_secure_gateway_communication_proposal_oauth2_bearer_validation, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_proposal_mutual_tls, openspec_changes_archive_2026_06_17_add_onboarding_pki_proposal_csr_driven_issuance, openspec_changes_archive_2026_06_17_add_onboarding_pki_proposal_pki_lifecycle_openxpki [INFERRED 0.75]
- **Local End-to-End Validation Stack** — openspec_changes_archive_2026_06_17_add_deployment_infrastructure_proposal_docker_compose_e2e, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_proposal_oauth2_bearer_validation, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_proposal_mutual_tls [EXTRACTED 0.85]
- **Secure gateway communication security pillars** — openspec_changes_archive_2026_06_17_add_secure_gateway_communication_specs_secure_gateway_communication_spec_gateway_only_mobile_communication, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_specs_secure_gateway_communication_spec_oauth2_protects_gateway_and_backend, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_specs_secure_gateway_communication_spec_mutual_tls_enforced, openspec_changes_archive_2026_06_17_add_secure_gateway_communication_specs_secure_gateway_communication_spec_tls_verification_fails_closed [INFERRED 0.85]
- **Test coverage and CI pipeline components** — openspec_changes_archive_2026_07_03_add_test_coverage_and_ci_design_kover_gradle_plugin, openspec_changes_archive_2026_07_03_add_test_coverage_and_ci_design_flutter_lcov_threshold, openspec_changes_archive_2026_07_03_add_test_coverage_and_ci_design_github_actions_pipeline, openspec_changes_archive_2026_07_03_add_test_coverage_and_ci_design_required_status_checks_branch_protection [INFERRED 0.85]
- **SDD contract migration target specs** — openspec_changes_archive_2026_07_05_migrate_sdd_contracts_specs_secure_gateway_communication_spec_secure_gateway_communication_migrated, openspec_changes_archive_2026_07_05_migrate_sdd_contracts_specs_onboarding_pki_spec_onboarding_pki, openspec_changes_archive_2026_07_05_migrate_sdd_contracts_specs_backend_simulation_api_spec_backend_simulation_api, openspec_changes_archive_2026_07_05_migrate_sdd_contracts_specs_mobile_banking_journeys_spec_mobile_banking_journeys, openspec_changes_archive_2026_07_05_migrate_sdd_contracts_specs_deployment_infrastructure_spec_deployment_infrastructure [INFERRED 0.85]
- **Local Compose Runtime Topology** — infrastructure_compose_keycloak_service, infrastructure_compose_backend_service, infrastructure_compose_gateway_bootstrap_service, infrastructure_compose_gateway_banking_service, infrastructure_compose_smoke_tests_service [INFERRED 0.85]
- **Terraform Multi-Cloud Deployment Paths** — infrastructure_terraform_aws_readme_aws_ecs_fargate_path, infrastructure_terraform_azure_readme_azure_container_apps_path, infrastructure_terraform_gcp_readme_gcp_cloud_run_path, infrastructure_terraform_readme_runtime_conventions_module [INFERRED 0.85]
- **Secure End-to-End Layer Specs** — openspec_specs_secure_gateway_communication_spec_secure_gateway_communication, openspec_specs_onboarding_pki_spec_onboarding_pki, openspec_specs_backend_simulation_api_spec_backend_simulation_api, openspec_specs_mobile_banking_journeys_spec_mobile_banking_journeys, openspec_specs_deployment_infrastructure_spec_deployment_infrastructure [INFERRED 0.75]
- **Protected banking routes on 8443 (OAuth2 + mTLS)** — api_gateway_openapi_quantum_bank_v1_route_pix_transfers, api_gateway_openapi_quantum_bank_v1_route_statements, api_gateway_openapi_quantum_bank_v1_route_profile, api_gateway_openapi_quantum_bank_v1_banking_listener [INFERRED 0.85]
- **OTK/CSR onboarding bootstrap flow** — api_gateway_openapi_quantum_bank_v1_route_auth_otk, api_gateway_openapi_quantum_bank_v1_route_auth_csr, pki_adapter_readme_sign_csr_command, mobile_app_readme_certificate_ready_state [INFERRED 0.75]
- **Per-layer CI validation gates aggregated by superproject** — pki_github_workflows_ci_pki_ci, mobile_app_github_workflows_ci_mobile_ci, api_gateway_github_workflows_ci_gateway_ci, github_workflows_ci_ci_gate_job [INFERRED 0.85]

## Communities (90 total, 22 thin omitted)

### Community 0 - "Project Overview & Specs"
Cohesion: 0.05
Nodes (51): Quantum Bank agent instructions (AGENTS.md), Secure app->gateway->backend end-to-end flow, Spec: onboarding-pki, Spec: pix-simulation (no real Pix rails), Spec: secure-gateway-communication, Stack constraints (Flutter 3.41, Spring Boot Kotlin 4.0.6, KrakenD), API Gateway CI workflow (KrakenD config validate), KrakenD banking listener (8443, OAuth2 + MutualTLS) (+43 more)

### Community 1 - "Mobile App Core (Flutter)"
Cohesion: 0.05
Nodes (45): app/app_state.dart, core/api/banking_client.dart, core/api/live_gateway_banking_api.dart, features/auth/keycloak_auth_client.dart, features/bootstrap/bootstrap_client.dart, features/pix/pix_screen.dart, features/profile/profile_screen.dart, features/statements/statement_screen.dart (+37 more)

### Community 2 - "Backend Banking Models"
Cohesion: 0.07
Nodes (22): CustomerProfile, NewStatementEntry, PixScenario, PixTransferAttempt, PixTransferStatus, ProfileUpdate, StatementEntry, StatementEntryType (+14 more)

### Community 3 - "Bootstrap Audit Events"
Cohesion: 0.12
Nodes (17): BootstrapAuditEvents, String, CertificateProfileMismatch, Consumed, DeviceMismatch, Expired, InMemoryOtkRepository, String (+9 more)

### Community 4 - "PKI Adapter (Backend)"
Cohesion: 0.13
Nodes (12): PkiAdapter, PkiHandoffException, PkiSignRequest, PkiSignResult, ScriptPkiAdapter, PkiProperties, SecurityProperties, FailingPkiAdapter (+4 more)

### Community 5 - "Secure Gateway Concepts"
Cohesion: 0.06
Nodes (37): Gateway-only mobile communication, KrakenD API gateway, Mutual TLS enforced for secure channels, OAuth2 protects gateway and backend, Project trust chain, Spring Boot backend resource server, TLS verification fails closed, Add secure gateway communication tasks (+29 more)

### Community 6 - "Mobile App State"
Cohesion: 0.06
Nodes (33): bool get, ChangeNotifier, core/config/runtime_config.dart, features/bootstrap/enrollment_orchestrator.dart, authenticate, authenticated, authenticating, _authenticator (+25 more)

### Community 7 - "Mobile Banking Client"
Cohesion: 0.07
Nodes (30): banking_client.dart, ../../features/auth/auth_client.dart, gateway_api.dart, BankingClient, BankingGatewayClient, createPixTransfer, gatewayBaseUrl, getProfile (+22 more)

### Community 8 - "Backend Simulation Capability"
Cohesion: 0.08
Nodes (32): Backend Simulation API (Capability), Banking Simulation APIs, In-Memory Persistence (H2), Spring Boot Kotlin Backend, Backend Simulation API Requirements Spec, Backend Simulation API Tasks, Deployment Infrastructure (Capability), Docker Compose End-to-End Setup (+24 more)

### Community 9 - "OTK/CSR Controllers (Backend)"
Cohesion: 0.12
Nodes (17): CsrController, CsrSubmitHttpRequest, HttpServletRequest, Jwt, HttpServletRequest, Jwt, OtkController, OtkIssueHttpRequest (+9 more)

### Community 10 - "Mobile Cert State"
Cohesion: 0.09
Nodes (26): DateTime? get, appInstanceId, certificateChainBytes, certificateProfile, CertState, csrRejected, CsrRejectedCertState, deviceId (+18 more)

### Community 11 - "iOS Runner (Flutter)"
Cohesion: 0.09
Nodes (17): Bool, Flutter, FlutterAppDelegate, FlutterImplicitEngineBridge, FlutterImplicitEngineDelegate, FlutterSceneDelegate, AppDelegate, Any (+9 more)

### Community 12 - "Backend Security Config"
Cohesion: 0.10
Nodes (13): AuthenticationUserDetailsService, Jwt, JwtDecoder, String, SecurityConfig, Jwt, List, String (+5 more)

### Community 13 - "Mobile Gateway API DTOs"
Cohesion: 0.08
Nodes (23): address, amount, copyWith, correlationId, description, detail, documentNumber, email (+15 more)

### Community 14 - "Backend JWT Security Tests"
Cohesion: 0.15
Nodes (9): BackendJwtSecurityTest, Jwt, JwtDecoder, List, MockMvc, String, Route, TestJwtConfiguration (+1 more)

### Community 15 - "CSR/Enrollment Tests (Mobile)"
Cohesion: 0.12
Nodes (18): dart:convert, main, encodePrivateKeyPem, enroll, generatePem, generateRsaKeyPair, issueOtk, main (+10 more)

### Community 16 - "Mobile TLS Context & Errors"
Cohesion: 0.10
Nodes (20): dart:io, Exception, BankingClientCertificateException, BankingHttpProblemException, GatewayProblemException, build, message, SecureContextFactory (+12 more)

### Community 17 - "Backend Problem Details"
Cohesion: 0.16
Nodes (15): BankingProblemDetails, BankingProblemException, correlationId(), Any, HttpServletRequest, Map, ResponseEntity, String (+7 more)

### Community 18 - "Mobile App Gate Tests"
Cohesion: 0.12
Nodes (15): authenticate, enroll, main, main, main, authenticate, enroll, gateState (+7 more)

### Community 19 - "Backend Layer Docs"
Cohesion: 0.15
Nodes (17): Gradle init.d Init Scripts Readme, Quantum Bank Backend Layer, Backend Bootstrap Package (OTK/CSR/PKI Adapter), Migrate SDD Contracts into OpenSpec, OpenSpec Config (spec-driven schema), Backend Simulation API Spec, Gateway OpenAPI Contract (quantum-bank-v1.yaml), Problem Details (application/problem+json) (+9 more)

### Community 20 - "Mobile Screen Tests"
Cohesion: 0.16
Nodes (11): main, main, main, main, main, package:flutter/material.dart, package:flutter_test/flutter_test.dart, package:quantum_bank_mobile/core/api/gateway_api.dart (+3 more)

### Community 21 - "Mobile Enrollment Orchestrator"
Cohesion: 0.13
Nodes (14): ../../core/tls/cert_state.dart, csr_service.dart, keypair_service.dart, certificateChainBytes, CertificateEnrollmentResult, _csrService, enroll, errorCode (+6 more)

### Community 22 - "Mobile Pix Screen"
Cohesion: 0.14
Nodes (13): amountController, api, build, createState, descriptionController, dispose, loading, problem (+5 more)

### Community 23 - "Live Gateway API Tests"
Cohesion: 0.14
Nodes (13): apiWith, createPixTransfer, getProfile, getStatements, main, pix, problem, _reply (+5 more)

### Community 24 - "Coverage Gap Analysis"
Cohesion: 0.15
Nodes (14): Coverage gap analysis, enrollment_orchestrator.dart error mapping, OtkService.submitCsr uncovered mass, 100% line coverage fail-closed gate, Docker Compose opt-in e2e job, Five layer submodules (backend, mobile-app, api-gateway, infrastructure, pki), Flutter lcov coverage threshold, GitHub Actions CI pipeline (+6 more)

### Community 25 - "KrakenD Banking Listener"
Cohesion: 0.15
Nodes (12): client_tls, ca_certs, client_certs, endpoints, name, port, $schema, tls (+4 more)

### Community 26 - "Backend Pix API Tests"
Cohesion: 0.17
Nodes (7): Jwt, JwtDecoder, List, MockMvc, String, PixApiTest, TestJwtConfiguration

### Community 27 - "Statement/Profile API Tests"
Cohesion: 0.17
Nodes (7): Jwt, JwtDecoder, List, MockMvc, String, StatementProfileApiTest, TestJwtConfiguration

### Community 28 - "Mobile Bootstrap Client"
Cohesion: 0.15
Nodes (12): enrollment_orchestrator.dart, HttpClient, baseUrl, BootstrapClient, _buildHttpClient, _httpClient, issueOtk, _postJson (+4 more)

### Community 29 - "Mobile Auth Error Tests"
Cohesion: 0.17
Nodes (12): Authenticator, KeycloakAuthClient, _TestAuthenticator, authenticate, enroll, main, OkAuthenticator, _state (+4 more)

### Community 30 - "Live Banking API Test Doubles"
Cohesion: 0.15
Nodes (12): authSession, createPixTransfer, getProfile, getStatements, lastBearerToken, lastCertState, lastPixPayload, lastProfilePayload (+4 more)

### Community 31 - "KrakenD Root Config"
Cohesion: 0.17
Nodes (11): client_tls, ca_certs, client_certs, endpoints, name, _phase3_mtls_note, port, $schema (+3 more)

### Community 32 - "Bootstrap Problem Details"
Cohesion: 0.38
Nodes (8): bootstrapCorrelationId(), BootstrapProblemDetails, Any, HttpServletRequest, HttpStatus, Map, ResponseEntity, String

### Community 33 - "Profile Not Found Tests"
Cohesion: 0.18
Nodes (7): Jwt, JwtDecoder, List, MockMvc, String, ProfileNotFoundTest, TestJwtConfiguration

### Community 34 - "Mobile Auth Client"
Cohesion: 0.17
Nodes (11): DateTime, accessToken, authenticate, expiresAt, isValidAt, message, scopes, statusCode (+3 more)

### Community 35 - "Mobile CSR Service"
Cohesion: 0.17
Nodes (11): appInstanceId, certificateProfile, CsrInput, CsrService, deviceId, distinguishedName, environment, generatePem (+3 more)

### Community 36 - "KrakenD Bootstrap Listener"
Cohesion: 0.18
Nodes (10): client_tls, ca_certs, client_certs, endpoints, name, port, $schema, tls (+2 more)

### Community 37 - "Keycloak Auth Client (Mobile)"
Cohesion: 0.18
Nodes (10): auth_client.dart, authenticate, _authenticateOnce, clientId, clientSecret, _decodeJwtPayload, _httpClient, password (+2 more)

### Community 38 - "Continuous Integration Spec"
Cohesion: 0.22
Nodes (11): Backend CI build-test-coverage Job, gradlew check (Kover 100% Coverage), Continuous Integration Spec, Optional End-to-End Validation Job, Fail-Closed Merge Gating, Per-Layer CI Build-Test-Coverage Job, Superproject CI Orchestrator, Flutter lcov Coverage Gate (+3 more)

### Community 39 - "Bootstrap Problem Details Tests"
Cohesion: 0.20
Nodes (6): BootstrapProblemDetailsTest, Jwt, JwtDecoder, MockMvc, String, TestJwtConfiguration

### Community 40 - "Mobile Real Flow Tests"
Cohesion: 0.18
Nodes (10): appInstanceId, authenticate, bearerToken, called, certificateProfile, deviceId, enroll, environment (+2 more)

### Community 41 - "Onboarding & PKI Spec"
Cohesion: 0.18
Nodes (11): OpenXPKI, PKI Layer, Bootstrap Audit Events, Local CA Adapter (OpenXPKI Swap-In), Mobile Key Material and Certificate-Ready State, Onboarding PKI Spec, One-Time Key (OTK), OTK State Machine (+3 more)

### Community 42 - "Backend CSR Validator"
Cohesion: 0.44
Nodes (4): CsrValidationException, CsrValidator, String, PKCS10CertificationRequest

### Community 43 - "JWT Subject Tests"
Cohesion: 0.27
Nodes (5): JwtSubjectTest, Any, Jwt, Map, String

### Community 44 - "Mobile Keypair Service"
Cohesion: 0.20
Nodes (9): dart:math, dart:typed_data, encodePrivateKeyPem, generateRsaKeyPair, KeypairService, _secureRandom, FakeKeypairService, package:basic_utils/basic_utils.dart (+1 more)

### Community 45 - "Access Denied Handler"
Cohesion: 0.28
Nodes (6): AccessDeniedException, AccessDeniedHandler, HttpServletRequest, HttpServletResponse, String, ProblemDetailsAccessDeniedHandler

### Community 46 - "Auth Entry Point (Backend)"
Cohesion: 0.28
Nodes (6): AuthenticationEntryPoint, AuthenticationException, HttpServletRequest, HttpServletResponse, String, ProblemDetailsAuthenticationEntryPoint

### Community 47 - "Gateway Banking API Impls"
Cohesion: 0.22
Nodes (8): core/api/gateway_api.dart, DemoGatewayBankingApi, GatewayBankingApi, LiveGatewayBankingApi, api, build, StatementScreen, StatelessWidget

### Community 48 - "Terraform Multi-Cloud Paths"
Cohesion: 0.31
Nodes (9): ci-validate.sh Script, Infrastructure CI validate Job, Quantum Bank Infrastructure Layer, AWS ECS/Fargate Deployment Path, Azure Container Apps Deployment Path, GCP Cloud Run v2 Deployment Path, runtime-conventions Shared Module, Quantum Bank Terraform Deployment Paths (+1 more)

### Community 49 - "Keycloak Auth Client Tests"
Cohesion: 0.22
Nodes (8): clientFor, jwtWith, main, respondJson, seg, serve, server, SocketException

### Community 50 - "Runtime Config Tests"
Cohesion: 0.22
Nodes (7): main, main, package:quantum_bank_mobile/core/api/banking_client.dart, package:quantum_bank_mobile/core/config/runtime_config.dart, package:quantum_bank_mobile/features/auth/keycloak_auth_client.dart, package:quantum_bank_mobile/features/bootstrap/bootstrap_client.dart, package:quantum_bank_mobile/features/bootstrap/enrollment_orchestrator.dart

### Community 51 - "Project Vision (project.md)"
Cohesion: 0.31
Nodes (9): Backend Independent OAuth2 Validation, Flutter Mobile App, KrakenD API Gateway, Pix Simulation (No Real Rails in v1), Quantum Bank Project, Secure End-to-End Banking Flow, Spring Boot Kotlin Backend, Superproject with Layer Submodules (+1 more)

### Community 52 - "Deployment Infrastructure Spec"
Cohesion: 0.22
Nodes (9): Terraform Multi-Cloud Paths, Deployment Infrastructure Spec, Gateway Listener Ports 8080/8443, Keycloak Local OAuth2 Issuer, quantum-bank-local Realm, Local Runtime Service Topology, Trust Material and Runtime Config, Certificate Profile quantum-bank-mobile-client-v1 (+1 more)

### Community 53 - "Mobile Banking Journeys Spec"
Cohesion: 0.22
Nodes (9): Scenario-Driven Pix Persistence, App-Selected Pix Scenario, Mobile Client Preconditions for Protected Calls, Gateway-Named Origins Only, Mobile Banking Journeys Spec, verify-gateway-only.sh Guard, No Real Pix Settlement Rails in v1, Pix Simulation Spec (+1 more)

### Community 54 - "Backend Statement Controller"
Cohesion: 0.36
Nodes (6): HttpServletRequest, Jwt, StatementController, StatementEntryResponse, StatementResponse, toResponse()

### Community 55 - "OTK Validation Tests"
Cohesion: 0.32
Nodes (3): BootstrapOtkValidationTest, String, Instant

### Community 56 - "Local E2E Smoke Test"
Cohesion: 0.57
Nodes (7): expect_body_contains(), expect_http_status(), expect_tls_failure(), fail(), local-e2e-smoke.sh script, wait_for_banking_ready(), wait_for_token()

### Community 57 - "Bootstrap Client Tests"
Cohesion: 0.25
Nodes (7): clientFor, main, respondJson, serve, server, submitCsr, return

### Community 58 - "Enrollment Test Doubles"
Cohesion: 0.29
Nodes (7): CertificateEnrollment, EnrollmentOrchestrator, _TestCertificateEnrollment, ReturningEnrollment, ThrowingEnrollment, RecordingCertificateEnrollment, _NoopEnrollment

### Community 59 - "Mobile Domain Models"
Cohesion: 0.33
Nodes (6): @immutable, CustomerProfile, GatewayProblem, PixResult, StatementItem, AuthSession

### Community 60 - "Flutter LLDB Helper"
Cohesion: 0.33
Nodes (5): handle_new_rx_page(), __lldb_init_module(), Intercept NOTIFY_DEBUGGER_ABOUT_RX_PAGES and touch the pages., SBDebugger, SBFrame

### Community 61 - "Backend App Entrypoint"
Cohesion: 0.40
Nodes (4): Array, String, main(), QuantumBankApplication

### Community 63 - "Docker Compose Services"
Cohesion: 0.80
Nodes (5): Compose Service: backend, Compose Service: gateway-banking, Compose Service: gateway-bootstrap, Compose Service: keycloak, Compose Service: smoke-tests

### Community 64 - "Local E2E Config Verify"
Cohesion: 0.70
Nodes (4): require_executable(), require_file(), require_string(), verify-local-e2e-config.sh script

### Community 67 - "PKI Runtime Cert Issuance"
Cohesion: 0.83
Nodes (3): issue_cert(), bootstrap-runtime-certs.sh script, write_ext()

### Community 68 - "PKI Negative mTLS Tests"
Cohesion: 0.83
Nodes (3): expect_tls_failure(), require_running_endpoint(), negative-mtls-tests.sh script

## Knowledge Gaps
- **368 isolated node(s):** `$schema`, `version`, `name`, `port`, `keys` (+363 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **22 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `BootstrapProblemException` connect `OTK/CSR Controllers (Backend)` to `Bootstrap Problem Details`?**
  _High betweenness centrality (0.017) - this node is a cross-community bridge._
- **Why does `CertState` connect `Mobile Cert State` to `Mobile Auth Error Tests`, `Mobile App State`, `Live Banking API Test Doubles`?**
  _High betweenness centrality (0.017) - this node is a cross-community bridge._
- **Why does `GatewayBankingApi` connect `Gateway Banking API Impls` to `Mobile App Core (Flutter)`, `Mobile Gateway API DTOs`, `Mobile Pix Screen`?**
  _High betweenness centrality (0.016) - this node is a cross-community bridge._
- **Are the 3 inferred relationships involving `OtkRecord` (e.g. with `.issue()` and `.consumeOnceAllowsOnlyOneWinnerAndMarksReplay()`) actually correct?**
  _`OtkRecord` has 3 INFERRED edges - model-reasoned connections that need verification._
- **What connects `$schema`, `version`, `name` to the rest of the system?**
  _375 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Project Overview & Specs` be split into smaller, more focused modules?**
  _Cohesion score 0.054901960784313725 - nodes in this community are weakly interconnected._
- **Should `Mobile App Core (Flutter)` be split into smaller, more focused modules?**
  _Cohesion score 0.04902867715078631 - nodes in this community are weakly interconnected._