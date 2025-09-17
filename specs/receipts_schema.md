# Receipts Schema (WIP)

**Purpose.** Tamper-evident records of user interactions and allocations. Every receipt can be exported per user and verified later.

## Fields (common to all receipts)
- `receipt_id` (string, uuid)
- `type` (enum: `C_TRANSFER`, `TAG_ACK`, `THEME_R_AWARD`, `WHISTLE_DEDUCT`, `WHISTLE_RETURN`)
- `user_id` (string) — primary beneficiary/actor
- `post_id` (string|null)
- `theme_id` (string|null)
- `amount` (number) — units depend on `type`
- `units` (enum: `C` | `R_future` | `R`)
- `ts` (ISO8601)
- `issuer` (string; service id, e.g., `craters.mvp.worldengine`)
- `sig` (string; signature placeholder in MVP)

## Verification (MVP → Launch)
- **MVP:** server-generated receipts; users can download JSON.
- **Launch:** add signing key, public-key rotation, and verification API (Ed25519 recommended).

## Type-specific invariants (examples)
- `C_TRANSFER`: sum of transfers for a post ≤ creator’s available C at time of emission.
- `THEME_R_AWARD`: Σ winners’ `amount` = `B_{R_future}` for that theme.
- `WHISTLE_*`: paired deduct/return receipts must net to zero unless appeal is upheld.

## Example

```json

{
  "receipt_id": "rcpt_R1",
  "type": "THEME_R_AWARD",
  "user_id": "usr_B",
  "post_id": "pst_Y",
  "theme_id": "thm_wk37",
  "amount": 500,
  "units": "R_future",
  "ts": "2025-09-24T12:00:00Z",
  "issuer": "craters.mvp.worldengine",
  "sig": "SIGNATURE_PLACEHOLDER"
}
```
## TODO
- [ ] Finalize signature scheme and rotation cadence.
- [ ] Publish full invariant set + validator checklist.

# Migration Plan (MVP → Launch)

## Mapping
- `User.C_balance (MVP)` → `User.C_balance (Launch)` (no burn/reset)
- `User.R_future_balance (MVP)` → `User.R_balance (Launch)` 1:1 (subject to KYC/ToS)
- `receipts[] (MVP)` → retained verbatim as history proofs

## Steps
1. **Snapshot** MVP DB and receipt store (height + hash).
2. **Validate** balances against receipts (sum/invariants check).
3. **Import** into launch DB; attach “Imported from MVP (snapshot X)” tag.
4. **Notify users**; show import banner + export link to raw snapshot receipts.

## Attestations (optional on-chain)
- Publish snapshot hash + R floor calc proof.
- Post-launch, publish periodic payout proofs.

## TODO
- [ ] Write the snapshot validator checklist.
- [ ] Decide where to publish attestations (chain + cadence).
