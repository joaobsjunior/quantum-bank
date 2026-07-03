## Why

Quantum Bank's v1 must exercise the real mobile app -> gateway -> backend path
through concrete user journeys. This capability defines the initial Flutter
screens (Pix transfer, account statement, customer registration data) and the
secure API configuration they use so the journeys drive genuine gateway traffic.

## What Changes

- Expose the three **initial journeys** in the Flutter app: Pix transfer,
  account statement, and customer registration data, reachable after
  authentication.
- Pin the mobile app to **Flutter 3.41** (documented or pinned in the toolchain).
- Use **secure API configuration**: banking requests target the configured
  KrakenD endpoint and the TLS client uses trusted CA and client-certificate
  material when mTLS is required.

## Impact

- **mobile-app**: journey screens, navigation, gateway endpoint configuration,
  and TLS trust material wiring.
- **Depends on**: secure-gateway-communication and backend-simulation-api.
