# Craters Receipts Spec (v0.1.0)

**Purpose.** A receipt is an immutable event record emitted by user interactions. Receipts power attribution, throttles, and (later) payouts. Every reward must trace back to signed receipts.

## 1. Event Types
- `post.create`
- `tag.add`
- `like.create`
- `comment.create`
- `boost.purchase`
- `id.purchase`

## 2. Canonical JSON
- UTF-8, sorted keys, no trailing spaces.
- Timestamps: ISO-8601 UTC.

## 3. Fields (common)
- `receipt_id` (string, ULID/UUID)
- `ts` (string, ISO-8601)
- `actor_id` (string)
- `action` (enum; see Event Types)
- `target_id` (string; post/comment/id as applicable)
- `tags` (array of strings)
- `context` (object; minimal PII)
- `value_basis` (object):
  - `r_floor_cents` (number) – floor reference at time of event
  - `quota_bucket` (string) – issuance bucket (e.g., `viewer_daily_v1`)
  - `weight` (number; default 1.0)
- `split_v1` (array):
  - items: `{ "to": "<role|id>", "c_share": <0..1> }` (sum=1.0)
- `signature`:
  - `alg` (`ed25519` for v0.1.0)
  - `pubkey` (base58/base64)
  - `sig` (base64)

## 4. Signing
- Message = canonical JSON of the receipt *excluding* `signature`.
- Signer: Craters World Engine key (`pubkey` rotated via `/docs/keys.md`, TBD).
- Verify: `ed25519.verify(sig, message, pubkey)`.

## 5. Privacy
- Public: event type, ts, tags, splits, signature.
- Redacted: raw user PII in `context`.

## 6. Anti-gaming hooks
- `weight` allows throttles (e.g., drop low-quality interactions to 0.1).
- Receipts may be **invalidated** by a follow-up `receipt.invalidate` with reason.

## 7. Examples
See `/examples/demo_receipts.json`.

## 8. Versioning
- Semver. Current: **v0.1.0**.
- Changes tracked in `/CHANGELOG.md`.

## 9. Future (not enforced in v0.1.0)
- On-chain anchors for daily digests.
- R-issuance proofs derived from receipt sets.

MVP format (array) vs v0.1 canonical (single). Use v0.1 going forward.
