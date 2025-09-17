# Migration Plan (MVP → Launch)

Mapping:
- User.C_balance (MVP) → User.C_balance (Launch)
- User.R_future_balance (MVP) → User.R_balance (Launch) 1:1 (KYC/ToS)
- receipts[] → retained as history proofs

Steps: snapshot → verify → import → notify.

## TODO
- [ ] Snapshot verification checklist
