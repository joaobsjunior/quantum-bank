## Why

The strict post-quantum transport (`add-pqc-ml-dsa-transport`) is only reachable
by peers that verify ML-DSA in TLS. Reading the TLS stacks the app edge must
serve shows none of the client stacks does: Dart 3.11 rejects ML-DSA material,
Dart 3.13 parses it but its BoringSSL neither offers ML-DSA in
`signature_algorithms` nor `X25519MLKEM768` in its default groups, and
`SecurityContext` exposes neither; browsers accept no ML-DSA certificate at all.
A production deployment of the mobile app was therefore impossible, and the
"known limitation" could not be closed by any Flutter upgrade.

## What Changes

- **Two listener tiers instead of one policy.** Every service-to-service hop
  stays strict (ML-DSA-65/87 only, `X25519MLKEM768` only). The app-facing
  listeners (issuer, gateway bootstrap, gateway banking) become dual-identity:
  HAProxy serves the ML-DSA-65 certificate to clients that only offer ML-DSA
  schemes and an ECDSA P-256 compatibility certificate to every other client,
  prefers `X25519MLKEM768`, accepts `X25519`, and refuses RSA. This was
  validated with HAProxy 3.2 built against OpenSSL 3.5.8.
- **Second PKI chain.** An ECDSA P-384 root and issuing CA (never cross-signed
  with the ML-DSA chain) issues the app-facing server identities and the mobile
  client certificates enrolled from an ECDSA P-256 CSR. `sign-csr.sh` picks the
  chain by key family and writes the issuing certificate next to the leaf.
- **CSR policy.** ML-DSA-65, ML-DSA-87 and ECDSA P-256 (`secp256r1` named
  curve) are accepted; RSA, EdDSA, ML-DSA-44 and every other curve stay
  rejected, in the backend validator and in the PKI script.
- **Mobile transport mode.** The startup probe now selects a mode instead of
  failing closed: ML-DSA-65 identity (PKCS#8 seed-only, RFC 9881) and both
  roots on a post-quantum capable stack, ECDSA P-256 identity and the
  compatibility root otherwise. The mode is shown to the user.
- **Evidence.** Handshake, smoke and negative tests cover both client classes
  on every listener; the strict backend hop is proven to refuse classical
  schemes, classical groups and compatibility identities.
- **BREAKING**: the app-facing listeners no longer refuse classical-only
  clients; `negative-mtls-tests.sh` moves the classical-group refusal cases to
  the backend port; new trust anchors (`root-ca-compat.crt`,
  `issuing-ca-compat.crt`) and runtime bundles (`ca-chain-all.crt`,
  `trust-anchors.crt`) are required by the terminators and the test clients.

## Capabilities

### Modified Capabilities
- `secure-gateway-communication`: strict and app-facing listener tiers, dual
  identity, client certificates from either chain.
- `onboarding-pki`: compatibility chain, ECDSA P-256 CSR intake, chain
  selection by key family, mobile transport mode and key family.
- `deployment-infrastructure`: dual-identity issuer terminator, both device
  roles in the runtime tests, Terraform policy with compatibility values.
- `external-service-integration`: `backend-client` stays a strict ML-DSA-only
  peer and is served the ML-DSA identity by the dual listeners.
- `mobile-banking-journeys`: transport mode replaces the fail-closed gate.

## Impact

- Submodules: `pki`, `api-gateway`, `infrastructure`, `backend`,
  `backend-client` (docs only), `mobile-app`; superproject README, AGENTS,
  `docs/pqc-ml-dsa-transport.md`.
- No new tooling: OpenSSL >= 3.5 already builds ECDSA material; HAProxy 3.2
  already selects certificates by client signature schemes.
- The compatibility tier is removable per listener once the client stacks
  offer ML-DSA and the hybrid group.
