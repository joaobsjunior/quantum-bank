<!-- GSD:project-start source:PROJECT.md -->
## Project

**Quantum Bank**

Quantum Bank is a mobile banking application with three initial user journeys: Pix transfer, account statement, and customer registration data. The app is built in Flutter 3.41 and communicates end to end with a Spring Boot Kotlin backend through a KrakenD API gateway.

The v1 project prioritizes the secure communication foundation first: OAuth2 authentication, OTK-based onboarding, CSR-driven certificate generation, mTLS, and PKI. Even if the first screens are simple, they must exercise the real app -> gateway -> backend flow.

**Core Value:** The app must prove a secure end-to-end banking flow from Flutter through KrakenD to the backend, including Pix success and error simulation.

### Constraints

- **Mobile stack**: Flutter 3.41 - requested app technology.
- **Backend stack**: Spring Boot Kotlin 4.0.6 - requested backend technology.
- **API gateway**: KrakenD - all mobile/backend communication should pass through the gateway.
- **Persistence**: H2 or memory database is acceptable for v1 - supports fast local end-to-end validation.
- **Security**: OAuth2, OTK, CSR, mTLS, and PKI are core v1 concerns - this is a banking app and security architecture is the primary risk.
- **PKI dependency**: Use KrakenD PKI features only if they satisfy the required certificate lifecycle; otherwise use an open source option such as OpenXPKI.
- **Deployment**: Backend and gateway must be dockerized; Terraform paths are required for AWS, GCP, and Azure.
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Technologies
| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Flutter | 3.41 | Mobile app for Pix, statement, and registration data screens | User requested Flutter 3.41, and the official Flutter docs list 3.41 as a live 2026 release. |
| Dart `dart:io` TLS APIs | Dart 3.11.x runtime family | Client TLS, trusted CA configuration, and client certificate material | `HttpClient` accepts a `SecurityContext`; `SecurityContext` supports trusted certificates, certificate chains, and private keys needed for mTLS-capable clients. |
| KrakenD Community Edition | 2.13.5 latest community docs | API Gateway and policy enforcement layer | Official docs identify CE 2.13.5 as current and document JWT validation, mTLS, schema validation, routing, and Docker deployment. |
| Spring Boot Kotlin | 4.0.6 | Backend API services | User requested Spring Boot 4.0.6, and Spring Boot docs list 4.0.6 as the current stable 4.0 line. |
| Spring Security OAuth2 Resource Server | Boot-managed | JWT/OAuth2 protection for backend APIs | Spring Security provides first-class OAuth2 Resource Server support and Spring Boot exposes the corresponding starter. |
| Spring Authorization Server | Boot-managed starter for v1 local IdP | Local OAuth2/OIDC provider for development and simulations | Spring docs document the authorization server role and Boot starter, useful when no external IdP is provided for v1. |
| H2 | Boot-managed | In-memory persistence for simulation data | Spring Boot recognizes H2 as an embedded database and can initialize it automatically for development. |
| OpenXPKI | Current master docs / Open Source Trustcenter | PKI fallback for CSR intake, issuance, renewal, and revocation | KrakenD can consume certificates for mTLS, but OpenXPKI is the better fit for certificate lifecycle and workflow-backed issuance. |
| Docker Compose | Current Docker Compose | Local multi-container orchestration | Docker Compose is the standard local way to define and run gateway, backend, PKI, and support services together. |
| Terraform | Current Terraform CLI with providers | Cloud deployment automation | Terraform's provider model supports AWS, Google Cloud, AzureRM, and TLS resources with official provider documentation. |
### Supporting Libraries
| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `spring-boot-starter-web` or Spring MVC equivalent | Boot-managed | REST API endpoints | Backend is request/response API oriented and does not need reactive complexity for v1. |
| `spring-boot-starter-oauth2-resource-server` | Boot-managed | Validate Bearer/JWT access tokens | Every protected backend endpoint should validate tokens even when KrakenD also validates at the edge. |
| `spring-boot-starter-oauth2-authorization-server` | Boot-managed | Local authorization server | Use if no external IdP is selected for v1; keep it replaceable by configuration. |
| `spring-boot-starter-data-jpa` + H2 | Boot-managed | Simulation persistence | Use for Pix attempt records, statement records, and customer data fixtures. |
| KrakenD `auth/validator` | KrakenD 2.13.x | JWT validation at gateway | Use on protected endpoints with issuer/audience checks and JWK caching. |
| KrakenD TLS / `client_tls` | KrakenD 2.13.x | mTLS app-to-gateway and gateway-to-backend | Use for service mTLS and backend client certificates. |
| Terraform AWS provider | 6.x line in registry examples | AWS infrastructure | Use for AWS-specific modules. |
| Terraform Google provider | 7.x line in registry docs | GCP infrastructure | Use for GCP-specific modules. |
| Terraform AzureRM provider | 4.x line in registry docs | Azure infrastructure | Use for Azure-specific modules. |
| Terraform TLS provider | 4.2.x docs | Local/dev certificate material | Use only for development/demo trust material, not as the production CA. |
### Development Tools
| Tool | Purpose | Notes |
|------|---------|-------|
| Git submodules | Layer isolation | Superproject coordinates planning; implementation commits belong to `mobile-app`, `backend`, `api-gateway`, `infrastructure`, or `pki`. |
| Docker Compose | Local environment | Compose should run backend, KrakenD, optional local auth server, H2-backed backend, and PKI service. |
| KrakenD `check` / `audit` | Gateway config validation | Run in CI for `api-gateway` before merging gateway changes. |
| Gradle Kotlin DSL | Backend build | Use Spring Boot plugin and Boot-managed dependency versions. |
| Flutter test tooling | Mobile tests | Use unit/widget tests first, then integration tests against local Compose environment. |
| Terraform `fmt`, `validate`, and per-cloud plans | Infrastructure validation | Keep AWS/GCP/Azure modules separate with shared abstractions only where they do not obscure provider differences. |
## Installation
# Superproject
# Backend
# Gateway
# Local environment
# Mobile
# Terraform
## Alternatives Considered
| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| Spring Authorization Server for local v1 IdP | Keycloak | Use Keycloak if the goal shifts to a closer enterprise IAM simulation instead of Spring-only local auth. |
| H2 in-memory | PostgreSQL | Use PostgreSQL when persistence behavior, SQL compatibility, migrations, or concurrency become part of validation. |
| OpenXPKI for PKI | Terraform TLS provider | Use Terraform TLS only for local static certificates; it is not a runtime enrollment authority. |
| KrakenD CE | KrakenD Enterprise | Use Enterprise if multiple IdPs, enterprise policies, or enterprise-only integrations become required. |
| Horizontal layer roadmap | Vertical MVP roadmap | Use vertical MVP only if the priority changes from security-first architecture to fastest screen demo. |
## What NOT to Use
| Avoid | Why | Use Instead |
|-------|-----|-------------|
| `badCertificateCallback` accepting all certificates | It disables meaningful TLS verification and invalidates the mTLS security goal. | Load the project CA into `SecurityContext` and fail closed. |
| Direct app-to-backend calls | Violates the requested API Gateway boundary and bypasses central security policy. | App -> KrakenD -> backend only. |
| KrakenD as the full PKI lifecycle owner | KrakenD supports mTLS consumption, not full CA/enrollment lifecycle. | OpenXPKI or a dedicated PKI service. |
| Real Pix rails in v1 | Adds external dependencies and regulatory complexity unrelated to validating this architecture. | Backend-controlled success/error simulation. |
| One Terraform module hiding all cloud differences | Lowest-common-denominator modules tend to become confusing and unsafe. | Shared conventions plus cloud-specific AWS/GCP/Azure modules. |
## Stack Patterns by Variant
- Use Spring Authorization Server in `backend` or a dedicated backend module for local OAuth2/OIDC.
- Because it keeps OAuth2 real while avoiding external IAM setup.
- Keep backend as OAuth2 Resource Server and point KrakenD/Spring Security to the external JWK issuer.
- Because the gateway and backend should validate tokens from the same issuer.
- Keep mTLS enforcement in KrakenD but move enrollment, CSR approval, issuance, renewal, and revocation into `pki`.
- Because certificate lifecycle is a PKI concern, not a routing concern.
## Version Compatibility
| Package A | Compatible With | Notes |
|-----------|-----------------|-------|
| Flutter 3.41 | Dart 3.11.x runtime family | Verify exact Dart SDK pinned by Flutter during mobile scaffold. |
| Spring Boot 4.0.6 | Spring Security / Spring Authorization Server Boot-managed versions | Prefer Boot starters and dependency management instead of manually pinning Spring modules. |
| KrakenD CE 2.13.5 | KrakenD schema v2.13 / config version 3 | Use `$schema` for IDE support and `krakend check` in CI. |
| H2 | Spring Boot embedded database auto-configuration | Works for v1 simulation; do not assume behavior matches production databases. |
| Terraform providers | Provider-specific major versions | Pin each provider in `required_providers` and commit lock files inside `infrastructure`. |
## Sources
- https://docs.flutter.dev/release/whats-new - verified Flutter 3.41 release.
- https://api.flutter.dev/flutter/dart-io/HttpClient-class.html - verified `HttpClient` can use `SecurityContext`.
- https://api.flutter.dev/flutter/dart-io/SecurityContext-class.html - verified certificate/key support in Dart TLS context.
- https://docs.spring.io/spring-boot/reference/index.html - verified Spring Boot 4.0.6 stable docs and managed modules.
- https://docs.spring.io/spring-boot/how-to/data-initialization.html - verified H2 is treated as an embedded database.
- https://docs.spring.io/spring-security/reference/7.0/servlet/oauth2/index.html - verified OAuth2 roles and resource server support.
- https://docs.spring.io/spring-security/reference/servlet/oauth2/authorization-server/index.html - verified authorization server features.
- https://www.krakend.io/docs/ - verified KrakenD CE 2.13.5 current docs.
- https://www.krakend.io/docs/authorization/ - verified KrakenD authN/authZ model.
- https://www.krakend.io/docs/authorization/mutual-authentication/ - verified KrakenD mTLS support.
- https://www.krakend.io/docs/configuration/structure/ - verified KrakenD schema/config structure.
- https://openxpki.readthedocs.io/en/master/subsystems/rpc.html - verified OpenXPKI HTTP API for certificate request/renew/revoke.
- https://openxpki.readthedocs.io/en/master/configuration/workflows/enroll.html - verified enrollment workflow concepts.
- https://docs.docker.com/compose/ - verified Docker Compose role for multi-container apps.
- https://developer.hashicorp.com/terraform/language/providers - verified Terraform provider model.
- https://registry.terraform.io/providers/hashicorp/aws/latest/docs - verified official AWS provider.
- https://registry.terraform.io/providers/hashicorp/google/latest/docs - verified official Google provider.
- https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs - verified official AzureRM provider.
- https://registry.terraform.io/providers/hashicorp/tls/latest/docs - verified TLS provider scope.
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

