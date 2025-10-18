## Spec quick links
- 📄 Receipts Spec: [`/docs/receipts.md`](docs/receipts.md)
- 🧪 MVP Example (array): [`/examples/demo_receipts_mvp.json`](examples/demo_receipts_mvp.json)
- 🧪 v0.1 Example (single): [`/examples/demo_receipt_v0.1.json`](examples/demo_receipt_v0.1.json)
- 🧾 Changelog: [`/CHANGELOG.md`](CHANGELOG.md)
- ✅ Schema: [`/specs/receipts.schema.json`](specs/receipts.schema.json)

# Craters — Public Specs (MVP → Launch)

**Short pitch:** Get tagged and collect royalties.  
**Now (MVP):** **C** = on-platform pieces transferred by user interactions (e.g., likes / recognition with tags). Every interaction emits a **signed receipt**. Optional events (e.g., **Theme Conquest**) can reveal **R_future** claim vouchers; no ad spend yet.  
**Later (Launch):** **R** = attention-unit / ad currency. Advertisers convert fiat/stablecoin → R to run boosts. C→R mint logic credits users’ influence into **cash-out-able R** (subject to KYC/AML). Royalty splits flow to tagged parties; R price is managed with solvency/floor models. C always recirculates on-platform.

## Repo Map

- [Receipts schema](specs/receipts_schema.md) — Signed-message receipt format (WIP examples)
- [C→R mint logic](specs/c_to_r_logic.md) — Public half of the “metabolic” model
- [Theme Conquest rules](specs/theme_conquest_rules.md) — Ranking, allocation, tie-breakers
- [Architecture overview](docs/architecture_overview.md) — Receipts Engine (off-chain) + on-chain options
- [Migration plan](docs/migration_plan.md) — How MVP balances/receipts carry forward to launch
- [Demo receipts (JSON)](examples/demo_receipts.json)
- [IR Deck (PDF)](deck/craters_ir_deck_v2.pdf)
- [Compliance notes](docs/compliance_notes.md) — (WIP)
- [Roadmap](roadmap/README.md) — (WIP)

## Quick Overview
- **Interactions → Receipts:** User actions (likes, tags, etc.) create **tamper-evident receipts** that track influence and royalty weights.
- **C circulates; R mints:** Users **spend/transfer C** inside the platform. Under the metabolic formula, accumulated C + receipts **mint R** when R is live.
- **R = ad currency:** Advertisers fund boosts in **R**; exposure and participation credit users toward R (cash-out path via partners, KYC/AML).
- **Royalty splits via tags:** Tagged contributors receive ongoing **R** shares once R is active, per receipt trails.
- **Throttles / integrity:** Tokens can be temporarily deducted on reports (whistleblowing) and reclaimed after review; forum/“court” planned.

## Why off-chain first?
Speed and iteration. We publish **verifiable receipts** and plan to settle critical flows on-chain (payout proofs, price-floor attestations).

## Links
- Website: https://craters.co
- Figma Demo: <link>
- 20-sec Clip: <link>
- IR Deck (PDF): https://docsend.com/view/43an5e9a4848nn7h

## License
Apache-2.0
