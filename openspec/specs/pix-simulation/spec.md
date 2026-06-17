## Purpose

Define the v1 Pix simulation behavior, including app-selected success and error
paths without real Pix settlement integration.

## Requirements

### Requirement: Pix transfer supports simulated outcomes
The Pix transfer flow SHALL allow the app to select a success or error scenario and receive the corresponding backend-controlled response.

#### Scenario: Success scenario is selected
- **WHEN** the app submits a Pix transfer with the success simulation selected
- **THEN** the backend returns a successful Pix simulation response through
  KrakenD
- **AND** the app displays the successful outcome

#### Scenario: Error scenario is selected
- **WHEN** the app submits a Pix transfer with an error simulation selected
- **THEN** the backend returns the configured Pix error response through
  KrakenD
- **AND** the app displays the error outcome

### Requirement: Pix v1 does not use real settlement rails
The v1 Pix implementation SHALL simulate Pix behavior without integrating with real Pix settlement systems.

#### Scenario: Pix transfer is submitted
- **WHEN** a Pix transfer is submitted in v1
- **THEN** the backend records and responds to a simulation attempt
- **AND** no external Pix settlement provider is called
