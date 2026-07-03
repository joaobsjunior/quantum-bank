## Why

Quantum Bank must prove a secure end-to-end banking flow from the Flutter app
through KrakenD to the Spring Boot backend. The v1 priority is the communication
security foundation: the mobile app must never talk to the backend directly, and
protected access must be gated by OAuth2 and mutual TLS with fail-closed
verification. This capability defines that secure communication contract.

## What Changes

- Establish **gateway-only** mobile communication: the app reaches backend
  capabilities exclusively through KrakenD; backend service addresses are never
  the mobile runtime base URL.
- Enforce **OAuth2 bearer-token validation** at both KrakenD (edge) and the
  Spring Boot resource server (backend).
- Enforce **mutual TLS** for app-to-gateway and gateway-to-backend channels
  wherever certificate material is provisioned.
- Require the mobile TLS client to **fail closed**: load the project CA trust
  material and never disable verification with permissive certificate callbacks.

## Impact

- **api-gateway**: KrakenD becomes the sole mobile-facing entrypoint with OAuth2
  and mTLS policies.
- **backend**: Spring Security resource-server validation on protected APIs.
- **mobile-app**: gateway base URL + trusted CA/client-cert TLS configuration.
- **Security posture**: no direct backend bypass, no permissive TLS.
