## Why

Pix is the flagship v1 journey, but integrating real Pix settlement rails is out
of scope and risky for a simulation. This capability defines a Pix transfer flow
where the app selects a success or error scenario and the backend returns the
matching simulated response, proving the end-to-end path without touching real
settlement systems.

## What Changes

- Support **app-selected simulated outcomes**: the Pix transfer flow lets the app
  choose a success or error scenario and receive the corresponding
  backend-controlled response through KrakenD, displayed in the app.
- Guarantee **no real settlement rails** in v1: Pix behavior is simulated; the
  backend records and responds to a simulation attempt and never calls an
  external Pix settlement provider.

## Impact

- **backend**: Pix simulation endpoint with success/error branches.
- **mobile-app**: Pix transfer flow with scenario selection and outcome display.
- **Scope guard**: no external Pix integration in v1.
