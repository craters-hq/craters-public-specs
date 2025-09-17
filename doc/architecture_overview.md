# Architecture Overview (WIP)

**Goal.** Ship fast off-chain, keep receipts/verifiability, and enable on-chain attestations at launch.

## MVP (off-chain world engine)
- **App/API layer**: auth (email OTP), handles, posts, tags, interactions.
- **Receipts service**: emits tamper-evident receipts per action.
- **DB**:
  - `users` (handle, country_code, flags, balances: C, R_future)
  - `posts`, `tags`
  - `receipts` (type, payload_json, sig placeholder)
  - `themes` (config, winners)
- **Jobs/Scheduler**: reservation expiry, theme finale, exports.
- **Anti-abuse**: rate limits, per-actor caps, whistleblow deductions (reversible).

## Data flow (MVP)
1. **Action** (like/tag/save) → **Controller** validates → **Receipts service** builds payload → write **receipt** + update **balances**.
2. **Theme loop** → ranking reads receipts stream; finale allocates **R_future** and emits `THEME_R_AWARD`.
3. **Exports**: user profile → “Download my receipts (JSON)”; admin snapshot for migration.

> Diagram placeholder: *(to add)* User → API → Receipts → DB (balances, receipts) → Exports/Attestations.

## Security & privacy
- Minimal PII (email, country code).  
- OTP verification, rate-limit per email/IP/device.  
- Audit trail via receipts; admin actions also receipt-logged.

## Launch options (on-chain)
- **Payout proofs**: publish digest of paid receipts.
- **Price-floor attestations**: treasury/floor calculations periodically posted.
- **Optional P2P R**: enable transfers/markets if policy allows.

## Compliance notes
- **R** treated as income; KYC tiers & quotas at activation.  
- **C** stays on-platform; **R_future → R** 1:1 at launch (subject to KYC/ToS).

## Migration (MVP → Launch)
- Snapshot DB + receipt store → validate balances vs receipts → import → show “Imported from MVP (snapshot X)” banner to users.

## Scalability & integrity
- Idempotency keys on writes; background workers for heavy tasks.
- Per-actor caps (`σ`) and recency decay (`λ`) applied in ranking.
- Content moderation can void fraudulent receipts (kept for audit).

## Interfaces (public)
- `GET /users/:id/receipts.export` (JSON bundle)
- `GET /themes/:id/leaderboard`
- `POST /whistleblow` (creates deduction receipt; reversible)

## TODO
- [ ] Add data-flow diagram.
- [ ] Define receipt signing & key rotation.
- [ ] Publish payout/attestation cadence.
