## Purpose

Define the Spring Boot Kotlin backend requirements for local simulation APIs,
OAuth2-protected banking data, and in-memory development persistence.

## Requirements

### Requirement: Backend uses Spring Boot Kotlin
The backend SHALL be implemented with Spring Boot Kotlin 4.0.6.

#### Scenario: Backend project is checked
- **WHEN** the backend build configuration is inspected
- **THEN** it uses the Spring Boot Kotlin 4.0.6 line or the documented project
  equivalent

### Requirement: Backend supports v1 in-memory persistence
The backend SHALL support H2 or another in-memory database for v1 development and simulation data.

#### Scenario: Local backend starts
- **WHEN** the backend starts in local development mode
- **THEN** simulation data can be stored without requiring an external database

### Requirement: Backend exposes banking simulation APIs
The backend SHALL expose APIs for Pix simulation, account statement data, and customer registration data.

#### Scenario: Authenticated request for statement
- **WHEN** an authenticated request asks for account statement data through
  KrakenD
- **THEN** the backend returns statement records from local simulation data

#### Scenario: Authenticated request for customer registration data
- **WHEN** an authenticated request asks for customer registration data through
  KrakenD
- **THEN** the backend returns the configured customer data fixture
