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
- Publish snapshot hash + R floor calculation proof.
- Post-launch, publish periodic payout proofs.

## TODO
- [ ] Write the snapshot validator checklist.
- [ ] Decide where to publish attestations (chain + cadence).
