## Purpose

Define the initial Flutter mobile banking journeys and their secure API
configuration expectations.

## Requirements

### Requirement: Mobile app exposes initial journeys
The Flutter mobile app SHALL expose screens for Pix transfer, account statement, and customer registration data.

#### Scenario: User opens the app after authentication
- **WHEN** an authenticated user reaches the main banking experience
- **THEN** the user can navigate to Pix transfer
- **AND** the user can navigate to account statement
- **AND** the user can navigate to customer registration data

### Requirement: Mobile app uses Flutter 3.41
The mobile application SHALL be built with Flutter 3.41.

#### Scenario: Mobile project is checked
- **WHEN** the mobile project toolchain is inspected
- **THEN** the Flutter version requirement is documented or pinned to the
  3.41 line

### Requirement: Mobile app uses secure API configuration
The mobile app SHALL use gateway API configuration and trusted TLS material for banking requests.

#### Scenario: Banking request is made
- **WHEN** the app sends a banking request
- **THEN** the request targets the configured KrakenD endpoint
- **AND** the TLS client uses trusted CA and client certificate material when
  mTLS is required
