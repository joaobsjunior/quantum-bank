# Data Model: Post-Quantum Application Envelope

## EnvelopeKeySet (backend → app)

| Field | Type | Notes |
| --- | --- | --- |
| `kid` | string (≤ 64) | base64url(SHA-256(mlkemPublicKey ‖ x25519PublicKey))[0..22] |
| `alg` | string | `X25519MLKEM768-HKDF-SHA256-AES256GCM` |
| `mlkemPublicKey` | base64 | 1184 bytes, FIPS 203 ML-KEM-768 encapsulation key |
| `x25519PublicKey` | base64 | 32 bytes, RFC 7748 |
| `notAfter` | RFC 3339 | key set validity; the app re-enrolls after it |

Canonical bytes for signing: UTF-8 of
`kid\nalg\nmlkemPublicKey\nx25519PublicKey\nnotAfter\n` (base64 values as
transmitted). Signature: ML-DSA-65, context `quantum-bank-envelope-keys-v1`.

## SignedEnvelopeKeySet

| Field | Type | Notes |
| --- | --- | --- |
| `keySet` | EnvelopeKeySet | |
| `signature` | base64 | ML-DSA-65 over the canonical bytes |
| `signerChain` | string[] | PEM certificates, leaf first, ending with the issuing CA; the app holds the root |

## EnvelopeMessage (both directions)

| Field | Type | Notes |
| --- | --- | --- |
| `v` | int | 1 |
| `kid` | string | key set used |
| `mlkemCiphertext` | base64 | 1088 bytes (request only) |
| `x25519PublicKey` | base64 | 32 bytes, ephemeral (request only) |
| `nonce` | base64 | 12 bytes, random per message |
| `ciphertext` | base64 | AES-256-GCM output with 16-byte tag |
| `contentType` | string | original media type of the plaintext (response only) |

For `GET` requests the message without `ciphertext` is sent base64url-encoded
in the `X-Quantum-Envelope` header; the response is still an envelope body.

## DeviceSigningKey (backend, H2)

```sql
CREATE TABLE device_signing_keys (
    subject VARCHAR(160) NOT NULL,
    device_id VARCHAR(160) NOT NULL,
    algorithm VARCHAR(20) NOT NULL,          -- ML-DSA-65
    public_key VARBINARY(4096) NOT NULL,     -- raw FIPS 204 public key (1952 bytes)
    registered_at TIMESTAMP NOT NULL,
    PRIMARY KEY (subject, device_id)
);
```

## TransactionSignature (request field + backend storage)

Request field on `POST /pix/transfers`:

| Field | Type | Notes |
| --- | --- | --- |
| `alg` | string | `ML-DSA-65` |
| `deviceId` | string | client identifier pattern of feature 007 |
| `nonce` | string | UUID v4 |
| `issuedAt` | RFC 3339 | ±120 s of backend clock |
| `value` | base64 | ML-DSA-65 signature, context `quantum-bank-pix-v1` |

Canonical message (UTF-8): `quantum-bank-pix-v1\n` followed by one line per
field in this order: `subject`, `deviceId`, `amount` (scale 2, plain string),
`recipientKey`, `description` (empty string when absent), `scenario`, `nonce`,
`issuedAt`.

```sql
CREATE TABLE pix_transaction_signatures (
    nonce VARCHAR(80) PRIMARY KEY,
    subject VARCHAR(160) NOT NULL,
    device_id VARCHAR(160) NOT NULL,
    transaction_id VARCHAR(80),
    algorithm VARCHAR(20) NOT NULL,
    signature VARBINARY(8192) NOT NULL,
    issued_at TIMESTAMP NOT NULL,
    verified_at TIMESTAMP NOT NULL
);
```

The nonce row is inserted before the transfer is simulated (replay rejection is
a primary-key violation), and updated with the transaction id afterwards.

## Mobile certificate state

`ReadyCertState` gains `envelopeKeySet` (verified EnvelopeKeySet) and
`signingKey` (ML-DSA-65 seed, public key, level). Both are required for a
banking call; a ready state without them is treated as `missing`.
