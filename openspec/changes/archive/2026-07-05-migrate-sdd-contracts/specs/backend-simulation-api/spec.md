## ADDED Requirements

### Requirement: Route-to-backend responsibility mapping
The backend SHALL implement the app-facing gateway routes with these
responsibilities: `POST /auth/otk` issues an OTK bound to the authenticated
subject, app, device, and profile; `POST /auth/csr` validates and consumes the
OTK and hands off the CSR; `POST /pix/transfers` returns a scenario-driven Pix
result; `GET /statements` returns statement entries for the subject;
`GET /profile` returns customer registration data; and `PUT /profile` edits it.
Backend behavior SHALL remain compatible with
`api-gateway/openapi/quantum-bank-v1.yaml`.

#### Scenario: Authenticated statement request
- **WHEN** an authenticated `GET /statements` request arrives through KrakenD
- **THEN** the backend returns statement entries for the authenticated subject
  matching the `StatementResponse` schema

#### Scenario: Backend stays contract-compatible
- **WHEN** a backend route response is produced
- **THEN** its shape matches the corresponding schema in
  `api-gateway/openapi/quantum-bank-v1.yaml`

### Requirement: Pix simulation is scenario-driven and persisted
For `POST /pix/transfers` the backend SHALL interpret the caller-provided
`scenario`: `SUCCESS` persists and returns status `COMPLETED` with the success
schema, and `ERROR` persists status `FAILED` and returns
`application/problem+json` with stable `errorCode` `pix_simulated_error`. The
backend SHALL NOT rely on random failure to produce the error path.

#### Scenario: Success scenario
- **WHEN** a Pix transfer is submitted with `scenario` `SUCCESS`
- **THEN** the backend persists a `COMPLETED` outcome and returns the success
  response

#### Scenario: Error scenario
- **WHEN** a Pix transfer is submitted with `scenario` `ERROR`
- **THEN** the backend persists a `FAILED` outcome and returns
  `application/problem+json` with `errorCode` `pix_simulated_error`

### Requirement: Statement and profile responses are deterministic
`GET /statements` and `GET /profile` SHALL return deterministic local-v1 data for
the authenticated subject, each including a `correlationId` and matching the
gateway OpenAPI `StatementResponse` and `ProfileResponse` schemas.

#### Scenario: Deterministic profile read
- **WHEN** the authenticated subject requests `GET /profile`
- **THEN** the backend returns the configured customer data fixture with a
  `correlationId` matching `ProfileResponse`

### Requirement: Profile write requires the profile:write scope
`PUT /profile` SHALL update editable customer registration fields for the
authenticated subject and SHALL require the `profile:write` scope; `profile:read`
is not sufficient.

#### Scenario: Write without the write scope
- **WHEN** a `PUT /profile` request carries only `profile:read`
- **THEN** the request is rejected and no profile change is persisted

#### Scenario: Write with the write scope
- **WHEN** a `PUT /profile` request carries `profile:write`
- **THEN** the backend updates the editable profile fields

### Requirement: Backend app-facing errors use problem details
All app-facing backend failures routed through KrakenD SHALL use
`application/problem+json` including `type`, `title`, `status`, `errorCode`, and
`correlationId`, with optional `detail`, `instance`, and `fieldErrors` when safe
for display.

#### Scenario: App-facing failure shape
- **WHEN** a backend route fails for an app-facing request
- **THEN** the response is `application/problem+json` with `type`, `title`,
  `status`, `errorCode`, and `correlationId`
