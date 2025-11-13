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

## 4. Key Hierarchy & Rotation

### 4.1 Master Key (Certificate Authority)

* Offline, split‑knowledge storage; used only to **issue/revoke subkeys**.
* Publishes signed `keys.json` and `crl.json`.

### 4.2 Subkeys (Operational Signers)

* Key IDs follow: `ed25519-YYYY-MM-<series>` (e.g., `ed25519-2025-10-a`).
* Rotate on a fixed cadence (e.g., monthly/quarterly) or after incident.
* Allowed uses are scoped: `{RECEIPT, MANIFEST, COURT, CONQUEST}`.

### 4.3 Public Keys Document (`/public/keys.json`)

```json
{
  "version": "1.0.0",
  "issued_by": "master-2025-01",
  "keys": [
    {"key_id":"ed25519-2025-10-a","alg":"ed25519","use":["RECEIPT","MANIFEST"],
     "public_key_base64":"MCowBQYDK2VwAyEA…","created_at":"2025-10-01T00:00:00Z","expires_at":"2026-01-01T00:00:00Z","revoked":false}
  ]
}
```

`keys.json` itself is signed by the **master key** and published with a detached signature: `keys.json.sig`.

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

## 9. Implementation Notes

* Libraries: `libsodium`/`TweetNaCl` for Ed25519; standard SHA‑256.
* All public files (`keys.json`, `crl.json`) are mirrored to `/.well-known/` and a CDN with immutable URLs per version.

## 10. Change Log

* v1: Initial publication with Ed25519 detached signatures and CRL model.
