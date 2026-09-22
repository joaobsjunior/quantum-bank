# Feature Specification: PKI Lifecycle and Cloud Passthrough Hardening

**Feature Branch**: `014-pki-lifecycle-hardening`

**Created**: 2026-09-19

**Status**: Planned (specification and plan; implementation follows feature 012)

**Input**: Recommendation item 5: the server side is already post-quantum
first; what is missing for production is the certificate lifecycle
(revocation, renewal) and a deploy-time guarantee that the cloud ingress
passes TLS through to the terminators.

## Overview

The PKI issues ML-DSA-65 service identities, ML-DSA-65/ECDSA P-256 device
identities and the envelope signer identity. Today rotation is manual
re-issuance and revocation only exists as a local script. This feature adds:

1. **Revocation**: a CRL per issuing CA (both chains), published with the
   runtime material, checked by the banking terminator (`crl-file`) and by the
   backend truststore (BCJSSE revocation checking), plus device revocation
   through the backend (`DELETE /auth/devices/{deviceId}` for operators).
2. **Renewal**: service identities re-issued before expiry by a scheduled
   script with hot reload of the terminators; device certificates renewed by
   the app through a fresh OTK + CSR when less than 20% of validity remains.
3. **Cloud passthrough gate**: Terraform validation that every internet-facing
   listener in front of the gateway sidecar is TCP passthrough (AWS NLB TCP,
   Azure Container Apps TCP ingress, GCP TCP proxy in front of Cloud Run) and
   that no managed HTTPS termination exists on the path.
4. **Token lifetime**: Keycloak access tokens at 5 minutes stay; refresh
   tokens for the mobile client limited to 30 minutes idle and bound to the
   client, documented as the maximum exposure of an RS256 token inside the
   envelope.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - A revoked device stops working within minutes (Priority: P1)

**Acceptance Scenarios**:

1. **Given** an enrolled device, **When** an operator revokes it, **Then**
   its serial is in the issuing CA's CRL, the banking terminator refuses its
   next handshake, and its signing key is deleted so Pix signatures fail.
2. **Given** a revoked service identity, **When** the gateway egress or the
   backend sees it, **Then** the strict-tier handshake fails.

### User Story 2 - Identities renew before they expire (Priority: P1)

**Acceptance Scenarios**:

1. **Given** a service certificate at 80% of its validity, **When** the
   renewal job runs, **Then** a new ML-DSA-65 identity is issued and the
   terminator reloads without dropping connections.
2. **Given** a device certificate at 80% of its validity, **When** the app
   starts, **Then** it re-enrolls (OTK + CSR + signing key) and the old
   certificate is revoked after the new one is active.

### User Story 3 - Deploys refuse a terminating ingress (Priority: P1)

**Acceptance Scenarios**:

1. **Given** a Terraform plan for any cloud, **When**
   `verify-terraform-config.sh` runs, **Then** it fails if a listener in
   front of the gateway sidecar terminates TLS (HTTPS listener, managed
   certificate, HTTP(S) load balancer) instead of TCP passthrough.

### Edge Cases

- CRL distribution to terminators in the cloud: shared volume or object
  storage with signed CRLs; a stale CRL past `nextUpdate` MUST fail closed on
  the strict tier and alert on the app edge.
- ML-DSA CRLs are large (ML-DSA-87 signature 4627 bytes); HAProxy and BCJSSE
  handle them, but the update cadence must be bounded (hourly).

## Requirements *(mandatory)*

- **FR-001**: Both issuing CAs MUST publish a CRL signed with their ML-DSA-87
  / ECDSA P-384 key; `revoke-local-cert.sh` MUST regenerate it.
- **FR-002**: The banking terminator MUST check the client certificate
  against `ca-chain-all.crl`; the backend MUST enable revocation checking for
  gateway identities.
- **FR-003**: The backend MUST expose an operator-only device revocation
  that revokes the certificate and deletes the signing key.
- **FR-004**: Renewal of service identities MUST be scripted and MUST reload
  the terminators; device renewal MUST be automatic in the app.
- **FR-005**: `verify-terraform-config.sh` MUST fail on any TLS-terminating
  ingress in front of the sidecar, per cloud.
- **FR-006**: Keycloak refresh tokens for the mobile client MUST be
  client-bound and limited to 30 minutes idle.

## Success Criteria *(mandatory)*

- **SC-001**: `negative-mtls-tests` includes a revoked device and a revoked
  service identity, both refused.
- **SC-002**: A renewal run on the Compose stack keeps `smoke-tests` green
  before and after.
- **SC-003**: A deliberately wrong Terraform variant (HTTPS listener) fails
  the validation script on all three clouds.
