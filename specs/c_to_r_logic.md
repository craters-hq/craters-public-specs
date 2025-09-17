# Public Half of C → R Logic (WIP)

**Model recap:** **C** are on-platform pieces that move with user recognition (likes, tagged acknowledgements, etc.). Every interaction emits a signed **receipt** that attributes influence to the right parties. When **R** (the attention/ad currency) is live, users’ influence—tracked by receipts and C circulation—**mints R** under a metabolic rule. **C always recirculates on-platform; R has a cash-out path** (fiat/stablecoin via partners, subject to KYC/AML). R price is managed with solvency/floor mechanisms; C’s containment supports the R ceiling.

## Scope of this doc
- Define visible/public parameters of the metabolic mint (private tuning stays internal).
- State invariants and guardrails so reviewers understand how C→R cannot be gamed.

## Invariants (public)
- **C non-exit:** C cannot be withdrawn off-platform.
- **Receipts first:** All R mint events must reference prior receipts (actions → influence).
- **Backed R:** Aggregate R minted ≤ budget derived from ad/boost revenue & policy quotas.
- **Price discipline:** R floor ≥ treasury cash / outstanding R (published attestations).
- **Throttle & appeals:** Tokens can be temporarily deducted on reports; reversible via review.

## Public variables (placeholders)
- `ρ` — receipt weight per action type (like, tagged ack, etc.)
- `λ(t)` — recency decay for receipts
- `σ` — per-user cap per post/theme window
- `B_R` — R mint budget per epoch (from revenue/allocations)
- `K` — normalization constant for fairness across cohorts

## Public mint sketch (illustrative, not final)
For user *u* in epoch *E*:

$$
\text{influence}_u = \sum_{i \in \mathrm{receipts}(u)} \rho(i)\,\lambda(\mathrm{age}_i) \quad \text{s.t. } \sigma
$$

$$
R^{\mathrm{mint}}_{u} = B_R \cdot \frac{\text{influence}_u}{\sum_{v} \text{influence}_v}
$$

> **Note:** Internals may adjust K, dampen self-loops, and account for retained C. Full formula + proofs will be published at launch.

## What is visible in MVP
- User sees: `C balance`, `C received/spent`, and **R_future** (claim vouchers).
- Exportable **receipts JSON** per user.

## TODO
- [ ] Publish concrete `ρ`, `λ`, and `σ` values for MVP auditing.
- [ ] Add examples showing how tags route royalties at launch.
- [ ] Add link to price-floor attestation plan (on-chain option).
