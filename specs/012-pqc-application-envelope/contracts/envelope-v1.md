# Contract: Quantum Bank Envelope v1

Media type: `application/vnd.quantum-bank.envelope+json`
Header (requests without a body): `X-Quantum-Envelope: <base64url(JSON EnvelopeMessage without ciphertext)>`
Response header: `X-Quantum-Envelope-Content-Type: <media type of the decrypted body>`

## Request (with body)

```http
POST /pix/transfers HTTP/1.1
Authorization: Bearer <RS256 JWT>
Content-Type: application/vnd.quantum-bank.envelope+json

{"v":1,"kid":"…","mlkemCiphertext":"<b64 1088B>","x25519PublicKey":"<b64 32B>",
 "nonce":"<b64 12B>","ciphertext":"<b64>"}
```

The plaintext is the JSON body of the OpenAPI operation (for Pix, including
the `signature` object).

## Request (no body)

```http
GET /statements HTTP/1.1
Authorization: Bearer <RS256 JWT>
X-Quantum-Envelope: <base64url({"v":1,"kid":"…","mlkemCiphertext":"…","x25519PublicKey":"…"})>
```

## Response

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.quantum-bank.envelope+json
X-Quantum-Envelope-Content-Type: application/json

{"v":1,"kid":"…","nonce":"<b64 12B>","ciphertext":"<b64>"}
```

Status codes are preserved. A problem produced before the envelope filter
(401/403 from the resource server or the gateway) is plaintext
`application/problem+json`.

## Key derivation

```
ss      = ML-KEM-768.Decaps(dk, mlkemCiphertext) || X25519(x25519Private, x25519PublicKey)
prk     = HKDF-Extract(SHA-256, salt = "quantum-bank-envelope-v1", ikm = ss)
k_req   = HKDF-Expand(prk, "request\0"  || kid || "\0" || aad, 32)
k_resp  = HKDF-Expand(prk, "response\0" || kid || "\0" || aad, 32)
aad     = METHOD + " " + PATH   (e.g. "POST /pix/transfers")
AEAD    = AES-256-GCM(key, nonce, plaintext, aad)
```

## Errors (problem+json)

| Status | errorCode | When |
| --- | --- | --- |
| 400 | `envelope_required` | client must use the envelope and did not |
| 400 | `envelope_invalid` | malformed envelope or AEAD failure |
| 400 | `envelope_key_unknown` | `kid` not current or within grace |
| 400 | `signing_key_required` | CSR from a required client without registration |
| 400 | `signing_key_invalid` | registration proof or algorithm invalid |
| 400 | `transaction_signature_required` | Pix from a required client without signature |
| 400 | `transaction_signature_invalid` | verification, device key, skew failure |
| 409 | `transaction_signature_replayed` | nonce already used |

## CSR submission additions

Request: `signingKey: {"alg":"ML-DSA-65","publicKey":"<b64 1952B>","proof":"<b64>"}`,
proof = ML-DSA-65 signature over the DER CSR, context `quantum-bank-signing-key-v1`.

Response: `envelopeKeys: {"keySet":{…},"signature":"<b64>","signerChain":["-----BEGIN CERTIFICATE-----…"]}`.
