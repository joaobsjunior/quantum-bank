# backend-client-console Specification

## Purpose
TBD - created by archiving change add-backend-client-service. Update Purpose after archive.
## Requirements
### Requirement: Console drives every external-integration flow
The `backend-client` SHALL provide a human-facing web console that lets a person
manually trigger every external-integration flow — Pix transfer, account
statement, and profile / registration data — without writing code or crafting
requests by hand.

#### Scenario: Person runs a flow from the console
- **WHEN** a person selects a banking flow in the console and submits it
- **THEN** the console triggers the corresponding external-integration call
  through the gateway
- **AND** the outcome of that call is shown in the console

#### Scenario: Console offers parity with the mobile journeys
- **WHEN** a person opens the console
- **THEN** the Pix transfer, account statement, and profile flows are each
  available to run
- **AND** each flow maps to the same gateway-fronted capability the mobile app
  uses

### Requirement: Console lets the operator select the Pix scenario
The console SHALL let the operator choose the Pix success or error simulation
scenario before submitting a Pix transfer.

#### Scenario: Operator selects a Pix scenario
- **WHEN** the operator chooses a success or error scenario and submits a Pix
  transfer
- **THEN** the `backend-client` submits that selected scenario through the
  gateway
- **AND** the console displays the corresponding simulated success or error
  result

### Requirement: Console surfaces request, response, and correlation detail
For each executed flow the console SHALL display the outbound request summary,
the gateway response, the `correlationId`, and any RFC 9457 problem-details error
body, so the operator can observe the secure path end to end.

#### Scenario: Successful call detail is shown
- **WHEN** a flow completes successfully
- **THEN** the console shows the request summary, the response, and the
  correlation id

#### Scenario: Error detail is shown
- **WHEN** a flow returns a problem-details error
- **THEN** the console shows the error type, status, detail, and correlation id
- **AND** it does not hide the failure behind a generic message

### Requirement: Console never exposes a direct-to-backend path
The console SHALL route every flow through the gateway-fronted
external-integration client and MUST NOT offer any option that calls a backend
service directly or bypasses OAuth2 and mTLS.

#### Scenario: No bypass control is available
- **WHEN** a person uses any control in the console
- **THEN** the resulting call goes through the gateway with the service token and
  mTLS identity
- **AND** no console action can reach the backend directly

