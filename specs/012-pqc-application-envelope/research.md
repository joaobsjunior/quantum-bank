# Research: Post-Quantum Application Envelope

## Decision 1: Hybrid KEM construction

**Decision**: single-shot envelope: `ss = ML-KEM-768.Encaps(ek_backend) ||
X25519(sk_eph, pk_backend)`; `prk = HKDF-Extract(salt = "quantum-bank-envelope-v1", ikm = ss_mlkem || ss_x25519)`;
request key `HKDF-Expand(prk, "request" || kid || aad, 32)`, response key
`HKDF-Expand(prk, "response" || kid || aad, 32)`; AES-256-GCM with a random
96-bit nonce per message; `aad = method + " " + path`.

**Rationale**: mirrors the `X25519MLKEM768` TLS construction (concatenated
secrets, FIPS 203 KEM + RFC 7748) so the security argument is the same one the
transport layer relies on; HKDF-SHA-256 and AES-256-GCM are available in pure
Dart (pointycastle) and the JDK. The response key is bound to the request
secret, so a response can only be produced by the party that opened the
request.

**Alternatives considered**: full HPKE (RFC 9180) with `DHKEM(X25519)` +
ML-KEM combiner: no interoperable Dart implementation; a bespoke combiner would
be as much new code as this construction with less clarity. TLS-only: not
achievable on `dart:io` today (see feature 011 capability matrix).

## Decision 2: Key set delivery and trust

**Decision**: the backend signs the canonical key set with its PKI-issued
ML-DSA-65 identity (the same `backend-server.p12` the TLS listener uses) and
returns `envelopeKeys` in the CSR response. The app verifies the signature and
the chain in Dart against the bundled `root-ca.crt` (ML-DSA-87).

**Rationale**: no new trust root to distribute; the certificate lifecycle the
PKI already runs covers the signer; verification in Dart is independent of the
platform TLS stack, which is the point of the feature.

**Alternatives considered**: a dedicated envelope-signer key as an app asset
(second root to rotate); a `GET /envelope/keys` route (new gateway surface,
same trust problem); pinning the key set at build time (no rotation).

## Decision 3: Who must use the envelope

**Decision**: policy by OAuth2 client (`azp` / `client_id` claim):
`quantum-bank.envelope.required-for-clients` defaults to
`[quantum-bank-mobile]`. Other clients (backend-client, `quantum-bank-test`)
are on the strict TLS tier and are exempt.

**Rationale**: the backend cannot see the TLS tier (the gateway terminates
it), but the OAuth2 client is a faithful proxy: the mobile client is the only
one served the dual tier. Keeping service clients exempt avoids duplicating the
envelope in `backend-client` and in the curl-based smoke tests.

## Decision 4: Transaction signature

**Decision**: ML-DSA-65 (FIPS 204, context `quantum-bank-pix-v1`) over a
canonical UTF-8 string of the order fields, with a UUID nonce and `issuedAt`
(±2 min skew). The signing key is separate from the transport identity,
generated at enrollment, registered with a proof of possession over the DER
CSR (context `quantum-bank-signing-key-v1`).

**Rationale**: a separate key lets the transport identity move to hardware
(ECDSA in Secure Enclave/StrongBox, feature 013) without weakening the
post-quantum signature; binding the registration to the CSR ties it to the
OTK and subject; contexts give domain separation between the two uses.

**Alternatives considered**: signing with the TLS device key (blocks hardware
backing and, in compatibility mode, would be ECDSA); HashML-DSA (unnecessary,
messages are small); JWS with ML-DSA (no library support on the Keycloak side,
and the token is not the order).

## Decision 5: X25519 in Dart

**Decision**: `package:cryptography` 2.9 (`X25519()` algorithm, pure Dart on
the VM, RFC 7748 test vectors in its suite).

**Rationale**: `pointycastle` 4.0 has no RFC 7748 X25519 and `pqcrypto` has no
classical primitives; writing a Montgomery ladder for a banking app would be
unreviewed cryptography.

## Decision 6: BouncyCastle ML-KEM on JDK 17

**Decision**: JCA `KeyPairGenerator("ML-KEM-768", "BC")`,
`KeyGenerator("ML-KEM", "BC")` with `KEMGenerateSpec/KEMExtractSpec` built
`withNoKdf()` so the extracted secret is the raw FIPS 203 shared secret; raw
public key bytes through `MLKEMPublicKey.getPublicData()`.

**Rationale**: JDK 17 has no `javax.crypto.KEM`; BC's spec API is the
documented path and `withNoKdf()` keeps the secret identical to the Dart side.

## Decision 7: Transport mode policy

**Decision**: `PQC_TRANSPORT_POLICY` build setting: `compatibility` (default)
or `probe` (feature 011 behaviour). The gate screen shows the mode and that the
envelope is active.

**Rationale**: a certificate-loading probe is a false positive on Dart 3.13
(certificates load, handshake signing does not); with the envelope in place
the compatibility transport is the safe default and the post-quantum transport
becomes an explicit operator decision.
