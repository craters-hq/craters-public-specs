# Craters Signature Format (v1)

**Status:** Draft for public review
**Scope:** Receipts, Manifests, Court & Conquest records
**Algorithms:** Ed25519 signatures over SHA‑256 framed payloads

## 1. Goals

* Deterministic, replay‑safe, and auditable signatures for all integrity‑critical events.
* Clear separation of **master CA key** (offline) and **operational subkeys** (online, rotating).
* Simple verification for third parties with only **public artifacts**: payload JSON, detached signature, and `keys.json`.

## 2. Canonicalization & Hash Framing

All signed objects are canonicalized before hashing.

**Canonical JSON rules**

* UTF‑8, no BOM.
* Strict JSON (no comments, NaN, Infinity).
* Object keys sorted lexicographically (ASCII byte order).
* No trailing zeros in numbers; integers where possible.
* Timestamps in RFC 3339 (`YYYY-MM-DDThh:mm:ssZ`).

**Framed hash input** (prevents cross‑type substitution):

```
"CRATERS|v1|<TYPE>|<CONTENT-BYTES>"
```

Where `<TYPE>` ∈ {`RECEIPT`,`MANIFEST`,`COURT`,`CONQUEST`} and `<CONTENT-BYTES>` is the canonical JSON bytes of the object **excluding** the `signature` block.

Hash algorithm: `SHA-256` over framed input → `hash`.

## 3. Detached Signature Block

The signature is detached and stored alongside the payload.

```json
{
  "signature": {
    "scheme": "ed25519",
    "key_id": "ed25519-2025-10-a",
    "hash": "sha256:4f2c…",
    "sig": "base64:MEUCIQD…",
    "created_at": "2025-11-12T00:00:00Z"
  }
}
```

Notes:

* `hash` is the hex-encoded SHA‑256 of the framed payload.
* `sig` is Base64 of the Ed25519 signature over the **raw hash bytes**.
* The payload itself carries a **`version`** and **`type`** field; signers MUST verify them before accepting.

## 4. Key Hierarchy

1) **Master CA (offline, shardable, effectively immortal)**
   - Signs *Operational CA* certs and **CRLs**.
   - Never used online. Stored offline with multi-party ceremony.

2) **Operational CA (nearline HSM, unlocked with human action)**
   - Signs **Operational Signer** subkeys.
   - Rotates ~6–12 months; at least two active to ease emergency rollover.

3) **Operational Signers (online, short-lived)**
   - Sign **RECEIPT/MANIFEST/COURT/CONQUEST** objects.
   - Rotate frequently (e.g., 30–90 days) or on incident.

### Public Documents
- `/public/keys.json` contains the **certificate chain**:
  - `master_ca` (pub), `operational_cas` (pub), and `signer_keys` (pub) with parent references.
- `/public/crl.json` contains revocations for:
  - operational signers, and (if ever needed) operational CAs.

### Example (excerpt)
```json
{
  "version": "1.1.0",
  "master_ca": {
    "key_id": "master-2025-01",
    "public_key_base64": "…",
    "created_at": "2025-01-01T00:00:00Z"
  },
  "operational_cas": [
    {
      "key_id": "opca-2025-10",
      "parent": "master-2025-01",
      "public_key_base64": "…",
      "not_before": "2025-10-01T00:00:00Z",
      "not_after":  "2026-04-01T00:00:00Z"
    }
  ],
  "signer_keys": [
    {
      "key_id": "ed25519-2025-11-a",
      "parent": "opca-2025-10",
      "use": ["RECEIPT","MANIFEST"],
      "public_key_base64": "…",
      "not_before": "2025-11-01T00:00:00Z",
      "not_after":  "2026-01-01T00:00:00Z",
      "revoked": false
    }
  ]
}
```


## 5. Revocation (CRL)

`/public/crl.json` lists revoked subkeys and specific signature IDs if needed.

```json
{
  "version": "1.0.0",
  "revoked_subkeys": [
    {"key_id":"ed25519-2025-10-a","revoked_at":"2025-11-20T12:34:56Z","reason":"suspected compromise"}
  ],
  "revoked_signatures": [
    {"sig_id":"sig-abc123","revoked_at":"2025-11-21T00:00:00Z","reason":"malformed framing"}
  ]
}
```

`crl.json` is signed by the **master key**; verifiers MUST check revocation before accepting a signature.

## 6. Receipt Payload Schema (excerpt)

```json
{
  "version": "1",
  "type": "RECEIPT",
  "receipt_id": "rcpt-…",
  "kind": "STEWARDSHIP_ASSIGNED",
  "actors": {"user_id":"u_…","tag_id":"tag_…"},
  "amounts": {"R": -500},
  "rules": {"pricing_version":"1.4.0","limits_version":"1.2.3","ruleset_id":"primary-sale-std"},
  "idempotency_key": "uuid-…",
  "nonce": 42,
  "manifest_id": "m-2025-11-12",
  "created_at": "2025-11-12T00:00:00Z"
}
```

The **`signature`** block is stored adjacent to (or embedded in) the document but excluded from the canonicalized bytes.

## 7. Verification Procedure (Pseudocode)

```
bytes = canonical_json_without_signature(payload)
framed = b"CRATERS|v1|" + payload.type + b"|" + bytes
hash = sha256(framed)
assert signature.hash == "sha256:" + hex(hash)

pub = keys_json.get(signature.key_id)
assert pub.exists and !pub.revoked and now < pub.expires_at
assert signature.scheme == "ed25519"
assert ed25519_verify(pub.key, hash, b64decode(signature.sig))

assert crl_json.not_revoked(signature.sig)
```

## 8. Security Considerations

* **Replay Protection:** Use `nonce` + `idempotency_key` inside payload, verified by server.
* **Cross‑Type Safety:** Framing string binds signatures to object type & version.
* **Key Scope:** Distinct subkeys for `RECEIPT` and `MANIFEST` reduce blast radius.
* **Clock Skew:** Verifiers allow small skew on `created_at` but MUST enforce expiry.
* **Transparency:** Every manifest includes `keys_version` and `pricing_version` used to compute that day’s receipts.

### Idempotency vs Nonce (Craters usage)

- **Idempotency Key** (client- or server-generated UUID in header/body):
  Ensures **the same *transaction*** is processed at most once, even if retried.
  Stored in an `idempotency_log` with (key, actor, action_fingerprint, response_hash, ttl).

- **Nonce** (monotonic per actor):
  Prevents **replay/reordering** of signed messages. Each actor has a strictly
  increasing integer; the Receipt Engine rejects nonces ≤ last_seen.

**Enforcement**
- Idempotency prevents *duplicate* effects from network retries.
- Nonce prevents *replay* or *reordering* of validly signed messages.

**DB sketch**


## 9. Implementation Notes

* Libraries: `libsodium`/`TweetNaCl` for Ed25519; standard SHA‑256.
* All public files (`keys.json`, `crl.json`) are mirrored to `/.well-known/` and a CDN with immutable URLs per version.

## 10. Change Log

* v1: Initial publication with Ed25519 detached signatures and CRL model.
