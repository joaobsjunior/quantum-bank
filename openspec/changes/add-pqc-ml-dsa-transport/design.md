## Context

Post-quantum TLS is available in OpenSSL >= 3.5 (native ML-DSA, ML-KEM),
BouncyCastle 1.86 (`bcprov`, `bctls` with `mldsa65/87` and `X25519MLKEM768`)
and curl 8.16. It is **not** available in Go 1.25/1.26 `crypto/tls` (KrakenD
2.13), the JDK 17/21/25 JSSE (Keycloak, plain Tomcat) or the BoringSSL build
inside Dart 3.11. The design keeps every product decision (KrakenD as the only
app-facing route, Keycloak as issuer, Spring Boot backend, Flutter app) and
moves the TLS termination to components that can speak post-quantum.

## Decisions

- **PQC terminator sidecars (HAProxy 3.2 + OpenSSL 3.5)** for KrakenD and
  Keycloak, sharing the network namespace (`network_mode: service:`). The
  paired process binds `127.0.0.1` only; loopback never leaves the namespace,
  so no unauthenticated byte crosses a container/host boundary. HAProxy was
  chosen over nginx because HAProxy's `verify required` fails a missing or
  invalid client certificate *inside* the handshake (TLS alert), which the
  existing negative matrix requires, whereas nginx answers HTTP 400.
- **KrakenD egress through loopback HAProxy**: backend calls and JWKS
  retrieval leave KrakenD as plain HTTP to `127.0.0.1:18081/18082`, where
  HAProxy adds the gateway's ML-DSA client identity (backend) or
  server-authenticates the issuer (JWKS). `disable_jwk_security` is allowed
  only for that loopback URL and verified by `verify-pqc-gateway.sh`.
- **BCJSSE in the JVM services** (backend Tomcat connector and JWK fetch,
  `backend-client` HttpClient) with a policy object that installs providers,
  pins `jdk.tls.*` to `mldsa65,mldsa87` / `X25519MLKEM768` / `TLSv1.3` and
  fails closed if the default `SSLContext` is not BCJSSE. The JDK keeps
  parsing PKCS#12 and X.509; BC provides the ML-DSA primitives.
- **Algorithm policy**: CA keys ML-DSA-87; leaf keys ML-DSA-65; CSR accepts
  ML-DSA-65/87 only; TLS accepts `mldsa65:mldsa87` only and the single group
  `X25519MLKEM768` (hybrid, so a hypothetical ML-KEM weakness still leaves
  X25519). JWT signatures stay RS256 (Keycloak/KrakenD have no ML-DSA JWS);
  the token only ever travels inside post-quantum TLS.
- **Mobile**: pure-Dart ML-DSA (`pqcrypto`, KAT-verified) for the identity key
  and CSR, PKCS#8 `both` encoding, startup probe of the platform TLS stack and
  fail-closed gating; the socket-layer gap is stated, not hidden.

## Alternatives considered

- Keep KrakenD/Keycloak TLS with classical certificates and only make the
  mobile client PQC: violates "every hop"; rejected.
- Replace KrakenD with a Go-free gateway: violates the KrakenD constraint.
- Use `jwk_local_path` with a JWKS file: static, misses key rotation; the
  loopback egress keeps dynamic retrieval and post-quantum authentication.

## Risks

- `dart:io` transport gap on mobile until Dart/Flutter ship ML-DSA in
  BoringSSL or a native TLS plugin is adopted (tracked in the mobile README).
- Cloud ingress services that terminate TLS at the provider edge (Cloud Run,
  Container Apps HTTP ingress) cannot present ML-DSA; the Terraform paths carry
  the terminator sidecar and document the passthrough requirement.
