# Theme Conquest Rules (WIP)

**Purpose.** Theme Conquest is an **optional MVP event** that showcases Craters’ loop **without ad spend**. Users’ interactions generate **receipts**; rankings come from those receipts. At the finale, winners receive **R_future** (claim vouchers). **No R is spent in MVP.** Royalty splits activate when **R** is live (claims convert 1:1, subject to KYC/ToS).

---

## Cycle
1. **Announce theme** (e.g., `#dailyoutfit`) with start/end time.
2. **Submit posts** (must include the theme tag; normal tagging rules apply).
3. **Live ranking** from the receipt stream.
4. **Finale & awards** → emit `THEME_R_AWARD` receipts (units: `R_future`).

---

## Ranking inputs (public, tunable)
- **Unique tagged acknowledgements** (primary signal; weight `ρ_tag`)
- **Saves / dwell** (bounded; weight `ρ_save`)
- **Views** (capped per actor/device; weight `ρ_view`)
- **Recency decay** `λ(age)` to favor late momentum
- **Per-user per-post cap** `σ` to reduce gaming

> All inputs are **receipts-backed**. Manual moderation may void fraudulent receipts.

### Public scoring sketch (illustrative)

$$
\text{score}(p)=\sum_{i \in \mathcal{R}(p)} \rho(i)\,\lambda(\mathrm{age}_i)
\quad \text{s.t. per-actor cap } \sigma
$$

### (clarification under the scoring formula)
Where \( \mathcal{R}(p) \) denotes the set of receipts linked to post \(p\).

---

## Allocation (math)
Let \( B_{R_{\mathrm{future}}} \) be the theme award pool and let \( r(p) \) be the rank of post \( p \) (1 = best). Two allocation options:

### A) Rank-bucket split (matches your prose example)

$$
w(r)=
\begin{cases}
0.50, & r=1\\
0.10, & r\in\{2,3,4,5\}\\
0.02, & r\in\{6,7,8,9,10\}\\
0, & \text{otherwise}
\end{cases}
\qquad
A(p)= B_{R_{\mathrm{future}}}\cdot w\!\left(r(p)\right).
$$

### B) Proportional to score (Top \(K\))
Choose \(K\) and smoothing \(\alpha \in [1,2]\). For the top \(K\) posts:

$$
A(p)= B_{R_{\mathrm{future}}} \cdot
\frac{\mathrm{score}(p)^{\alpha}}
{\sum_{q \in \mathrm{Top}\,K}\mathrm{score}(q)^{\alpha}}
\quad \text{s.t.} \quad A_{\min} \le A(p) \le A_{\max}.
$$

---

## UI requirements (MVP)
- Countdown timer; **live leaderboard**.
- Post card shows receipt-backed stats (unique tags, saves, capped views).
- Profile shows **R_future** balance and **download receipts**.

---

## Integrity & appeals
- **Whistleblow**: temporary deduction receipts (reversible after review).
- **Appeals / forum**: decisions are logged as receipts.
- Device/IP heuristics and per-actor caps to limit sybil amplification.

---

## Notes
- **No C “bounties”:** during MVP, C circulates via recognition/likes/tags; **R is the ad currency at launch**, not used in MVP.
- At launch, **royalty splits** flow to tagged parties per the accumulated receipt trail.

---

## TODO
- [ ] Publish exact weights (`ρ_*`) and decay `λ`.
- [ ] Document tie-break rules and suspension criteria.
- [ ] Add a worked example with 10 posts and the final allocation.
