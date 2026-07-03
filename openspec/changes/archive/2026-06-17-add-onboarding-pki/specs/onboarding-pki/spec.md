## ADDED Requirements

### Requirement: OTK starts device onboarding
The app SHALL use a one-time-key onboarding flow before provisioning runtime client certificate material.

#### Scenario: Valid OTK onboarding
- **WHEN** the app submits a valid one-time key during onboarding
- **THEN** the backend accepts the onboarding attempt
- **AND** the app can proceed to CSR generation and certificate provisioning

#### Scenario: Invalid OTK onboarding
- **WHEN** the app submits an invalid, expired, or reused one-time key
- **THEN** onboarding is rejected
- **AND** no client certificate is issued

### Requirement: CSR-driven client certificate issuance
The app SHALL generate or provide a certificate signing request during onboarding so the PKI layer can issue a client certificate for mTLS.

#### Scenario: Valid CSR is submitted
- **WHEN** onboarding is approved and the app submits a valid CSR
- **THEN** the PKI layer issues a client certificate bound to the approved
  onboarding context

#### Scenario: Invalid CSR is submitted
- **WHEN** the CSR is malformed or does not match the onboarding context
- **THEN** certificate issuance is rejected

### Requirement: PKI lifecycle is explicit
The project SHALL include certificate lifecycle capability for issuance, renewal, and revocation.

#### Scenario: KrakenD lifecycle is insufficient
- **WHEN** KrakenD cannot satisfy runtime certificate issuance, renewal, and
  revocation requirements
- **THEN** an open source PKI such as OpenXPKI is used for lifecycle workflows

#### Scenario: Certificate is revoked
- **WHEN** a client certificate is revoked
- **THEN** subsequent mTLS attempts with that certificate are rejected after
  revocation state is enforced
