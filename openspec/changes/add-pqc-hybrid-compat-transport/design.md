## Context

Post-quantum TLS is available in OpenSSL >= 3.5, BouncyCastle 1.86 and curl
8.16 (the servers, terminators and test clients). It is not usable from the
clients the app edge must serve: `dart:io` (BoringSSL with classical-only
default `signature_algorithms` and groups, no configuration surface on
`SecurityContext`) and browsers. HAProxy 3.2 maps certificate key types to
RSA/ECDSA/DSA and treats ML-DSA as "anonymous", so on a bind with both an ECDSA
and an ML-DSA certificate it serves ECDSA to any client advertising ECDSA and
ML-DSA to clients that do not (verified locally against OpenSSL 3.5.8 with
`s_client` in both roles, with and without SNI, with both client certificate
families).

## Decisions

- **Per-listener tiers, not per-environment profiles.** The same
  configuration must be deployable everywhere. Strict hops keep the previous
  policy verbatim (global HAProxy `ssl-default-*` values, BCJSSE properties).
  Only the app-facing `bind` lines widen curves and signature schemes, and
  `verify-pqc-gateway.sh` forbids the widened values anywhere else.
- **Dual identity, compatibility certificate first.** The first `crt` is the
  no-SNI default; IP-literal clients (a physical device on the LAN) are
  compatibility clients, ML-DSA-only clients always connect by hostname.
- **Separate ECDSA chain, never cross-signed.** A mobile stack must validate
  the server chain classically, so the chain must be classical end to end;
  mixing families inside one chain would make the issuer signature the weakest
  link and confuse chain selection. ECDSA P-384 CA, P-256 leaves,
  `ecdsa-with-SHA384` signatures, `digitalSignature` only. No service identity
  ever exists on this chain.
- **Chain by key family at issuance.** `sign-csr.sh` derives the family from
  the CSR (`pqc_key_family`) and signs with the matching CA; it writes
  `<leaf>.issuer` so the backend adapter returns the right chain without
  configuration. The backend validator mirrors the policy (`secp256r1` named
  curve only; explicit or implicit EC parameters rejected).
- **Mobile mode selection instead of fail-closed.** Both modes are PKI-issued,
  mutually authenticated transports. The ML-DSA private key is stored seed-only
  (RFC 9881), the only encoding BoringSSL parses, so the app is ready for a
  Dart release that enables ML-DSA without a further format change. The
  compatibility root is always trusted (a dual listener may serve the ECDSA
  chain to any ECDSA-capable client); the ML-DSA root only when loadable.
- **Hybrid group preferred, X25519 accepted, only on the app edge.** OpenSSL
  picks the first server-preferred group the client offers, so any client with
  `X25519MLKEM768` (browsers, curl, BCJSSE) still gets it; the strict hops keep
  the hybrid group as the only option.
- **RSA nowhere.** Neither chain, neither tier accepts RSA schemes.

## Alternatives considered

- Upgrade Flutter to 3.47 (Dart 3.13) and keep strict-only: Dart 3.13 parses
  ML-DSA but its BoringSSL still does not offer ML-DSA schemes or the hybrid
  group, so the app would still fail; rejected as insufficient (documented as
  the future path).
- Classical-only app edge: throws away the post-quantum authentication that
  BCJSSE clients and the gateway JWKS egress already perform; rejected.
- Separate hostnames or ports per client class: pushes the choice to DNS and
  the client; rejected in favour of TLS 1.3's own signature-scheme selection.
- A Cronet/NSURLSession-based mobile transport: gives hybrid key exchange but
  no client certificates, which the banking listener requires; rejected.

## Risks

- HAProxy's selection is by ECDSA advertisement, not by ML-DSA preference: a
  client that offers both gets ECDSA. Strict peers avoid this by offering
  ML-DSA only (BCJSSE properties, `-sigalgs` in the evidence scripts).
- The compatibility tier is a transition tier. Retiring it from the gateway
  binds is a one-line change per bind once the client stacks catch up.
