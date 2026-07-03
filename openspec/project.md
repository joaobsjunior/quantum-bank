# Quantum Bank

## Purpose

Quantum Bank is a mobile banking application with three initial user journeys:
Pix transfer, account statement, and customer registration data. The product
must prove a secure end-to-end banking flow from Flutter through KrakenD to a
Spring Boot Kotlin backend.

The v1 priority is the communication security foundation: OAuth2
authentication, OTK-based onboarding, CSR-driven certificate generation, mTLS,
and PKI. Initial screens may stay simple, but they must exercise the real
mobile app -> API gateway -> backend path.

## Core Value

The app must prove a secure end-to-end banking flow from Flutter through
KrakenD to the backend, including Pix success and error simulation.

## Stack

- Flutter 3.41 for the mobile app.
- Dart `dart:io` TLS APIs for trusted CA configuration and client certificate
  material.
- KrakenD Community Edition 2.13.x for API gateway routing, JWT validation,
  and mTLS enforcement.
- Spring Boot Kotlin 4.0.6 for backend APIs.
- Spring Security OAuth2 Resource Server for protected backend endpoints.
- Spring Authorization Server or a replaceable local IdP for v1 OAuth2/OIDC.
- H2 or in-memory persistence for v1 simulation data.
- OpenXPKI or equivalent open source PKI if KrakenD does not satisfy the full
  certificate lifecycle.
- Docker Compose for local multi-container validation.
- Terraform for AWS, GCP, and Azure deployment paths.

## Architecture

- All mobile/backend communication goes through KrakenD.
- Backend services must still validate OAuth2 tokens; gateway validation is not
  the only authorization boundary.
- Runtime certificate lifecycle belongs to PKI/onboarding components, while
  KrakenD consumes certificates and enforces mTLS.
- Pix v1 is simulated by backend-controlled success and error scenarios; it
  does not integrate with real Pix settlement rails.
- The repository is coordinated as a superproject with layer submodules:
  `mobile-app`, `backend`, `api-gateway`, `infrastructure`, and `pki`.

## Development Workflow

- Use OpenSpec for project planning and change control.
- Check existing specs before implementing a change:
  `openspec list --specs` and `openspec show <spec-id>`.
- For new behavior, create a proposal with `/opsx:propose` or
  `openspec new change <change-id>` before implementation.
- Keep implementation commits in the relevant layer repository when submodules
  are initialized.

## Constraints

- Do not call the backend directly from the mobile app.
- Do not use permissive TLS bypasses such as accepting all certificates.
- Do not use Terraform TLS provider as the production CA.
- Do not hide cloud differences behind one lowest-common-denominator Terraform
  module.
- Do not implement real Pix rails in v1.
