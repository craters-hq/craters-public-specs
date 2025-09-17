# Theme Conquest Rules (WIP)

**Model recap:** Theme Conquest is an **optional MVP event** to showcase the loop without ad spend. Rankings are driven by organic interaction receipts (e.g., likes with tags, saves, views with bounds). At the finale, winners receive **R_future** (claim vouchers). **No R is spent in MVP**; royalty splits activate when **R** is live and claims convert 1:1 (subject to KYC/ToS).

## Cycle
1. **Announce theme** (e.g., #dailyoutfit) with start/end time.
2. **Submissions window** (posts with valid tags).
3. **Live ranking** (updates from receipt stream).
4. **Finale & allocation** → emit `THEME_R_AWARD` receipts (R_future).

## Ranking inputs (public, tunable)
- **Unique tagged acknowledgements** (primary signal, weight `ρ_tag`)
- **Saves / dwell** (bounded contribution, weight `ρ_save`)
- **Views** (capped per user/device, weight `ρ_view`)
- **Recency decay** `λ(age)` to favor late momentum
- **Per-user per-post cap** `σ` to reduce gaming

> All inputs come from **receipts**; manual moderation can remove fraudulent receipts.

## Allocation (example split)
- Pool `B_R_future` per theme.
- Top 1: 50%, Ranks 2–5: 40% (even), Ranks 6–10: 10% (even).
- Emit one `THEME_R_AWARD` receipt per winner (amount in `R_future` units).

## UI requirements (MVP)
- Countdown timer; live leaderboard.
- Post card shows interaction counts and receipt-backed stats.
- Profile shows `R_future` balance and downloadable receipts.

## Anti-gaming & integrity
- Unique-actor checks; device/IP heuristics; recency weighting.
- Whistleblow deductions (reversible) recorded as receipts.
- Appeals flow (forum/court) logged with decision receipts.

## TODO
- [ ] Publish the exact weights (`ρ_*`) and decay `λ`.
- [ ] Document tie-break rules and suspension criteria.
- [ ] Add worked example with 10 posts and final allocation.
