## MODIFIED Requirements

### Requirement: PKI algorithm policy is post-quantum only
The PKI SHALL run two trust chains that never cross-sign each other: a
post-quantum chain (ML-DSA-87 root and issuing CA, ML-DSA-65 or ML-DSA-87
leaves) used by every service identity and every strict hop, and a
compatibility chain (ECDSA P-384 root and issuing CA, `ecdsa-with-SHA384`
signatures, ECDSA P-256 leaves) used only by the app-facing server identities
and by mobile client certificates enrolled from ECDSA P-256 keys. The PKI
scripts SHALL require OpenSSL >= 3.5 and fail closed without it.

#### Scenario: Trust anchors are verified
- **WHEN** `verify-trust-anchors.sh` inspects the four anchors
- **THEN** the post-quantum anchors carry ML-DSA-87 keys and signatures, the
  compatibility anchors carry ECDSA P-384 keys and `ecdsa-with-SHA384`
  signatures, each chain validates only against its own root, and both mobile
  root assets match

#### Scenario: Runtime certificates are generated
- **WHEN** `bootstrap-runtime-certs.sh` runs
- **THEN** every service identity is ML-DSA-65 under the post-quantum chain,
  only `gateway-server-compat`, `keycloak-server-compat` and the ECDSA smoke
  client exist under the compatibility chain, and the union bundles carry both
  chains

### Requirement: CSR submission is validated before PKI handoff
On `POST /auth/csr` the backend SHALL, before any PKI handoff, verify the
proof-of-possession signature and accept only ML-DSA-65, ML-DSA-87 or ECDSA
P-256 (`secp256r1` named curve) subject public keys, rejecting RSA, EdDSA,
ML-DSA-44, every other curve and EC keys with explicit or implicit parameters
with `csr_key_rejected`; the PKI SHALL issue the certificate under the chain
of the key family and return that chain's issuing certificate with the leaf.

#### Scenario: ECDSA P-256 CSR is enrolled
- **WHEN** a valid ECDSA P-256 CSR is submitted with a valid OTK
- **THEN** the certificate is signed with `ecdsa-with-SHA384` by the
  compatibility issuing CA and the response chain carries that issuing
  certificate

#### Scenario: ML-DSA CSR is enrolled
- **WHEN** a valid ML-DSA-65 or ML-DSA-87 CSR is submitted with a valid OTK
- **THEN** the certificate is signed with ML-DSA-87 by the post-quantum
  issuing CA and the response chain carries that issuing certificate

#### Scenario: Other key families are submitted
- **WHEN** the CSR key is RSA, EdDSA, ML-DSA-44 or an EC key that is not the
  `secp256r1` named curve
- **THEN** the backend responds `csr_key_rejected` and the PKI script refuses
  to sign

### Requirement: Mobile identity is post-quantum
The mobile app SHALL select its device key family from the platform TLS
stack: an ML-DSA-65 key pair (PKCS#8 seed-only, RFC 9881) when the stack loads
ML-DSA material, an ECDSA P-256 key pair (PKCS#8 wrapping RFC 5915) otherwise,
and SHALL build the PKCS#10 request with the matching proof-of-possession
signature.

#### Scenario: CSR interoperability
- **WHEN** the app-generated CSR of either family and its PKCS#8 key are
  checked with OpenSSL >= 3.5
- **THEN** the proof of possession verifies, the key family matches, and the
  private key re-derives the CSR public key
