## Why

The secure end-to-end flow needs a backend that serves the three v1 banking
journeys (Pix, statement, registration data) from local simulation data, without
depending on external systems. This capability defines the Spring Boot Kotlin
backend, its in-memory persistence for development, and the OAuth2-protected
simulation APIs consumed through KrakenD.

## What Changes

- Implement the backend with **Spring Boot Kotlin 4.0.6** (or the documented
  project equivalent).
- Support **in-memory persistence** (H2 or equivalent) so simulation data works
  without an external database in v1.
- Expose **banking simulation APIs** for Pix, account statement, and customer
  registration data, returning records from local simulation fixtures for
  authenticated requests routed through KrakenD.

## Impact

- **backend**: Spring Boot Kotlin service, H2 dev persistence, simulation
  controllers/fixtures for the three journeys.
- **Consumers**: mobile-app journeys and Pix simulation depend on these APIs,
  reached only through the secure gateway path.