Conventions not yet established. Will populate as patterns emerge during development.
<!-- GSD:conventions-end -->

<!-- GSD:architecture-start source:ARCHITECTURE.md -->
## Architecture

Architecture not yet mapped. Follow existing patterns found in the codebase.
<!-- GSD:architecture-end -->

<!-- GSD:skills-start source:skills/ -->
## Project Skills

No project skills found. Add skills to any of: `.claude/skills/`, `.agents/skills/`, `.cursor/skills/`, `.github/skills/`, or `.codex/skills/` with a `SKILL.md` index file.
<!-- GSD:skills-end -->

<!-- GSD:workflow-start source:GSD defaults -->
## GSD Workflow Enforcement

Before using Edit, Write, or other file-changing tools, start work through a GSD command so planning artifacts and execution context stay in sync.

Use these entry points:
- `/gsd-quick` for small fixes, doc updates, and ad-hoc tasks
- `/gsd-debug` for investigation and bug fixing
- `/gsd-execute-phase` for planned phase work

Do not make direct repo edits outside a GSD workflow unless the user explicitly asks to bypass it.
<!-- GSD:workflow-end -->



<!-- GSD:profile-start -->
## Developer Profile

> Profile not yet configured. Run `/gsd-profile-user` to generate your developer profile.
> This section is managed by `generate-claude-profile` -- do not edit manually.
<!-- GSD:profile-end -->
